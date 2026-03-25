<script>
	import wordsData from "./words.json";
	const cleanWord = (w) => w.replace(/\d+$/, "");
	let words = wordsData.map((w, index) => ({
		...w,
		displayWord: cleanWord(w.word),
		index,
	})).length;
	import itemsData from "./items.json";
	const cleanWord = (w) => w.replace(/\d+$/, "");
	let items = itemsData.map((item, index) => ({ ...item, index })).length;
	function increment() {
	  vc += 0.025 * (180 - vc); // manually update until conjugation rules go in a json file retrieved by both this and the Conjugations page
	  w += 0.025 * (words - w);
	  gr += 0.025 * (items - gr);
	  c += 0.02 * (21 - c);
	  v += 0.015 * (6 - v);
	  l += 0.005 * (1 - l);
	  document.querySelector("#vc").textContent = Math.round(vc);
	  document.querySelector("#w").textContent = Math.round(w);
	  document.querySelector("#gr").textContent = Math.round(gr);
	  document.querySelector("#c").textContent = Math.round(c);
	  document.querySelector("#v").textContent = Math.round(v);
	  document.querySelector("#l").textContent = Math.round(l);
	  document.querySelector("#plural1").textContent = " verb conjugations";
	  document.querySelector("#plural2").textContent = " words";
	  document.querySelector("#plural3").textContent = " grammar rules";
	  document.querySelector("#plural4").textContent = " consonants";
	  document.querySelector("#plural5").textContent = " vowels";
	  document.querySelector("#plural6").textContent = " languages";
	  if (Math.round(vc) == 1) {
		document.querySelector("#plural1").textContent = " verb conjugation";
	  }
	  if (Math.round(w) == 1) {
		document.querySelector("#plural2").textContent = " word";
	  }
	  if (Math.round(gr) == 1) {
		document.querySelector("#plural3").textContent = " grammar rule";
	  }
	  if (Math.round(c) == 1) {
		document.querySelector("#plural4").textContent = " consonant";
	  }
	  if (Math.round(v) == 1) {
		document.querySelector("#plural5").textContent = " vowel";
	  }
	  if (Math.round(l) == 1) {
		document.querySelector("#plural6").textContent = " language";
	  }
	}
	
	let vc = 0;
	let w = vc;
	let gr = w;
	let c = gr;
	let v = c;
	let l = v;
	window.onload = () => {
	  setTimeout(() => {setInterval(increment, 30)}, 500);
	}
</script>

<main class="container">
	<h1>Batelu</h1>
	<p>welcome to batelu. more information about this language can be found on the About page.</p><hr>
	<h2>At-a-Glance Statistics</h2>
	<span class="fancy" id="vc">0</span><span id="plural1"> verb conjugations</span><br>
	<span class="fancy" id="w">0</span><span id="plural2"> words</span><br>
	<span class="fancy" id="gr">0</span><span id="plural3"> grammar rules</span><br>
	<span class="fancy" id="c">0</span><span id="plural4"> consonants</span><br>
	<span class="fancy" id="v">0</span><span id="plural5"> vowels</span><br>
	<span class="fancy" id="l">0</span><span id="plural6"> languages</span><br>

	<p>check out the conjugation/pronouns spreadsheet: <a href="https://docs.google.com/spreadsheets/d/1-hEdtf7EK9FfPEpfMT1NzB222KUJgFtyccIxuLS_V90/edit?gid=0#gid=0">google sheets</a></p>
	<p>join the <a href="https://discord.gg/PqRqeztXQF">discord</a> :3</p>
</main>

<style>
@import url('https://fonts.cdnfonts.com/css/selawik');

body {
  background-color: #0b0b0c;
  color: #f5f5f5;
}

span {
  font-family: Arial;
  font-size: 2em;
  margin-top: 50px;
  padding-top: 50px;
}

.fancy {
  font-size: 4em;
  margin-left: 50px;
  font-family: "Segoe UI", "Selawik", sans-serif;
  font-weight: 900;
  background-image: linear-gradient(90deg, violet, yellow);
  color: transparent;
  background-clip: text;
}
</style>
