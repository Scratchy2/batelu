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
      const color = `color-mix(in oklch, rgb(255, 0, 0) ${percentage}, #c0c0c0)`;
      element.style.fill = color;
      element.querySelectorAll("path").forEach((path) => {
        path.style.fill = color;
      });
    });
  });

  let tooltipData = null;
  $: currentCountryWords = tooltipData
    ? countryWords.get(tooltipData.country)
    : null;
  $: tooltipCountryWords = currentCountryWords?.reduce((acc, word) => {
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
        tooltipData = {
          country: maybeCountry,
          name: title,
          mouseX: e.clientX + document.documentElement.scrollLeft,
          mouseY: e.clientY + document.documentElement.scrollTop,
        };
        return;
      }
    }
    tooltipData = null;
  };

  const inChunksOf = (arr, num) =>
    arr.reduce(
      (acc, i) => {
        if (acc[acc.length - 1].length === num) {
          acc.push([i]);
        } else {
          acc[acc.length - 1].push(i);
        }
        return acc;
      },
      [[]],
    );

  const attachment = (tooltipData) => (/** @type {HTMLElement} */ el) => {
    delete el.dataset.right;
    el.style.setProperty("--x", "0");
    el.style.setProperty("--y", "0");
    const width = el.clientWidth;
    const height = el.clientHeight;
    el.style.setProperty(
      "--x",
      window.innerWidth - tooltipData.mouseX - width < 0
        ? tooltipData.mouseX - width + "px"
        : tooltipData.mouseX + "px",
    );
    console.log(
      parseFloat(tooltipData.mouseY) - height,
      tooltipData.mouseY,
      height,
    );
    el.style.setProperty(
      "--y",
      window.innerHeight - tooltipData.mouseY - height < 0
        ? tooltipData.mouseY - height + "px"
        : tooltipData.mouseY + "px",
    );
  };
</script>

<div class="container">
  <h1>Map</h1>

  <MapImage onmousemove={onMouseOver} bind:this={svg} />
  {#if tooltipData !== null}
    <div
      class="hoverer"
      style={`--x: ${tooltipData.mouseX}px; --y: ${tooltipData.mouseY}px`}
      {@attach attachment(tooltipData)}
    >
      <strong>{tooltipData.name} - {currentCountryWords.length}</strong><br />
      <div class="languages">
        {#each Object.entries(tooltipCountryWords) as [language, words]}
          <div>
            <strong>{language}</strong><br />
            <div class="words">
              {#each inChunksOf(words, 8) as chunk}
                <div>
                  {#each chunk as word}
                    {word.word} - {word.summary}<br />
                  {/each}
                </div>
              {/each}
            </div>
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
    left: var(--x);
    top: var(--y);
    padding: 5px 10px;
    pointer-events: none;
  }
  .languages {
    display: flex;
    gap: 1rem;
  }
  .words {
    display: flex;
    gap: 1rem;
  }
</style>
