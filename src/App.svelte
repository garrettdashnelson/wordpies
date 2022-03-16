<script>

  const magicWord = 'able';
  const fullAlphabet = 'abcdefghijklmnopqrstuvwxyz'.split("");


  const radiansInCircle = 360 * Math.PI/180;

  const shuffle = a => {
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    const temp = a[i];
    a[i] = a[j];
    a[j] = temp;
    }

    return a;
  }


  let shuffledAlphabet = shuffle(fullAlphabet);
  let wheelOffset = 0;

  let playingLetterIndex = 1;

  let incorrectGuesses = 0;
  let currentGuess = '';


  const pivotWheel = (direction) => {
    wheelOffset = direction === "f" ? wheelOffset + 1 : wheelOffset - 1;
  }

  const executeCut = () => {
    let offset = wheelOffset % shuffledAlphabet.length;
    let lowerHalf = [];
    
    for(let i = 0; i < shuffledAlphabet.length/2; i++) {
      lowerHalf.push( shuffledAlphabet[((offset+i)+shuffledAlphabet.length)%shuffledAlphabet.length] )
    }
    
    if(lowerHalf.includes( magicWord.substring(playingLetterIndex,playingLetterIndex+1) )) {
      shuffledAlphabet = shuffle(lowerHalf);
    } else {
      shuffledAlphabet = shuffle(shuffledAlphabet.filter(d=>!lowerHalf.includes(d)));
    }

  }

  const executeGuess = () => {
    if(currentGuess.toLowerCase() === magicWord.substring(playingLetterIndex,playingLetterIndex+1)) {
      window.alert("That's it!")
      playingLetterIndex++;
      shuffledAlphabet = shuffle(fullAlphabet);
      wheelOffset = 0;
      currentGuess = "";
    } else {
      window.alert("That wasn't it");
      currentGuess = "";
      incorrectGuesses++;
    }
  }



</script>

<main>

  <div id="wheel-outer">
    <div id="wheel-inner">
      <svg id="wheel-svg">
        <g transform="translate(175,175)">
          {#each shuffledAlphabet as letter, i}
            <text text-anchor="middle" x="{150* Math.cos((radiansInCircle/shuffledAlphabet.length)*(i+(-wheelOffset)))}" y="{150* Math.sin((radiansInCircle/shuffledAlphabet.length)*(i+(-wheelOffset)))}" transform="translate(0,5)">{letter.toUpperCase()}</text>
          {/each}
            <line x1="-200" y1="0" x2="200" y2="0" stroke="black" transform="rotate(-7)" />
        </g>
  
      </svg>
    </div>
  </div>

  <button on:click={()=>{pivotWheel('f')}}>Turn Right</button>
  <button on:click={()=>{pivotWheel('l')}}>Turn Left </button>
  <button on:click={executeCut}>Cut</button>

  <div id="guess-section-outer">
    <div id="guess-section-inner">
      <input bind:value={currentGuess} type="text" name="" id="guess-input" maxlength="1" size="1">
      <button on:click={executeGuess}>⬅️ Guess this letter</button>
    </div>
  </div>

  <div id="magic-word-boxes-outer">
    <div id="magic-word-boxes-inner">
      {#each magicWord.split("") as magicWordLetter, i}
        <div class="magic-word-box" class:revealed="{i < playingLetterIndex}">{magicWordLetter}</div>
      {/each}
    </div>
  </div>

  <div id="incorrect-guesses-outer">
    {#each Array(incorrectGuesses) as _ }
      ☠️
    {/each}
  </div>

</main>

<style>
  :root {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen,
      Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  }

  #wheel-svg {
    width: 350px;
    height: 400px;
    background-color: pink;
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
    font-size: 3.0rem;
  }

  #guess-section-outer {
    margin: 1em 0;
    background-color: blue;
  }

</style>
