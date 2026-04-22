<script>
  import wordsData from "./words.json";
  import { toIPA } from "../lib/toIPA";

  export let navigate;

  const cleanWord = (w) => w.replace(/\d+$/, "");

  function WeightedDL(s1, s2) {
    function substitutionCost(a, b, i, j) {
      if (a === b) return 0;
      let cost = 1;
      if (i < 2) cost += 0.5;
      return cost;
    }
    let tokens = s2.split(", ");
    if (s1 === s2) return -1;
    if (tokens.includes(s1)) return -0.5;
    const m = s1.length;
    const n = s2.length;
    const dp = Array.from({ length: m + 1 }, () => new Array(n + 1).fill(0));
    for (let i = 0; i <= m; i++) dp[i][0] = i;
    for (let j = 0; j <= n; j++) dp[0][j] = j;
    for (let i = 1; i <= m; i++) {
      for (let j = 1; j <= n; j++) {
        const costSub = substitutionCost(s1[i - 1], s2[j - 1], i, j);
        dp[i][j] = Math.min(dp[i - 1][j] + 1, dp[i][j - 1] + 1, dp[i - 1][j - 1] + costSub);
        if (i > 1 && j > 1 && s1[i - 1] === s2[j - 2] && s1[i - 2] === s2[j - 1]) {
          dp[i][j] = Math.min(dp[i][j], dp[i - 2][j - 2] + 0.5);
        }
      }
    }
    return dp[m][n] / Math.max(m, n);
  }

  let words = wordsData.map((w, index) => {
    const displayWord = cleanWord(w.word);
    const ipa = toIPA(displayWord);
    return {
      ...w,
      displayWord,
      index,
      ipa,
    };
  });
  let errors = words.filter((word) => word.ipa.error);

  import { tick, onMount } from "svelte";
  import { preventDefault } from "svelte/legacy";

  let q = "";
  let sortBy = "best-match";
  let selected = null;
  let drawerEl;
  let initialized = false;

  onMount(() => {
    const params = new URLSearchParams(window.location.search);
    if (params.has("q")) q = params.get("q");
    if (params.has("sort")) sortBy = params.get("sort");

    if (params.has("word")) {
      const w = words.find((item) => item.word === params.get("word"));
      if (w) selected = w;
    }
    initialized = true;
  });

  // Update URL when search, sort, or selection changes
  $: {
    if (initialized && typeof window !== "undefined") {
      const url = new URL(window.location.href);
      if (q.trim()) url.searchParams.set("q", q.trim());
      else url.searchParams.delete("q");

      if (sortBy !== "best-match") url.searchParams.set("sort", sortBy);
      else url.searchParams.delete("sort");

      if (selected) url.searchParams.set("word", selected.word);
      else url.searchParams.delete("word");

      window.history.replaceState({}, "", url.toString());
    }
  }

  const parseDate = (d) => {
    if (!d) return 0;
    const [day, month, year] = d.split("-").map(Number);
    return new Date(year, month - 1, day).getTime();
  };

  const formatDate = (d) => {
    if (!d) return "";
    const [day, month, year] = d.split("-").map(Number);
    const date = new Date(year, month - 1, day);
    return date.toLocaleDateString("en-GB", {
      day: "numeric",
      month: "long",
      year: "numeric",
    });
  };

  $: normalized = q.trim().toLowerCase();
  $: filtered = normalized
    ? words.filter(
        (w) =>
          w.displayWord.toLowerCase().includes(normalized) ||
          (w.definition && w.definition.toLowerCase().includes(normalized)) ||
          (WeightedDL(normalized, w.displayWord.toLowerCase()) < 0.4) || // play with this number
          (WeightedDL(normalized, w.definition.toLowerCase()) < 0.5), // this one too
      )
    : words;
  $: sorted = (() => {
    const copy = [...filtered];
    switch (sortBy) {
      case "alphabetical":
        return copy.sort((a, b) => a.displayWord.localeCompare(b.displayWord));
      case "reverse":
        return copy.sort((a, b) => b.displayWord.localeCompare(a.displayWord));
      case "newest":
        return copy.sort((a, b) => {
          const dateA = parseDate(a.date);
          const dateB = parseDate(b.date);
          if (dateB !== dateA) return dateB - dateA;
          return b.index - a.index;
        });
      case "oldest":
        return copy.sort((a, b) => {
          const dateA = parseDate(a.date);
          const dateB = parseDate(b.date);
          if (dateA !== dateB) return dateA - dateB;
          return a.index - b.index;
        });
      case "type":
        return copy.sort((a, b) => {
          const typeCompare = a.type.localeCompare(b.type);
          return typeCompare !== 0
            ? typeCompare
            : a.displayWord.localeCompare(b.displayWord);
        });
      case "length-short":
        return copy.sort((a, b) => {
          const lengthCompare = a.displayWord.length - b.displayWord.length;
          return lengthCompare !== 0
            ? lengthCompare
            : a.displayWord.localeCompare(b.displayWord);
        });
      case "length-long":
        return copy.sort((a, b) => {
          const lengthCompare = b.displayWord.length - a.displayWord.length;
          return lengthCompare !== 0
            ? lengthCompare
            : a.displayWord.localeCompare(b.displayWord);
        });
      case "best-match":
        if (normalized === "") {
          return copy.sort((a, b) => a.displayWord.localeCompare(b.displayWord));
        } else if (normalized === "noun") {
          return copy.map(word => {
            const wdist = WeightedDL(normalized, word.displayWord.toLowerCase());
            const defsplit = word.definition.toLowerCase().split(", ")
            let mindef = 999;
            let fdist = 999;
            for (const def of defsplit) {
              fdist = WeightedDL(normalized, def);
              if (fdist < mindef) mindef = fdist;
            }
            return {word, dist: Math.min(wdist, mindef)};
          }).sort((a, b) => a.dist - b.dist).map(x => x.word).filter((w) => w.type !== "noun").concat(words.filter((w) => w.type === "noun"));
        } else if (normalized === "verb") {
          return copy.map(word => {
            const wdist = WeightedDL(normalized, word.displayWord.toLowerCase());
            const defsplit = word.definition.toLowerCase().split(", ")
            let mindef = 999;
            let fdist = 999;
            for (const def of defsplit) {
              fdist = WeightedDL(normalized, def);
              if (fdist < mindef) mindef = fdist;
            }
            return {word, dist: Math.min(wdist, mindef)};
          }).sort((a, b) => a.dist - b.dist).map(x => x.word).filter((w) => w.type !== "verb").concat(words.filter((w) => w.type === "verb"));
        } else if (normalized === "modifier") {
          return copy.map(word => {
            const wdist = WeightedDL(normalized, word.displayWord.toLowerCase());
            const defsplit = word.definition.toLowerCase().split(", ")
            let mindef = 999;
            let fdist = 999;
            for (const def of defsplit) {
              fdist = WeightedDL(normalized, def);
              if (fdist < mindef) mindef = fdist;
            }
            return {word, dist: Math.min(wdist, mindef)};
          }).sort((a, b) => a.dist - b.dist).map(x => x.word).filter((w) => w.type !== "modifier").concat(words.filter((w) => w.type === "modifier"));
        } else if (normalized === "particle") {
          return copy.map(word => {
            const wdist = WeightedDL(normalized, word.displayWord.toLowerCase());
            const defsplit = word.definition.toLowerCase().split(", ")
            let mindef = 999;
            let fdist = 999;
            for (const def of defsplit) {
              fdist = WeightedDL(normalized, def);
              if (fdist < mindef) mindef = fdist;
            }
            return {word, dist: Math.min(wdist, mindef)};
          }).sort((a, b) => a.dist - b.dist).map(x => x.word).filter((w) => w.type !== "particle").concat(words.filter((w) => w.type === "particle"));
        } else if (normalized === "numeral") {
          return copy.map(word => {
            const wdist = WeightedDL(normalized, word.displayWord.toLowerCase());
            const defsplit = word.definition.toLowerCase().split(", ")
            let mindef = 999;
            let fdist = 999;
            for (const def of defsplit) {
              fdist = WeightedDL(normalized, def);
              if (fdist < mindef) mindef = fdist;
            }
            return {word, dist: Math.min(wdist, mindef)};
          }).sort((a, b) => a.dist - b.dist).map(x => x.word).filter((w) => w.type !== "numeral").concat(words.filter((w) => w.type === "numeral"));
        } else if (normalized === "interjection") {
          return copy.map(word => {
            const wdist = WeightedDL(normalized, word.displayWord.toLowerCase());
            const defsplit = word.definition.toLowerCase().split(", ")
            let mindef = 999;
            let fdist = 999;
            for (const def of defsplit) {
              fdist = WeightedDL(normalized, def);
              if (fdist < mindef) mindef = fdist;
            }
            return {word, dist: Math.min(wdist, mindef)};
          }).sort((a, b) => a.dist - b.dist).map(x => x.word).filter((w) => w.type !== "interjection").concat(words.filter((w) => w.type === "interjection"));
        } else if (normalized === "adposition") {
          return copy.map(word => {
            const wdist = WeightedDL(normalized, word.displayWord.toLowerCase());
            const defsplit = word.definition.toLowerCase().split(", ")
            let mindef = 999;
            let fdist = 999;
            for (const def of defsplit) {
              fdist = WeightedDL(normalized, def);
              if (fdist < mindef) mindef = fdist;
            }
            return {word, dist: Math.min(wdist, mindef)};
          }).sort((a, b) => a.dist - b.dist).map(x => x.word).filter((w) => w.type !== "adposition").concat(words.filter((w) => w.type === "adposition"));
        }
        return copy.map(word => {
          const wdist = WeightedDL(normalized, word.displayWord.toLowerCase());
          const defsplit = word.definition.toLowerCase().split(", ")
          let mindef = 999;
          let fdist = 999;
          for (const def of defsplit) {
            fdist = WeightedDL(normalized, def);
            if (fdist < mindef) mindef = fdist;
          }
          return {word, dist: Math.min(wdist, mindef)};
        }).sort((a, b) => a.dist - b.dist).map(x => x.word);
      case "noun-match":
        return copy
          .filter((w) => w.type === "noun")
          .sort((a, b) => a.displayWord.localeCompare(b.displayWord))
      case "verb-match":
        return copy
          .filter((w) => w.type === "verb")
          .sort((a, b) => a.displayWord.localeCompare(b.displayWord))
      case "modifier-match":
        return copy
          .filter((w) => w.type === "modifier")
          .sort((a, b) => a.displayWord.localeCompare(b.displayWord))
      case "particle-match":
        return copy
          .filter((w) => w.type === "particle")
          .sort((a, b) => a.displayWord.localeCompare(b.displayWord))
      case "numeral-match":
        return copy
          .filter((w) => w.type === "numeral")
          .sort((a, b) => a.displayWord.localeCompare(b.displayWord))
      case "interjection-match":
        return copy
          .filter((w) => w.type === "interjection")
          .sort((a, b) => a.displayWord.localeCompare(b.displayWord))
      case "adposition-match":
        return copy
          .filter((w) => w.type === "adposition")
          .sort((a, b) => a.displayWord.localeCompare(b.displayWord))
      default:
        return copy;
    }
  })();

  async function open(w) {
    selected = w;
    await tick();
    drawerEl?.focus();
  }
  function close() {
    selected = null;
  }
  function onCardKeydown(e, w) {
    if (e.key === "Enter" || e.key === " ") {
      e.preventDefault();
      open(w);
    }
  }
  function onWindowKeydown(e) {
    if (e.key === "Escape") {
      close();
    }
  }

  // Prevent background scroll when drawer is open
  $: document &&
    document.body &&
    (document.body.style.overflow = selected ? "hidden" : "");
