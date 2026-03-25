<script>
  import { onMount } from "svelte";

  let wordType = "verb";
  let initialized = false;
  let word = "";
  const computeVerbWord = (word, wordType) => {
    if (wordType !== "verb") return null;
    const match = word.match(/^(.*)([aeiouy])re$/);
    if (!match) return null;
    return { stem: match[1], vowel: match[2] };
  };
  $: verbWord = computeVerbWord(word, wordType);
  $: wordEntryError = word !== "" && wordType === "verb" && !verbWord;

  onMount(() => {
    const params = new URLSearchParams(window.location.search);
    if (params.has("type")) {
      const paramWordType = params.get("type");
      if (
        paramWordType === "noun" ||
        paramWordType === "verb" ||
        paramWordType === "pronoun"
      ) {
        wordType = paramWordType;
      }
    }
    if (params.has("q")) {
      word = params.get("q");
    }
    initialized = true;
  });

  $: {
    if (initialized && typeof window !== "undefined") {
      const url = new URL(window.location.href);
      url.searchParams.set("type", wordType);
      if (
        word &&
        !(wordType === "verb" && !verbWord) &&
        wordType !== "pronoun"
      ) {
        url.searchParams.set("q", word);
      } else {
        url.searchParams.delete("q");
      }
      window.history.replaceState({}, "", url.toString());
    }
  }
</script>

