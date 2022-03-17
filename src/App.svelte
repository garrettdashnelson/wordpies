<script>
  import { tweened } from "svelte/motion";
  import { cubicOut } from "svelte/easing";

  const possibleWords = ['adze','pore','elks','bump','chat','node','dink','lump'];
  const magicWord = possibleWords[Math.floor(Math.random() * possibleWords.length)];
  const fullAlphabet = "abcdefghijklmnopqrstuvwxyz".split("");

  const radiansInCircle = (360 * Math.PI) / 180;

  const shuffle = (a) => {
    for (let i = a.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      const temp = a[i];
      a[i] = a[j];
      a[j] = temp;
    }

    return a;
  };

  let shuffledAlphabet = shuffle(fullAlphabet);
  const wheelOffset = tweened(0, {
		duration: 500,
		easing: cubicOut
	});

  let playingLetterIndex = 1;
  let numberOfCuts = 0;

  let incorrectGuesses = 0;
  let currentGuess = "";

  $: pointsToEarn = Math.pow(2, 4 - numberOfCuts) * (4 - playingLetterIndex);

  let earnedPoints = 0;

  const pivotWheel = (direction) => {
    if($wheelOffset%1 == 0) {
    if(direction === "f") {
      wheelOffset.set($wheelOffset+1);
    } else {
      wheelOffset.set($wheelOffset-1);
    }
    }
  };

  const executeCut = () => {
    let offset = $wheelOffset % shuffledAlphabet.length;
    let lowerHalf = [];

    for (let i = 0; i < shuffledAlphabet.length / 2; i++) {
      lowerHalf.push(
        shuffledAlphabet[
          (offset + i + shuffledAlphabet.length) % shuffledAlphabet.length
        ]
      );
    }

    if (
      lowerHalf.includes(
        magicWord.substring(playingLetterIndex, playingLetterIndex + 1)
      )
    ) {
      shuffledAlphabet = shuffle(lowerHalf);
    } else {
      shuffledAlphabet = shuffle(
        shuffledAlphabet.filter((d) => !lowerHalf.includes(d))
      );
    }

    if (shuffledAlphabet.length == 13) {
      shuffledAlphabet.push("🙈", "🙈", "🙊");
      shuffledAlphabet = shuffle(shuffledAlphabet);
    }
    numberOfCuts++;
  };

  const executeGuess = () => {
    if (
      currentGuess.toLowerCase() ===
      magicWord.substring(playingLetterIndex, playingLetterIndex + 1)
    ) {
      window.alert("That's it!");
      playingLetterIndex++;
      shuffledAlphabet = shuffle(fullAlphabet);
      wheelOffset.set(0);
      currentGuess = "";
      numberOfCuts = 0;
      earnedPoints = earnedPoints + pointsToEarn;
    } else {
      window.alert("That wasn't it");
      currentGuess = "";
      incorrectGuesses++;
      if(incorrectGuesses > 3) {
        window.alert("Game over")
      }
    }
  };
</script>

<main>
  <div id="wheel-outer">
    <div id="wheel-inner">
      <svg id="wheel-svg">
        <g transform="translate(175,175)">
          {#each shuffledAlphabet as letter, i}
            <text
              text-anchor="middle"
              x={150 *
                Math.cos(
                  (radiansInCircle / shuffledAlphabet.length) *
                    (i + -$wheelOffset) + (radiansInCircle/shuffledAlphabet.length/2)
                )}
              y={150 *
                Math.sin(
                  (radiansInCircle / shuffledAlphabet.length) *
                    (i + -$wheelOffset) + (radiansInCircle/shuffledAlphabet.length/2)
                )}
              transform="translate(0,5)">{letter.toUpperCase()}</text
            >
          {/each}
          <line
            x1="-200"
            y1="0"
            x2="200"
            y2="0"
            stroke="#130D4C"
            stroke-width="4"
          />
        </g>
      </svg>
    </div>
  </div>

  <button
    on:click={() => {
      pivotWheel("f");
    }}>Turn Right</button
  >
  <button
    on:click={() => {
      pivotWheel("l");
    }}
    >Turn Left
  </button>
  <button on:click={executeCut} disabled={numberOfCuts > 3}>Cut</button>

  <div id="guess-section-outer">
    <div id="guess-section-inner">
      <input
        bind:value={currentGuess}
        type="text"
        name=""
        id="guess-input"
        maxlength="1"
        size="1"
      />
      <button on:click={executeGuess}>⬅️ Guess this letter</button>
    </div>
  </div>

  <div>
    A correct guess earns {pointsToEarn} points
  </div>

  <div id="magic-word-boxes-outer">
    <div id="magic-word-boxes-inner">
      {#each magicWord.split("") as magicWordLetter, i}
        <div class="magic-word-box" class:revealed={i < playingLetterIndex}>
          {magicWordLetter}
        </div>
      {/each}
    </div>
  </div>

  <div>
    You've earned a total of {earnedPoints} points
  </div>

  <div id="incorrect-guesses-outer">
    {#each [0,1,2,3] as i}
      {#if i < incorrectGuesses}
        💀
      {:else}
        🙂
      {/if}
    {/each}
  </div>
</main>

<style>
  :root {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
      Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
  }

  :global(html) {
    min-height: 100%;
  }

  :global(body) {
    min-height: 100%;
    background: rgb(34,193,195);
    background: linear-gradient(38deg, rgba(34,193,195,0.17) 0%, rgba(253,187,45,0.17) 100%);
  }

  main { width: 400px; margin: auto; text-align: center; }

  #wheel-svg {
    width: 350px;
    height: 350px;
  }

  .magic-word-box {
    background-color: black;
    display: inline-block;
    text-transform: uppercase;
    padding: 1rem;
    border: 3px solid black;
    margin: 0 1rem 0 0;
  }

  .magic-word-box.revealed {
    background-color: white;
  }

  #guess-input {
    font-size: 3rem;
  }

  #guess-section-outer {
    margin: 1em 0;
    border: 2px solid #130D4C;
    border-radius: 8px;
    padding: 1rem;
  }

  #guess-section-outer input {
    width: 3rem;
  }
</style>