</script>

<svelte:window on:keydown={onWindowKeydown} />

<main class="container">
  <header class="main-header">
    <h1>Words</h1>
    <a href="/" on:click|preventDefault={() => navigate("/etymology")}>
      Show etymology by country
    </a>
  </header>

  <div class="search">
    <input
      id="word-search"
      type="search"
      placeholder="Search words or definitions…"
      bind:value={q}
      autocomplete="off"
      spellcheck="false"
      aria-controls="word-grid"
      aria-label="Search words"
    />
    <span class="count" aria-live="polite">
      {sorted.length}
    </span>
  </div>

  <div class="sort-control">
    <label for="sort-select">Sort:</label>
    <select id="sort-select" bind:value={sortBy} aria-label="Sort order">
      <option value="alphabetical">A → Z</option>
      <option value="reverse">Z → A</option>
      <option value="newest">Newest</option>
      <option value="oldest">Oldest</option>
      <option value="type">Type</option>
      <option value="length-short">Length (shortest first)</option>
      <option value="length-long">Length (longest first)</option>
      <option value="noun-match">Nouns Only</option>
      <option value="verb-match">Verbs Only</option>
      <option value="modifier-match">Modifiers Only</option>
      <option value="particle-match">Particles Only</option>
      <option value="numeral-match">Numerals Only</option>
      <option value="interjection-match">Interjections Only</option>
      <option value="adposition-match">Adpositions Only</option>
      <option value="best-match">Best Match</option>
    </select>
  </div>
  {#if errors.length !== 0}
    <b>Some words are invalid!</b>
    <ul>
      {#each errors as error}
        <li>{error.word}: {error.ipa.message}</li>
      {/each}
    </ul>
  {/if}
</main>
<section class="cards-wrap">
  <div id="word-grid" class="grid" role="list">
    <!-- optionally add grid-scroll -->
    {#if sorted.length === 0}
      <div class="empty">
        {#if [":3", ":3c"].includes(normalized)}
        nya mrow purr uwu
        {:else}
        No matches. Try a different search.
        {/if}
      </div>
    {:else}
      {#each sorted as w (w.word)}
        <div
          class="card"
          role="listitem"
          tabindex="0"
          aria-label={`Word ${w.displayWord}`}
          aria-controls="word-panel"
          on:click={() => open(w)}
          on:keydown={(e) => onCardKeydown(e, w)}
        >
          <div class="accent accent-{w.type}" aria-hidden="true"></div>
          <header class="title">{w.displayWord}</header>
          {#if w.definition}
            <p class="definition">
              ({w.type === "numeral" ? "#" : w.type[0]}) {w.definition}
            </p>
          {/if}
        </div>
      {/each}
    {/if}
  </div>
</section>

{#if selected}
  <div class="overlay" aria-hidden="true" on:click={close}></div>
  <aside
    id="word-panel"
    class="drawer"
    role="dialog"
    aria-modal="true"
    aria-labelledby="panel-title"
    bind:this={drawerEl}
    tabindex="-1"
  >
    <div class="drawer-handle" aria-hidden="true"></div>
    <header class="drawer-header">
      <h2 id="panel-title">{selected.displayWord}</h2>
      {#if selected.type === "verb" || selected.type === "noun" || selected.type === "modifier"}
        <button
          class="aside-top-button"
          on:click={() => {
            const { type, displayWord } = selected;
            close();
            setTimeout(() => {
              navigate(`/inflect?type=${type}&q=${displayWord}`);
            });
          }}>Inflect</button
        >
      {/if}
      <button
        class="close aside-top-button"
        on:click={close}
        aria-label="Close details">×</button
      >
    </header>
    <section class="drawer-body">
      <div class="detail-display">
        <h1 class="detail-word">{selected.displayWord}</h1>
        <p class="detail-syllables">
          {selected.ipa?.ipa ?? "this word is invalid"}
        </p>
        <p class="detail-type">{selected.type}</p>
        <label class="detail-label">definition</label>
        <p class="detail-text">{selected.definition}</p>
        {#if selected.usage}
          <label class="detail-label">usage</label>
          <p class="detail-text">{selected.usage}</p>{/if}
        <label class="detail-label">etymology</label>
        {#snippet etymologyDescription(etymology)}
          {#if etymology === false}
            a priori
          {:else if Array.isArray(etymology)}
            {#if Array.isArray(etymology[1])}
              {#if etymology.length > 2 && etymology[2]}
                <span class="ety-lang">{etymology[0]}</span>
                <em class="ety-word">{etymology[1][0]}</em>
                (<span class="ety-roman">{etymology[1][1]}</span>) "<span class='ety-gloss'>{etymology[2]}</span>"
              {:else}
                <span class="ety-lang">{etymology[0]}</span>
                <em class="ety-word">{etymology[1][0]}</em>
                (<span class="ety-roman">{etymology[1][1]}</span>)
              {/if}
            {:else if etymology.length > 2 && etymology[2]}
              <span class="ety-lang">{etymology[0]}</span>
              <em class="ety-word">{etymology[1]}</em>
              "<span class="ety-gloss">{etymology[2]}</span>"
            {:else if etymology[0] === "Batelu" && etymology[1].includes(" ")}
              {@const words = etymology[1].split(" ")}
              {#each words as word, i}
                {@render etymologyDescription(["Batelu", word])}
                {#if i !== words.length - 1}
                  <br />
                {/if}
              {/each}
            {:else if etymology[0] === "Batelu"}
              {@const word = words.find((word) => word.word === etymology[1])}
              <span class="ety-lang">{etymology[0]}</span>
              <em class="ety-word"><a href={(() => {
                    const url = new URL(location.href);
                    url.searchParams.set("word", etymology[1]);
                    return url.toString();
                  })()} on:click={(e) => {
                    e.preventDefault();
                    open(word);
                  }}>{etymology[1]}</a></em>
              {#if word}
                "<span class="ety-gloss">{word.definition.split(", ")[0]}</span>"
                <br />
                <span class="ety-further">
                  ← {@render etymologyDescription(word.etymology)}
                </span>
              {:else}
                <span class="ety-error">word not found</span>
              {/if}
            {:else}
              <span class="ety-lang">{etymology[0]}</span>
              <em class="ety-word">{etymology[1]}</em>
            {/if}
          {:else}
            {etymology}
          {/if}
        {/snippet}
        <p class="detail-text">
          {@render etymologyDescription(selected.etymology)}
        </p>

        {#if selected.date}
          <label class="detail-label">creation date</label>
          <p class="detail-text">{formatDate(selected.date)}</p>
        {/if}
      </div>
    </section>
  </aside>
{/if}

<style>
  .main-header {
    display: flex;
    align-items: baseline;
  }
  .main-header h1 {
    flex: 1;
  }

  /* Search (layout fix for count alignment) */
  .search {
    display: grid;
    grid-template-columns: 1fr auto;
    align-items: center;
    gap: 0.6rem;
    margin-block: 0.25rem 0.75rem;
  }

  .search input {
    padding: 0.75rem 1rem;
    min-height: 44px; /* consistent height with count */
    border-radius: 12px;
    background: #0f0f0f;
    border: 1px solid var(--border);
    color: var(--fg);
    outline: none;
    transition:
      border-color 0.15s ease,
      box-shadow 0.15s ease;
  }

  .search input:focus {
    border-color: color-mix(in srgb, var(--accent) 45%, var(--border));
    box-shadow: 0 0 0 3px color-mix(in srgb, var(--accent) 28%, transparent);
  }

  .search .count {
    min-width: 2.6rem;
    height: 44px; /* match input height */
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 0 0.7rem; /* horizontal padding only */
    border: 1px solid var(--border);
    border-radius: 10px;
    color: var(--muted);
    background: #0f0f0f;
    line-height: 1; /* avoid vertical offset */
  }

  /* Sort control */
  .sort-control {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    margin-bottom: 0.75rem;
  }

  .sort-control label {
    color: var(--muted);
    font-weight: 600;
    font-size: 0.95rem;
  }

  .sort-control select {
    padding: 0.5rem 0.75rem;
    border-radius: 10px;
    background: #0f0f0f;
    border: 1px solid var(--border);
    color: var(--fg);
    outline: none;
    cursor: pointer;
    transition:
      border-color 0.15s ease,
      box-shadow 0.15s ease;
    font-size: 0.95rem;
  }

  .sort-control select:focus {
    border-color: color-mix(in srgb, var(--accent) 45%, var(--border));
    box-shadow: 0 0 0 3px color-mix(in srgb, var(--accent) 28%, transparent);
  }

  .sort-control select:hover {
    border-color: color-mix(in srgb, var(--accent) 35%, var(--border));
  }

  /* Wide, backgroundless wrapper (outside .container) */
  .cards-wrap {
    width: clamp(320px, 96vw, 1240px);
    margin: 0.75rem auto 2rem;
    padding: 0;
  }

  /* Scrollable grid */
  .grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
    gap: 0.9rem;
  }

  .scroll-region {
    max-height: min(64vh, 740px);
    overflow: auto;
    padding-right: 2px;
    scroll-behavior: smooth;
    border-radius: 12px;
  }

  .scroll-region:focus {
    outline: none;
    box-shadow: 0 0 0 3px color-mix(in srgb, var(--accent) 22%, transparent);
  }

  .empty {
    grid-column: 1 / -1;
    color: var(--muted);
    text-align: center;
    padding: 1rem 0.5rem;
    border: 1px dashed var(--border);
    border-radius: 12px;
    background: color-mix(in srgb, var(--fg) 2%, transparent);
  }

  /* Cards */
  .card {
    position: relative;
    display: grid;
    grid-template-rows: auto 1fr;
    gap: 0.5rem;

    border: 1px solid var(--border);
    border-radius: 14px;
    background:
      linear-gradient(180deg, rgba(255, 255, 255, 0.03), transparent 40%),
      #101010;
    padding: 0.8rem 0.9rem;

    box-shadow:
      0 1px 0 rgba(255, 255, 255, 0.03) inset,
      0 8px 20px rgba(0, 0, 0, 0.2);
    transition:
      transform 120ms ease,
      border-color 160ms ease,
      box-shadow 160ms ease,
      background-color 160ms ease;

    cursor: pointer;
    /* Same size, even less height */
    height: clamp(110px, 10.8rem, 150px);
    box-sizing: border-box;
  }

  .card .accent {
    position: absolute;
    inset: 0 0 auto 0;
    height: 3px;
    border-top-left-radius: 14px;
    border-top-right-radius: 14px;
    background: color-mix(in srgb, var(--accent) 90%, #fff 0%);
    opacity: 0.95;
    pointer-events: none;
  }

  :root {
    --accent-noun: color-mix(in srgb, #52cbff 90%, #fff 0%);
    --accent-verb: color-mix(in srgb, #ff5252 90%, #fff 0%);
    --accent-modifier: color-mix(in srgb, #6cff52 90%, #fff 0%);
    --accent-particle: color-mix(in srgb, #8052ff 90%, #fff 0%);
    --accent-numeral: #ffcc33;
    --accent-interjection: color-mix(in srgb, #fff652 90%, #fff 0%);
    --accent-adposition: color-mix(in srgb, #ff52e2 90%, #fff 0%);
  }

  .card .accent.accent-noun {
    background: color-mix(in srgb, var(--accent-noun) 90%, #fff 0%);
  }

  .card .accent.accent-verb {
    background: color-mix(in srgb, var(--accent-verb) 90%, #fff 0%);
  }

  .card .accent.accent-modifier {
    background: color-mix(in srgb, var(--accent-modifier) 90%, #fff 0%);
  }

  .card .accent.accent-particle {
    background: color-mix(in srgb, var(--accent-particle) 90%, #fff 0%);
  }

  .card .accent.accent-numeral {
    background: color-mix(in srgb, var(--accent-numeral) 90%, #fff 0%);
  }

  .card .accent.accent-interjection {
    background: color-mix(in srgb, var(--accent-interjection) 90%, #fff 0%);
  }

  .card .accent.accent-adposition {
    background: color-mix(in srgb, var(--accent-adposition) 90%, #fff 0%);
  }

  .card:hover,
  .card:focus-visible {
    transform: translateY(-2px);
    border-color: color-mix(in srgb, var(--accent) 35%, var(--border));
    box-shadow:
      0 1px 0 rgba(255, 255, 255, 0.04) inset,
      0 12px 28px rgba(0, 0, 0, 0.26);
  }

  .title {
    text-align: center;
    font-weight: 800;
    letter-spacing: 0.2px;
    font-size: clamp(1.2rem, 3.2vw, 1.45rem);
    color: var(--fg);
    margin-top: 0.05rem;
  }

  .definition {
    color: var(--muted);
    font-size: 0.96rem;
    line-height: 1.4;
    display: -webkit-box;
    line-clamp: 3;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  /* Remove incorrect early overlay layout if present (ensure overlay is full-screen) */

  /* Overlay (full-screen backdrop) */
  .overlay {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.5);
    backdrop-filter: blur(2px);
    z-index: 60;
  }

  /* Keyframes (top-level for compatibility) */
  @keyframes popIn {
    from {
      transform: translate(-50%, calc(-50% + 14px));
      opacity: 0;
    }
    to {
      transform: translate(-50%, -50%);
      opacity: 1;
    }
  }

  @keyframes slideUp {
    from {
      transform: translateX(-50%) translateY(18px);
      opacity: 0;
    }
    to {
      transform: translateX(-50%) translateY(0);
      opacity: 1;
    }
  }

  /* Details panel: default = centered popup (desktop/tablet) */
  .drawer {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: clamp(320px, 92vw, 980px);
    max-height: 96vh; /* taller */
    min-height: 60vh; /* bigger minimum height */
    background: #101010;
    border: 1px solid var(--border);
    border-radius: 18px;
    box-shadow:
      0 24px 70px rgba(0, 0, 0, 0.55),
      0 1px 0 rgba(255, 255, 255, 0.04) inset;
    z-index: 70;
    display: grid;
    grid-template-rows: auto 1fr;
    padding: 1rem 1.2rem 1.2rem;
    box-sizing: border-box; /* prevent layout overflow */
    overflow: hidden; /* clip internals (no horiz scroll) */
    animation: popIn 160ms ease-out;
  }

  .drawer-handle {
    display: none; /* only visible on mobile sheet */
  }

  .drawer-header {
    display: flex;
    align-items: stretch;
    gap: 0.5rem;
  }

  .drawer-header h2 {
    flex: 1;
    margin: 0.2rem 0 0.1rem;
    font-size: clamp(1.25rem, 2.2vw, 1.7rem);
    letter-spacing: 0.2px;
  }

  .aside-top-button {
    appearance: none;
    background: transparent;
    border: 1px solid var(--border);
    color: var(--fg);
    border-radius: 12px;
    padding: 0.35rem 0.65rem;
    line-height: 1;
    cursor: pointer;
    width: auto;
  }
  .close {
    font-size: 1.6rem; /* bigger X */
  }

  .drawer-body {
    overflow-y: auto; /* scroll vertically when needed */
    overflow-x: hidden; /* never scroll horizontally */
    margin-top: 0.25rem;
    -webkit-overflow-scrolling: touch; /* smooth scroll on iOS */
    overflow-wrap: anywhere; /* avoid horizontal overflow from long text */
    padding-right: 2px; /* ensure scrollbar doesn't overlap text */
  }

  .meta-row {
    display: grid;
    grid-template-columns: 110px 1fr;
    gap: 0.6rem;
    align-items: start;
    padding: 0.5rem 0.6rem;
    border: 1px solid var(--border);
    border-radius: 12px;
    background: color-mix(in srgb, var(--fg) 2%, transparent);
  }

  .meta-label {
    color: var(--muted);
    font-weight: 600;
  }

  .meta-value {
    color: var(--fg);
  }

  /* Popup content styling inspired layout */
  .detail-display {
    display: block;
    width: min(840px, 100%);
    margin: 0 auto;
    padding: 0.5rem 0.5rem 0.75rem;
    box-sizing: border-box;
  }

  .detail-word {
    font-family:
      system-ui,
      -apple-system,
      Segoe UI,
      Roboto,
      Helvetica,
      Arial,
      "Apple Color Emoji",
      "Segoe UI Emoji";
    font-weight: 800;
    font-size: clamp(2.4rem, 6vw, 3.6rem);
    line-height: 1.08;
    margin: 0 0 0.35rem 0;
    letter-spacing: 0.2px;
    color: var(--fg);
  }

  .detail-syllables {
    font-size: 1.1rem;
    color: var(--muted);
    margin: 0.3rem 0 0.9rem;
  }

  .detail-type {
    font-size: 1.1rem;
    color: var(--muted);
    margin: 0.3rem 0 0.9rem;
    font-style: italic;
  }

  .detail-label {
    display: block;
    font-weight: 700;
    color: var(--muted);
    margin-top: 1rem;
    letter-spacing: 0.2px;
  }

  .detail-text {
    margin: 0.45rem 0 0.35rem;
    font-size: 1.18rem;
    line-height: 1.65;
    color: var(--fg);
  }

  /* Etymology accents */
  .ety-lang {
    font-weight: 700;
  }
  .ety-word {
    font-style: italic;
  }
  .ety-roman {
    opacity: 0.95;
  }
  .ety-gloss {
    opacity: 0.92;
  }
  .ety-further {
    display: inline-block;
    margin-left: 20px;
  }
  .ety-error {
    color: var(--accent-error);
  }

  /* Outlined box specifically for the definition content */
  .def-box {
    border: 2px solid #ffffff; /* visible white outline */
    border-radius: 12px;
    padding: 0.75rem 0.85rem;
    background: color-mix(in srgb, var(--fg) 2.5%, transparent);
    box-shadow: 0 2px 14px rgba(0, 0, 0, 0.25);
    margin-top: 0.35rem;
  }

  /* Mobile: turn into bottom sheet/drawer */
  @media (max-width: 640px) {
    .search {
      grid-template-columns: 1fr auto;
      gap: 0.5rem;
    }
    .search input {
      min-height: 48px; /* larger touch target on mobile */
    }
    .search .count {
      height: 48px; /* match input */
      min-width: 2.8rem;
    }

    .cards-wrap {
      width: 100%;
      padding-inline: 0;
      margin-inline: 0;
    }

    .grid {
      grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
      gap: 0.75rem;
    }

    .card {
      border-radius: 12px;
      padding: 0.7rem 0.8rem;
      /* Consistent height on mobile too (even less) */
      height: clamp(100px, 12.15rem, 140px);
    }

    .card .accent {
      border-top-left-radius: 12px;
      border-top-right-radius: 12px;
    }

    .detail-display {
      width: 100%;
    }

    .detail-word {
      font-size: clamp(2rem, 9vw, 2.6rem);
    }

    .detail-text {
      font-size: 1.08rem;
    }

    .drawer-handle {
      display: block;
      width: 48px;
      height: 5px;
      border-radius: 999px;
      background: color-mix(in srgb, var(--fg) 14%, transparent);
      margin: 0 auto 0.6rem;
    }

    .meta-row {
      grid-template-columns: 90px 1fr;
    }
  }
</style>