<main class="container">
  <h1>Inflections</h1>
  <div class="header">
    <button
      class={{ "btn-primary": wordType === "verb" }}
      on:click={() => (wordType = "verb")}>Verbs</button
    >
    <button
      class={{ "btn-primary": wordType === "noun" }}
      on:click={() => (wordType = "noun")}>Nouns</button
    >
    <button
      class={{ "btn-primary": wordType === "pronoun" }}
      on:click={() => (wordType = "pronoun")}>Pronouns</button
    >
    {#if wordType !== "pronoun"}
      <input
        type="text"
        placeholder="Word"
        class={{
          "word-entry": true,
          "word-entry-error": wordEntryError,
        }}
        bind:value={word}
        aria-invalid={wordEntryError}
      />
    {/if}
  </div>
  {#if wordType === "verb"}
    {#snippet conjugate(ending)}
      {#if verbWord}
        <span class="inflections-secondary">{verbWord.stem}</span
        >{ending.replace("V", verbWord.vowel)}
      {:else}
        -{ending}
      {/if}
    {/snippet}
    <div>
      <table>
        <thead>
          <tr>
            <th class="table-name" colspan="3">1st Person Singular</th>
            <th>Simple</th>
            <th>Continous</th>
            <th>Perfect</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td rowspan="6" class="no-bottom-border horizontal-header">
              <b>Animate</b><br /><b>Inanimate </b>
            </td>
            <td rowspan="3" class="td-bottom horizontal-header"
              ><b>Intentional</b></td
            >
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("a")}</td>
            <td>{@render conjugate("ad")}</td>
            <td>{@render conjugate("ak")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("e")}</td>
            <td>{@render conjugate("ed")}</td>
            <td>{@render conjugate("ek")}</td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("i")}</td>
            <td>{@render conjugate("id")}</td>
            <td>{@render conjugate("ik")}</td>
          </tr>
          <tr>
            <td rowspan="3" class="no-bottom-border horizontal-header"
              ><b>Unintentional</b></td
            >
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("o")}</td>
            <td>{@render conjugate("od")}</td>
            <td>{@render conjugate("ok")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("u")}</td>
            <td>{@render conjugate("ud")}</td>
            <td>{@render conjugate("uk")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("y")}</td>
            <td>{@render conjugate("yd")}</td>
            <td>{@render conjugate("yk")}</td>
          </tr>
        </tbody>
      </table>
      <table>
        <thead>
          <tr>
            <th class="table-name" colspan="3">1st Person Plural</th>
            <th>Simple</th>
            <th>Continous</th>
            <th>Perfect</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td rowspan="6" class="no-bottom-border horizontal-header">
              <b>Animate</b><br /><b>Inanimate </b>
            </td>
            <td rowspan="3" class="td-bottom horizontal-header"
              ><b>Intentional</b></td
            >
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("at")}</td>
            <td>{@render conjugate("ag")}</td>
            <td>{@render conjugate("ar")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("et")}</td>
            <td>{@render conjugate("eg")}</td>
            <td>{@render conjugate("er")}</td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("it")}</td>
            <td>{@render conjugate("ig")}</td>
            <td>{@render conjugate("ir")}</td>
          </tr>
          <tr>
            <td rowspan="3" class="no-bottom-border horizontal-header"
              ><b>Unintentional</b></td
            >
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("ot")}</td>
            <td>{@render conjugate("og")}</td>
            <td>{@render conjugate("or")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("ut")}</td>
            <td>{@render conjugate("ug")}</td>
            <td>{@render conjugate("ur")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("yt")}</td>
            <td>{@render conjugate("yg")}</td>
            <td>{@render conjugate("yr")}</td>
          </tr>
        </tbody>
      </table>
      <table>
        <thead>
          <tr>
            <th class="table-name" colspan="3">2nd Person Singular</th>
            <th>Simple</th>
            <th>Continous</th>
            <th>Perfect</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td rowspan="6" class="horizontal-header td-bottom-strong">
              <b>Animate</b>
            </td>
            <td rowspan="3" class="td-bottom horizontal-header">
              <b>Intentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("Vva")}</td>
            <td>{@render conjugate("Vfa")}</td>
            <td>{@render conjugate("Vgha")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("Vve")}</td>
            <td>{@render conjugate("Vfe")}</td>
            <td>{@render conjugate("Vghe")}</td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("Vvi")}</td>
            <td>{@render conjugate("Vfi")}</td>
            <td>{@render conjugate("Vghi")}</td>
          </tr>
          <tr>
            <td rowspan="3" class="horizontal-header td-bottom-strong">
              <b>Unintentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("Vvo")}</td>
            <td>{@render conjugate("Vfo")}</td>
            <td>{@render conjugate("Vgho")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("Vvu")}</td>
            <td>{@render conjugate("Vvu")}</td>
            <td>{@render conjugate("Vghu")}</td>
          </tr>
          <tr class="tr-bottom-strong">
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("Vvy")}</td>
            <td>{@render conjugate("Vvy")}</td>
            <td>{@render conjugate("Vghy")}</td>
          </tr>
          <tr>
            <td rowspan="6" class="horizontal-header">
              <b>Inanimate</b>
            </td>
            <td rowspan="3" class="td-bottom horizontal-header">
              <b>Intentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("Vab")}</td>
            <td>{@render conjugate("Vaj")}</td>
            <td>{@render conjugate("Vazh")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("Veb")}</td>
            <td>{@render conjugate("Vej")}</td>
            <td>{@render conjugate("Vezh")}</td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("Vib")}</td>
            <td>{@render conjugate("Vij")}</td>
            <td>{@render conjugate("Vizh")}</td>
          </tr>
          <tr>
            <td rowspan="3" class="no-bottom-border horizontal-header"
              ><b>Unintentional</b></td
            >
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("Vob")}</td>
            <td>{@render conjugate("Voj")}</td>
            <td>{@render conjugate("Vozh")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("Vub")}</td>
            <td>{@render conjugate("Vuj")}</td>
            <td>{@render conjugate("Vuzh")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("Vyb")}</td>
            <td>{@render conjugate("Vyj")}</td>
            <td>{@render conjugate("Vyzh")}</td>
          </tr>
        </tbody>
      </table>
      <table>
        <thead>
          <tr>
            <th class="table-name" colspan="3">2nd Person Plural</th>
            <th>Simple</th>
            <th>Continous</th>
            <th>Perfect</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td rowspan="6" class="horizontal-header td-bottom-strong">
              <b>Animate</b>
            </td>
            <td rowspan="3" class="td-bottom horizontal-header">
              <b>Intentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("Vvas")}</td>
            <td>{@render conjugate("Vfas")}</td>
            <td>{@render conjugate("Vghas")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("Vves")}</td>
            <td>{@render conjugate("Vfes")}</td>
            <td>{@render conjugate("Vghes")}</td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("Vvis")}</td>
            <td>{@render conjugate("Vfis")}</td>
            <td>{@render conjugate("Vghis")}</td>
          </tr>
          <tr>
            <td rowspan="3" class="horizontal-header td-bottom-strong">
              <b>Unintentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("Vvos")}</td>
            <td>{@render conjugate("Vfos")}</td>
            <td>{@render conjugate("Vghos")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("Vvus")}</td>
            <td>{@render conjugate("Vvus")}</td>
            <td>{@render conjugate("Vghus")}</td>
          </tr>
          <tr class="tr-bottom-strong">
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("Vvys")}</td>
            <td>{@render conjugate("Vvys")}</td>
            <td>{@render conjugate("Vghys")}</td>
          </tr>
          <tr>
            <td rowspan="6" class="horizontal-header">
              <b>Inanimate</b>
            </td>
            <td rowspan="3" class="td-bottom horizontal-header">
              <b>Intentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("Vaby")}</td>
            <td>{@render conjugate("Vajy")}</td>
            <td>{@render conjugate("Vazhy")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("Veby")}</td>
            <td>{@render conjugate("Vejy")}</td>
            <td>{@render conjugate("Vezhy")}</td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("Viby")}</td>
            <td>{@render conjugate("Vijy")}</td>
            <td>{@render conjugate("Vizhy")}</td>
          </tr>
          <tr>
            <td rowspan="3" class="no-bottom-border horizontal-header"
              ><b>Unintentional</b></td
            >
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("Voby")}</td>
            <td>{@render conjugate("Vojy")}</td>
            <td>{@render conjugate("Vozhy")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("Vuby")}</td>
            <td>{@render conjugate("Vujy")}</td>
            <td>{@render conjugate("Vuzhy")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("Vyby")}</td>
            <td>{@render conjugate("Vyjy")}</td>
            <td>{@render conjugate("Vyzhy")}</td>
          </tr>
        </tbody>
      </table>
      <table>
        <thead>
          <tr>
            <th class="table-name" colspan="3">3rd Person Singular</th>
            <th>Simple</th>
            <th>Continous</th>
            <th>Perfect</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td rowspan="6" class="horizontal-header td-bottom-strong">
              <b>Animate</b>
            </td>
            <td rowspan="3" class="td-bottom horizontal-header">
              <b>Intentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("Vra")}</td>
            <td>{@render conjugate("Vra")}</td>
            <td>{@render conjugate("Vba")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("Vren")}</td>
            <td>{@render conjugate("Vren")}</td>
            <td>{@render conjugate("Vbe")}</td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("Vri")}</td>
            <td>{@render conjugate("Vri")}</td>
            <td>{@render conjugate("Vbi")}</td>
          </tr>
          <tr>
            <td rowspan="3" class="horizontal-header td-bottom-strong">
              <b>Unintentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("Vro")}</td>
            <td>{@render conjugate("Vro")}</td>
            <td>{@render conjugate("Vbo")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("Vru")}</td>
            <td>{@render conjugate("Vru")}</td>
            <td>{@render conjugate("Vbu")}</td>
          </tr>
          <tr class="tr-bottom-strong">
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("Vry")}</td>
            <td>{@render conjugate("Vry")}</td>
            <td>{@render conjugate("Vby")}</td>
          </tr>
          <tr>
            <td rowspan="6" class="horizontal-header">
              <b>Inanimate</b>
            </td>
            <td rowspan="3" class="td-bottom horizontal-header">
              <b>Intentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("Vra")}</td>
            <td>{@render conjugate("Vra")}</td>
            <td>{@render conjugate("Vba")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("Vren")}</td>
            <td>{@render conjugate("Vren")}</td>
            <td>{@render conjugate("Vbe")}</td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("Vri")}</td>
            <td>{@render conjugate("Vri")}</td>
            <td>{@render conjugate("Vbi")}</td>
          </tr>
          <tr>
            <td rowspan="3" class="no-bottom-border horizontal-header"
              ><b>Unintentional</b></td
            >
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("Vro")}</td>
            <td>{@render conjugate("Vro")}</td>
            <td>{@render conjugate("Vbo")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("Vru")}</td>
            <td>{@render conjugate("Vru")}</td>
            <td>{@render conjugate("Vbu")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("Vry")}</td>
            <td>{@render conjugate("Vry")}</td>
            <td>{@render conjugate("Vby")}</td>
          </tr>
        </tbody>
      </table>
      <table>
        <thead>
          <tr>
            <th class="table-name" colspan="3">3rd Person Plural</th>
            <th>Simple</th>
            <th>Continous</th>
            <th>Perfect</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td rowspan="6" class="horizontal-header td-bottom-strong">
              <b>Animate</b>
            </td>
            <td rowspan="3" class="td-bottom horizontal-header">
              <b>Intentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("Vzwa")}</td>
            <td>{@render conjugate("Vlwa")}</td>
            <td>{@render conjugate("Vbwa")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("Vzwe")}</td>
            <td>{@render conjugate("Vlwe")}</td>
            <td>{@render conjugate("Vbwe")}</td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("Vzwi")}</td>
            <td>{@render conjugate("Vlwi")}</td>
            <td>{@render conjugate("Vbwi")}</td>
          </tr>
          <tr>
            <td rowspan="3" class="horizontal-header td-bottom-strong">
              <b>Unintentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("Vzwo")}</td>
            <td>{@render conjugate("Vlwo")}</td>
            <td>{@render conjugate("Vbwo")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("Vzwu")}</td>
            <td>{@render conjugate("Vlwu")}</td>
            <td>{@render conjugate("Vbwu")}</td>
          </tr>
          <tr class="tr-bottom-strong">
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("Vzwy")}</td>
            <td>{@render conjugate("Vlwy")}</td>
            <td>{@render conjugate("Vbwy")}</td>
          </tr>
          <tr>
            <td rowspan="6" class="horizontal-header">
              <b>Inanimate</b>
            </td>
            <td rowspan="3" class="td-bottom horizontal-header">
              <b>Intentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("Vzwa")}</td>
            <td>{@render conjugate("Vlwa")}</td>
            <td>{@render conjugate("Vbwa")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("Vzwe")}</td>
            <td>{@render conjugate("Vlwe")}</td>
            <td>{@render conjugate("Vbwe")}</td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("Vzwi")}</td>
            <td>{@render conjugate("Vlwi")}</td>
            <td>{@render conjugate("Vbwi")}</td>
          </tr>
          <tr>
            <td rowspan="3" class="no-bottom-border horizontal-header">
              <b>Unintentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td>{@render conjugate("Vzwa")}</td>
            <td>{@render conjugate("Vlwa")}</td>
            <td>{@render conjugate("Vbwa")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td>{@render conjugate("Vzwe")}</td>
            <td>{@render conjugate("Vlwe")}</td>
            <td>{@render conjugate("Vbwe")}</td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Future</b></td>
            <td>{@render conjugate("Vzwi")}</td>
            <td>{@render conjugate("Vlwi")}</td>
            <td>{@render conjugate("Vbwi")}</td>
          </tr>
        </tbody>
      </table>
    </div>
  {:else if wordType === "noun"}
    <table class="noun-table">
      <thead>
        <tr>
          <th>Singular</th>
          <th>Plural</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>
            {#if word}
              <span class="inflections-secondary">{word}</span>
            {:else}
              <span class="inflections-secondary">(no change)</span>
            {/if}
          </td>
          <td>
            {#if word}
              cu<span class="inflections-secondary">{word}</span>
            {:else}
              cu-
            {/if}
          </td>
        </tr>
      </tbody>
    </table>
  {:else if wordType === "pronoun"}
    <table>
      <thead>
        <tr>
          <th class="table-name" rowspan="2" colspan="2">Pronouns</th>
          <th class="vertical-header" colspan="2">Singular</th>
          <th class="vertical-header" colspan="2">Plural</th>
        </tr>
        <tr>
          <th class="vertical-header">Personal</th>
          <th class="vertical-header">Possessive</th>
          <th class="vertical-header">Personal</th>
          <th class="vertical-header">Possessive</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td rowspan="2" class="td-bottom horizontal-header">
            <b>1st Person</b>
          </td>
          <td class="horizontal-header"><b>Day</b></td>
          <td>ja</td>
          <td>jami</td>
          <td>cuja</td>
          <td>cujami</td>
        </tr>
        <tr class="tr-bottom">
          <td class="horizontal-header"><b>Night</b></td>
          <td>jo</td>
          <td>jomi</td>
          <td>cujo</td>
          <td>cujomi</td>
        </tr>
        <tr>
          <td rowspan="2" class="td-bottom horizontal-header">
            <b>2nd Person</b>
          </td>
          <td class="horizontal-header"><b>Animate</b></td>
          <td>voze</td>
          <td>vozemi</td>
          <td>cuvoze</td>
          <td>cuvozemi</td>
        </tr>
        <tr class="tr-bottom">
          <td class="horizontal-header"><b>Inanimate</b></td>
          <td>vozi</td>
          <td>vozimi</td>
          <td>cuvozi</td>
          <td>cuvozimi</td>
        </tr>
        <tr>
          <td rowspan="2" class="td-bottom horizontal-header">
            <b>3rd Person</b>
          </td>
          <td class="horizontal-header"><b>Animate</b></td>
          <td>kal</td>
          <td>kalami</td>
          <td>cukal</td>
          <td>cukalami</td>
        </tr>
        <tr class="tr-bottom">
          <td class="horizontal-header"><b>Inanimate</b></td>
          <td>kel</td>
          <td>kelemi</td>
          <td>cukel</td>
          <td>cukelemi</td>
        </tr>
        <tr>
          <td rowspan="2" class="horizontal-header">
            <b>4th Person</b>
          </td>
          <td class="horizontal-header"><b>Animate</b></td>
          <td class="inflections-secondary">(no word yet)</td>
          <td class="inflections-secondary">(no word yet)</td>
          <td class="inflections-secondary">(no word yet)</td>
          <td class="inflections-secondary">(no word yet)</td>
        </tr>
        <tr>
          <td class="horizontal-header"><b>Inanimate</b></td>
          <td class="inflections-secondary">(no word yet)</td>
          <td class="inflections-secondary">(no word yet)</td>
          <td class="inflections-secondary">(no word yet)</td>
          <td class="inflections-secondary">(no word yet)</td>
        </tr>
      </tbody>
    </table>
  {/if}
</main>

<style>
  .header {
    display: flex;
    gap: 0.5rem;
  }
  .word-entry {
    --input-accent: var(--accent);
  }
  input.word-entry-error {
    --input-accent: var(--accent-error);
    border: 1px solid var(--accent-error);
  }
  table {
    margin-top: 2rem;
  }
  .table-name {
    font-size: 1.2rem;
    text-align: center;
  }
  .td-bottom,
  .tr-bottom td {
    border-bottom: 3px solid var(--border);
  }
  .td-bottom-strong,
  .tr-bottom-strong td {
    border-bottom: 5px solid var(--border);
  }
  .no-bottom-border {
    border-bottom: 0;
  }
  .horizontal-header {
    width: 0;
    white-space: nowrap;
    text-align: center;
  }
  .vertical-header {
    text-align: center;
  }
  .inflections-secondary {
    opacity: 0.5;
  }
  .noun-table td {
    width: 50%;
  }
</style>
