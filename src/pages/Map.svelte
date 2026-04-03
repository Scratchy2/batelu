<script>
  import MapImage from "./MapImage.svelte";
  import words from "./words.json";
  import {
    languagesToCountries,
    countriesToLanguages,
  } from "../lib/languagesToCountries";
  import { onMount } from "svelte";

  const missingLanguages = new Set();
  const countryAmounts = new Map();
  const countryWords = new Map();
  let maxAmount = 0;
  words.forEach((word) => {
    if (!Array.isArray(word.etymology) || typeof word.etymology[0] !== "string")
      return;
    const language = word.etymology[0];
    if (!Object.hasOwn(languagesToCountries, language)) {
      missingLanguages.add(language);
    }
    const countries = languagesToCountries[language];
    if (!countries) return;
    countries.forEach((country) => {
      const amount = countryAmounts.get(country);
      const newAmount = (amount ?? 0) + 1;
      if (newAmount > maxAmount) maxAmount = newAmount;
      countryAmounts.set(country, newAmount);

      if (countryWords.has(country)) {
        countryWords.get(country).push(word);
      } else {
        countryWords.set(country, [word]);
      }
    });
  });

  let svg;
  onMount(() => {
    countryAmounts.forEach((amount, country) => {
      const element = svg.element().getElementById(country);
      if (!element) {
        console.log(country);
        return;
      }
      const distribution = countryAmounts.get(country) / maxAmount;
      const percentage = String(distribution * 100) + "%";
      const color = `rgba(255, 0, 0, ${percentage})`;
      element.style.fill = color;
      element.querySelectorAll("path").forEach((path) => {
        path.style.fill = color;
      });
    });
  });

  let currentCountry = null;
  let currentCountryName = null;
  let mouseX = null;
  let mouseY = null;
  $: currentCountryWords = currentCountry
    ? countryWords.get(currentCountry)
    : null;
  $: displayCurrentCountryWords = currentCountryWords?.reduce((acc, word) => {
    const displayWord = word.word.replace(/\d+$/, "");
    const summary = word.definition.split(", ")[0];
    const newWord = { ...word, displayWord, summary };
    if (Object.hasOwn(acc, word.etymology[0])) {
      acc[word.etymology[0]].push(newWord);
    } else {
      acc[word.etymology[0]] = [newWord];
    }
    return acc;
  }, {});
  const onMouseOver = (e) => {
    let title = null;
    for (
      let parent = e.target;
      parent.nodeName !== "svg";
      parent = parent.parentElement
    ) {
      title ??= parent.dataset.title;
      const maybeCountry = parent.id;
      if (countryWords.has(maybeCountry)) {
        currentCountry = maybeCountry;
        currentCountryName = title;
        mouseX = e.clientX;
        mouseY = e.clientY;
        return;
      }
    }
    currentCountry = null;
    currentCountryName = null;
    mouseX = null;
    mouseY = null;
  };
</script>

<div class="container">
  <h1>Map</h1>
  <MapImage onmousemove={onMouseOver} bind:this={svg} />
  {#if currentCountryName !== null && mouseX !== null && mouseY !== null && displayCurrentCountryWords !== null}
    <div class="hoverer" style={`--left: ${mouseX}px; --top: ${mouseY}px`}>
      <strong>{currentCountryName}</strong><br />
      <div class="languages">
        {#each Object.entries(displayCurrentCountryWords) as [language, words]}
          <div>
            <strong>{language}</strong><br />
            {#each words as word}
              {word.word} - {word.summary}<br />
            {/each}
          </div>
        {/each}
      </div>
    </div>
  {/if}
</div>

<style>
  .hoverer {
    background-color: black;
    position: absolute;
    left: var(--left);
    top: var(--top);
    padding: 5px 10px;
    pointer-events: none;
  }
  .languages {
    display: flex;
    gap: 1rem;
  }
</style>
