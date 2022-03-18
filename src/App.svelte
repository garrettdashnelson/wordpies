<script>
  import { tweened } from "svelte/motion";
  import { cubicOut } from "svelte/easing";

  const possibleWords = [
    "adze",
    "pore",
    "elks",
    "bump",
    "chat",
    "node",
    "dink",
    "lump",
  ];
  const magicWord =
    possibleWords[Math.floor(Math.random() * possibleWords.length)];
  const fullAlphabet = "abcdefghijklmnopqrstuvwxyz".split("");

  const radiansInCircle = (360 * Math.PI) / 180;

  let state = {
    guessBegin: false,
  };

  let potentialGuess = "";
  let showCheck = "";

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
    easing: cubicOut,
  });

  let playingLetterIndex = 1;
  let numberOfCuts = 0;

  let incorrectGuesses = 0;

  $: pointsToEarn = Math.pow(2, 4 - numberOfCuts) * (4 - playingLetterIndex);
  $: nextCutPoints = Math.pow(2, 4 - (numberOfCuts+1)) * (4 - playingLetterIndex);

  let earnedPoints = 0;

  const pivotWheel = (direction) => {
    if ($wheelOffset % 1 == 0) {
      if (direction === "f") {
        wheelOffset.set($wheelOffset + 1);
      } else {
        wheelOffset.set($wheelOffset - 1);
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
      showCheck = "lower";
      setTimeout(function () {
        showCheck = "";
        shuffledAlphabet = lowerHalf;
        padAlphabet();
      }, 1500);
    } else {
      showCheck = "upper";
      setTimeout(function () {
        showCheck = "";
        shuffledAlphabet = shuffledAlphabet.filter((d) => !lowerHalf.includes(d));
        padAlphabet();
      }, 1500);
    }

    numberOfCuts++;
  };

  const padAlphabet = () => {
    if(shuffledAlphabet.length === 13) {
      shuffledAlphabet.splice(3,0,"🙈");
      shuffledAlphabet.splice(6,0,"🙊");
      shuffledAlphabet.splice(9,0,"🙉");
    }
  }

  const beginGuess = (letter) => {
    state.guessBegin = true;
    potentialGuess = letter;
  };

  const executeGuess = () => {
    state.guessBegin = false;
    if (
      potentialGuess.toLowerCase() ===
      magicWord.substring(playingLetterIndex, playingLetterIndex + 1)
    ) {
      window.alert("That's it!");
      playingLetterIndex++;
      shuffledAlphabet = shuffle(fullAlphabet);
      wheelOffset.set(0);
      potentialGuess = "";
      numberOfCuts = 0;
      earnedPoints = earnedPoints + pointsToEarn;
    } else {
      window.alert("That wasn't it");
      potentialGuess = "";
      incorrectGuesses++;
      if (incorrectGuesses > 3) {
        window.alert("Game over");
      }
    }
  };
</script>

<main>
  <div id="wheel-outer">
    <div id="wheel-inner">
      <svg id="wheel-svg">
        <g transform="translate(175,175)">
          {#if showCheck === "lower"}
            <text class="check" x="-20" y="80"> ✅ </text>
          {:else if showCheck === "upper"}
            <text class="check" x="-20" y="-40"> ✅ </text>
          {/if}

          {#each shuffledAlphabet as letter, i}
            <g
              on:click={() => beginGuess(letter)}
              transform="translate({150 *
                Math.cos(
                  (radiansInCircle / shuffledAlphabet.length) *
                    (i + -$wheelOffset) +
                    radiansInCircle / shuffledAlphabet.length / 2
                )}, {150 *
                Math.sin(
                  (radiansInCircle / shuffledAlphabet.length) *
                    (i + -$wheelOffset) +
                    radiansInCircle / shuffledAlphabet.length / 2
                )})"
            >
              <circle r="12" />
              <text
                class="letter-wheel-text"
                text-anchor="middle"
                transform="translate(0,5)">{letter.toUpperCase()}</text
              >
            </g>
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

  <div id="magic-word-boxes-outer">
    <div id="magic-word-boxes-inner">
      {#each magicWord.split("") as magicWordLetter, i}
        <div
          class="magic-word-box"
          class:revealed={i < playingLetterIndex}
          class:now-guessing={i === playingLetterIndex}
        >
          {#if i < playingLetterIndex}
            {magicWordLetter}
          {:else}
            ?
          {/if}
        </div>
      {/each}
    </div>
  </div>

  {#if state.guessBegin}
    <div id="guess-modal">
      Guess {potentialGuess.toUpperCase()}
      <button class="button" on:click={executeGuess}>Do it</button>
    </div>
  {/if}

  <div>
    <div>
      A correct guess earns {pointsToEarn} points
    </div>
    <div>
      If you cut, then your guess would earn {nextCutPoints} points
    </div>
    <div>
      You've earned a total of {earnedPoints} points
    </div>
    <div id="incorrect-guesses-outer">
      {#each [0, 1, 2, 3] as i}
        {#if i < incorrectGuesses}
          💀
        {:else}
          🙂
        {/if}
      {/each}
    </div>
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
    background: rgb(34, 193, 195);
    background: linear-gradient(
      38deg,
      rgba(34, 193, 195, 0.17) 0%,
      rgba(253, 187, 45, 0.17) 100%
    );
  }

  button,
  input,
  optgroup,
  select,
  textarea {
    font-family: inherit; /* 1 */
    font-size: 100%; /* 1 */
    line-height: 1.15; /* 1 */
    margin: 0; /* 2 */
  }

  /**
 * Show the overflow in IE.
 * 1. Show the overflow in Edge.
 */

  button,
  input {
    /* 1 */
    overflow: visible;
  }

  /**
 * Remove the inheritance of text transform in Edge, Firefox, and IE.
 * 1. Remove the inheritance of text transform in Firefox.
 */

  button,
  select {
    /* 1 */
    text-transform: none;
  }

  /**
 * Correct the inability to style clickable types in iOS and Safari.
 */

  button,
  [type="button"],
  [type="reset"],
  [type="submit"] {
    -webkit-appearance: button;
  }

  main {
    width: 400px;
    margin: auto;
    text-align: center;
  }

  #wheel-svg {
    width: 350px;
    height: 350px;
  }

  #wheel-svg circle,
  text {
    cursor: pointer;
  }

  .letter-wheel-text {
    font-weight: bold;
    fill: white;
  }

  #magic-word-boxes-outer {
    margin: 1rem;
  }

  .magic-word-box {
    background-color: black;
    display: inline-block;
    text-transform: uppercase;
    padding: 0.5rem;
    border: 3px solid black;
    margin: 0 1rem 0 0;
    width: 2rem;
    height: 2rem;
    font-size: 1.7rem;
    font-weight: bold;
  }

  .magic-word-box.revealed {
    background-color: white;
  }

  .magic-word-box.now-guessing {
    background-color: #444;
    color: white;
  }

  #guess-modal {
    margin: 1rem;
    background-color: #9dbbae;
    border-radius: 5px;
    color: white;
    font-weight: bold;
    padding: 1rem;
    font-size: 1.6rem;
  }

  .button {
    display: inline-block;
    padding: 0.35em 0.8em;
    border: 0.1em solid #ffffff;
    border-radius: 0.12em;
    box-sizing: border-box;
    text-decoration: none;
    font-weight: 300;
    color: #ffffff;
    background-color: #6a2e35;
    text-align: center;
    transition: all 0.2s;
  }
  .button:hover {
    color: #000000;
    background-color: #ffffff;
  }

  .check {
    font-size: 3rem;
  }
</style>
