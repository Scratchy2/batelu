<script>
  import { onMount } from "svelte";
  import Conjugate from "./Conjugate.svelte";

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
            <td><Conjugate {verbWord} ending="a" /></td>
            <td><Conjugate {verbWord} ending="ad" /></td>
            <td><Conjugate {verbWord} ending="ak" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="e" /></td>
            <td><Conjugate {verbWord} ending="ed" /></td>
            <td><Conjugate {verbWord} ending="ek" /></td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="i" /></td>
            <td><Conjugate {verbWord} ending="id" /></td>
            <td><Conjugate {verbWord} ending="ik" /></td>
          </tr>
          <tr>
            <td rowspan="3" class="no-bottom-border horizontal-header"
              ><b>Unintentional</b></td
            >
            <td class="horizontal-header"><b>Past</b></td>
            <td><Conjugate {verbWord} ending="o" /></td>
            <td><Conjugate {verbWord} ending="od" /></td>
            <td><Conjugate {verbWord} ending="ok" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="u" /></td>
            <td><Conjugate {verbWord} ending="ud" /></td>
            <td><Conjugate {verbWord} ending="uk" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="y" /></td>
            <td><Conjugate {verbWord} ending="yd" /></td>
            <td><Conjugate {verbWord} ending="yk" /></td>
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
            <td><Conjugate {verbWord} ending="at" /></td>
            <td><Conjugate {verbWord} ending="ag" /></td>
            <td><Conjugate {verbWord} ending="ar" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="et" /></td>
            <td><Conjugate {verbWord} ending="eg" /></td>
            <td><Conjugate {verbWord} ending="er" /></td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="it" /></td>
            <td><Conjugate {verbWord} ending="ig" /></td>
            <td><Conjugate {verbWord} ending="ir" /></td>
          </tr>
          <tr>
            <td rowspan="3" class="no-bottom-border horizontal-header"
              ><b>Unintentional</b></td
            >
            <td class="horizontal-header"><b>Past</b></td>
            <td><Conjugate {verbWord} ending="ot" /></td>
            <td><Conjugate {verbWord} ending="og" /></td>
            <td><Conjugate {verbWord} ending="or" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="ut" /></td>
            <td><Conjugate {verbWord} ending="ug" /></td>
            <td><Conjugate {verbWord} ending="ur" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="yt" /></td>
            <td><Conjugate {verbWord} ending="yg" /></td>
            <td><Conjugate {verbWord} ending="yr" /></td>
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
            <td><Conjugate {verbWord} ending="Vva" /></td>
            <td><Conjugate {verbWord} ending="Vfa" /></td>
            <td><Conjugate {verbWord} ending="Vgha" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="Vve" /></td>
            <td><Conjugate {verbWord} ending="Vfe" /></td>
            <td><Conjugate {verbWord} ending="Vghe" /></td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="Vvi" /></td>
            <td><Conjugate {verbWord} ending="Vfi" /></td>
            <td><Conjugate {verbWord} ending="Vghi" /></td>
          </tr>
          <tr>
            <td rowspan="3" class="horizontal-header td-bottom-strong">
              <b>Unintentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td><Conjugate {verbWord} ending="Vvo" /></td>
            <td><Conjugate {verbWord} ending="Vfo" /></td>
            <td><Conjugate {verbWord} ending="Vgho" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="Vvu" /></td>
            <td><Conjugate {verbWord} ending="Vvu" /></td>
            <td><Conjugate {verbWord} ending="Vghu" /></td>
          </tr>
          <tr class="tr-bottom-strong">
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="Vvy" /></td>
            <td><Conjugate {verbWord} ending="Vvy" /></td>
            <td><Conjugate {verbWord} ending="Vghy" /></td>
          </tr>
          <tr>
            <td rowspan="6" class="horizontal-header">
              <b>Inanimate</b>
            </td>
            <td rowspan="3" class="td-bottom horizontal-header">
              <b>Intentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td><Conjugate {verbWord} ending="Vab" /></td>
            <td><Conjugate {verbWord} ending="Vaj" /></td>
            <td><Conjugate {verbWord} ending="Vazh" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="Veb" /></td>
            <td><Conjugate {verbWord} ending="Vej" /></td>
            <td><Conjugate {verbWord} ending="Vezh" /></td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="Vib" /></td>
            <td><Conjugate {verbWord} ending="Vij" /></td>
            <td><Conjugate {verbWord} ending="Vizh" /></td>
          </tr>
          <tr>
            <td rowspan="3" class="no-bottom-border horizontal-header"
              ><b>Unintentional</b></td
            >
            <td class="horizontal-header"><b>Past</b></td>
            <td><Conjugate {verbWord} ending="Vob" /></td>
            <td><Conjugate {verbWord} ending="Voj" /></td>
            <td><Conjugate {verbWord} ending="Vozh" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="Vub" /></td>
            <td><Conjugate {verbWord} ending="Vuj" /></td>
            <td><Conjugate {verbWord} ending="Vuzh" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="Vyb" /></td>
            <td><Conjugate {verbWord} ending="Vyj" /></td>
            <td><Conjugate {verbWord} ending="Vyzh" /></td>
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
            <td><Conjugate {verbWord} ending="Vvas" /></td>
            <td><Conjugate {verbWord} ending="Vfas" /></td>
            <td><Conjugate {verbWord} ending="Vghas" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="Vves" /></td>
            <td><Conjugate {verbWord} ending="Vfes" /></td>
            <td><Conjugate {verbWord} ending="Vghes" /></td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="Vvis" /></td>
            <td><Conjugate {verbWord} ending="Vfis" /></td>
            <td><Conjugate {verbWord} ending="Vghis" /></td>
          </tr>
          <tr>
            <td rowspan="3" class="horizontal-header td-bottom-strong">
              <b>Unintentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td><Conjugate {verbWord} ending="Vvos" /></td>
            <td><Conjugate {verbWord} ending="Vfos" /></td>
            <td><Conjugate {verbWord} ending="Vghos" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="Vvus" /></td>
            <td><Conjugate {verbWord} ending="Vvus" /></td>
            <td><Conjugate {verbWord} ending="Vghus" /></td>
          </tr>
          <tr class="tr-bottom-strong">
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="Vvys" /></td>
            <td><Conjugate {verbWord} ending="Vvys" /></td>
            <td><Conjugate {verbWord} ending="Vghys" /></td>
          </tr>
          <tr>
            <td rowspan="6" class="horizontal-header">
              <b>Inanimate</b>
            </td>
            <td rowspan="3" class="td-bottom horizontal-header">
              <b>Intentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td><Conjugate {verbWord} ending="Vaby" /></td>
            <td><Conjugate {verbWord} ending="Vajy" /></td>
            <td><Conjugate {verbWord} ending="Vazhy" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="Veby" /></td>
            <td><Conjugate {verbWord} ending="Vejy" /></td>
            <td><Conjugate {verbWord} ending="Vezhy" /></td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="Viby" /></td>
            <td><Conjugate {verbWord} ending="Vijy" /></td>
            <td><Conjugate {verbWord} ending="Vizhy" /></td>
          </tr>
          <tr>
            <td rowspan="3" class="no-bottom-border horizontal-header"
              ><b>Unintentional</b></td
            >
            <td class="horizontal-header"><b>Past</b></td>
            <td><Conjugate {verbWord} ending="Voby" /></td>
            <td><Conjugate {verbWord} ending="Vojy" /></td>
            <td><Conjugate {verbWord} ending="Vozhy" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="Vuby" /></td>
            <td><Conjugate {verbWord} ending="Vujy" /></td>
            <td><Conjugate {verbWord} ending="Vuzhy" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="Vyby" /></td>
            <td><Conjugate {verbWord} ending="Vyjy" /></td>
            <td><Conjugate {verbWord} ending="Vyzhy" /></td>
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
            <td><Conjugate {verbWord} ending="Vra" /></td>
            <td><Conjugate {verbWord} ending="Vra" /></td>
            <td><Conjugate {verbWord} ending="Vba" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="Vren" /></td>
            <td><Conjugate {verbWord} ending="Vren" /></td>
            <td><Conjugate {verbWord} ending="Vbe" /></td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="Vri" /></td>
            <td><Conjugate {verbWord} ending="Vri" /></td>
            <td><Conjugate {verbWord} ending="Vbi" /></td>
          </tr>
          <tr>
            <td rowspan="3" class="horizontal-header td-bottom-strong">
              <b>Unintentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td><Conjugate {verbWord} ending="Vro" /></td>
            <td><Conjugate {verbWord} ending="Vro" /></td>
            <td><Conjugate {verbWord} ending="Vbo" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="Vru" /></td>
            <td><Conjugate {verbWord} ending="Vru" /></td>
            <td><Conjugate {verbWord} ending="Vbu" /></td>
          </tr>
          <tr class="tr-bottom-strong">
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="Vry" /></td>
            <td><Conjugate {verbWord} ending="Vry" /></td>
            <td><Conjugate {verbWord} ending="Vby" /></td>
          </tr>
          <tr>
            <td rowspan="6" class="horizontal-header">
              <b>Inanimate</b>
            </td>
            <td rowspan="3" class="td-bottom horizontal-header">
              <b>Intentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td><Conjugate {verbWord} ending="Vra" /></td>
            <td><Conjugate {verbWord} ending="Vra" /></td>
            <td><Conjugate {verbWord} ending="Vba" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="Vren" /></td>
            <td><Conjugate {verbWord} ending="Vren" /></td>
            <td><Conjugate {verbWord} ending="Vbe" /></td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="Vri" /></td>
            <td><Conjugate {verbWord} ending="Vri" /></td>
            <td><Conjugate {verbWord} ending="Vbi" /></td>
          </tr>
          <tr>
            <td rowspan="3" class="no-bottom-border horizontal-header"
              ><b>Unintentional</b></td
            >
            <td class="horizontal-header"><b>Past</b></td>
            <td><Conjugate {verbWord} ending="Vro" /></td>
            <td><Conjugate {verbWord} ending="Vro" /></td>
            <td><Conjugate {verbWord} ending="Vbo" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="Vru" /></td>
            <td><Conjugate {verbWord} ending="Vru" /></td>
            <td><Conjugate {verbWord} ending="Vbu" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="Vry" /></td>
            <td><Conjugate {verbWord} ending="Vry" /></td>
            <td><Conjugate {verbWord} ending="Vby" /></td>
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
            <td><Conjugate {verbWord} ending="Vzwa" /></td>
            <td><Conjugate {verbWord} ending="Vlwa" /></td>
            <td><Conjugate {verbWord} ending="Vbwa" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="Vzwe" /></td>
            <td><Conjugate {verbWord} ending="Vlwe" /></td>
            <td><Conjugate {verbWord} ending="Vbwe" /></td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="Vzwi" /></td>
            <td><Conjugate {verbWord} ending="Vlwi" /></td>
            <td><Conjugate {verbWord} ending="Vbwi" /></td>
          </tr>
          <tr>
            <td rowspan="3" class="horizontal-header td-bottom-strong">
              <b>Unintentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td><Conjugate {verbWord} ending="Vzwo" /></td>
            <td><Conjugate {verbWord} ending="Vlwo" /></td>
            <td><Conjugate {verbWord} ending="Vbwo" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="Vzwu" /></td>
            <td><Conjugate {verbWord} ending="Vlwu" /></td>
            <td><Conjugate {verbWord} ending="Vbwu" /></td>
          </tr>
          <tr class="tr-bottom-strong">
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="Vzwy" /></td>
            <td><Conjugate {verbWord} ending="Vlwy" /></td>
            <td><Conjugate {verbWord} ending="Vbwy" /></td>
          </tr>
          <tr>
            <td rowspan="6" class="horizontal-header">
              <b>Inanimate</b>
            </td>
            <td rowspan="3" class="td-bottom horizontal-header">
              <b>Intentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td><Conjugate {verbWord} ending="Vzwa" /></td>
            <td><Conjugate {verbWord} ending="Vlwa" /></td>
            <td><Conjugate {verbWord} ending="Vbwa" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="Vzwe" /></td>
            <td><Conjugate {verbWord} ending="Vlwe" /></td>
            <td><Conjugate {verbWord} ending="Vbwe" /></td>
          </tr>
          <tr class="tr-bottom">
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="Vzwi" /></td>
            <td><Conjugate {verbWord} ending="Vlwi" /></td>
            <td><Conjugate {verbWord} ending="Vbwi" /></td>
          </tr>
          <tr>
            <td rowspan="3" class="no-bottom-border horizontal-header">
              <b>Unintentional</b>
            </td>
            <td class="horizontal-header"><b>Past</b></td>
            <td><Conjugate {verbWord} ending="Vzwa" /></td>
            <td><Conjugate {verbWord} ending="Vlwa" /></td>
            <td><Conjugate {verbWord} ending="Vbwa" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Present</b></td>
            <td><Conjugate {verbWord} ending="Vzwe" /></td>
            <td><Conjugate {verbWord} ending="Vlwe" /></td>
            <td><Conjugate {verbWord} ending="Vbwe" /></td>
          </tr>
          <tr>
            <td class="horizontal-header"><b>Future</b></td>
            <td><Conjugate {verbWord} ending="Vzwi" /></td>
            <td><Conjugate {verbWord} ending="Vlwi" /></td>
            <td><Conjugate {verbWord} ending="Vbwi" /></td>
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
