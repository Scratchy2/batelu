<script>
  export let verbWord;
  export let ending;
</script>

<span>
  {#if verbWord}
    <span class="conjugate-secondary">{verbWord.stem}</span>{ending.replace(
      "V",
      verbWord.vowel,
    )}
  {:else}
    -{ending}
  {/if}
</span>

<style>
  .conjugate-secondary {
    opacity: 0.5;
  }
</style>
