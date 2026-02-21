<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Twisted Wordle: Ranked Mode</title>
<style>
body {
  font-family: 'Arial', sans-serif;
  margin: 0;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(135deg, #f0f0f0, #b0c0f0);
}
#frontPage, #gamePage { display: none; flex-direction: column; align-items: center; }
#frontPage.active, #gamePage.active { display: flex; }
h1 { margin-bottom: 20px; text-shadow: 1px 1px 3px #555; text-align:center; }
button {
  padding: 15px 35px;
  font-size: 18px;
  border: none;
  border-radius: 10px;
  background: #6a5acd;
  color: white;
  cursor: pointer;
  transition: 0.3s;
  margin-top:10px;
}
button:hover { background: #483d8b; }
.grid { display: grid; grid-gap: 10px; margin-bottom: 10px; }
.cell {
  width: 60px;
  height: 60px;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #fff;
  border: 2px solid #ccc;
  font-size: 24px;
  font-weight: bold;
  transition: all 0.5s;
}
.correct { background: #6aaa64; color: white; animation: bounce 0.5s; }
.wrong-place { background: #c9b458; color: white; transform: rotate(15deg); }
.wrong { background: #787c7e; color: white; opacity: 0.6; }
@keyframes bounce { 0% {transform:translateY(0);} 50%{transform:translateY(-15px);} 100%{transform:translateY(0);} }
#clue, #score, #level, #topScore { margin-bottom: 10px; font-weight: bold; text-shadow:1px 1px 2px #555; }
#guessInput { font-size: 18px; padding: 5px; text-transform: uppercase; width: 180px; text-align: center; border-radius: 5px; border: 2px solid #6a5acd; outline: none; }
#guessInput:focus { border-color: #483d8b; }
</style>
</head>
<body>

<div id="frontPage" class="active">
  <h1>Twisted Wordle Adventure</h1>
  <p style="text-align:center; max-width:300px;">Guess the word in 6 tries. Words get harder each level. Clues are provided!</p>
  <button onclick="startGame()">Start Game</button>
  <p>Top Score: <span id="frontTopScore">0</span></p>
</div>

<div id="gamePage">
  <h1>Twisted Wordle</h1>
  <div id="score">Score: 0</div>
  <div id="level">Level: 1</div>
  <div id="topScore">Top Score: 0</div>
  <div id="clue">Clue: ???</div>
  <div class="grid" id="grid"></div>
  <input type="text" id="guessInput" placeholder="Type your guess" autofocus>
</div>

<script>
// Word bank by length (short version, you can expand)
let wordBank = [
  // 5-letter words
  {word:"CRANE", clue:"A bird and a machine"}, {word:"PLANT", clue:"Grows in the ground"}, {word:"GHOST", clue:"Spooky presence"}, 
  {word:"BRAVE", clue:"Not afraid"}, {word:"LIGHT", clue:"Opposite of dark"}, {word:"HOUSE", clue:"Place to live"}, 
  {word:"APPLE", clue:"A fruit"}, {word:"TRAIN", clue:"Moves on rails"}, {word:"RIVER", clue:"Flows"}, {word:"STORM", clue:"Bad weather"},
  {word:"SWEET", clue:"Tasty"}, {word:"STONE", clue:"Hard object"}, {word:"DREAM", clue:"Hopes"}, {word:"GLASS", clue:"Transparent"}, 
  {word:"PLANE", clue:"Flying vehicle"}, {word:"FROST", clue:"Frozen dew"}, {word:"CROWN", clue:"Royal headwear"}, {word:"SWORD", clue:"Weapon"}, 
  {word:"WATCH", clue:"Tells time"}, {word:"BREAD", clue:"Food"}, {word:"FISHY", clue:"Suspicious or seafood"}, {word:"MOUSE", clue:"Small animal"}, 
  {word:"FLAME", clue:"Fire"}, {word:"BERRY", clue:"Small fruit"}, {word:"SCALE", clue:"Measure"}, {word:"CHAIN", clue:"Linked rings"}, 
  {word:"SHADE", clue:"Dark area"}, {word:"THORN", clue:"Sharp plant"}, {word:"WATER", clue:"H2O"}, {word:"EARTH", clue:"Ground or planet"}, 
  {word:"ALARM", clue:"Warning signal"}, {word:"FRAME", clue:"Picture holder"}, {word:"MIGHT", clue:"Power"}, {word:"NIGHT", clue:"Opposite of day"}, 
  {word:"PLUSH", clue:"Soft"}, {word:"QUICK", clue:"Fast"}, {word:"RANGE", clue:"Span"}, {word:"SPLIT", clue:"Divide"}, {word:"TREND", clue:"Fashion"},
  // 6-letter
  {word:"FOSSIL", clue:"Ancient remains"}, {word:"MARKER", clue:"Used for writing"}, {word:"BRIDGE", clue:"Connects places"}, 
  {word:"PLANET", clue:"Orbits a star"}, {word:"ROCKET", clue:"Launches into space"}, {word:"CASTLE", clue:"Fortified building"}, 
  {word:"TUNNEL", clue:"Underground path"}, {word:"MARKET", clue:"Place to buy"}, {word:"BOTTLE", clue:"Holds liquid"}, 
  {word:"GUITAR", clue:"Musical instrument"}, {word:"WINDOW", clue:"See outside"}, {word:"SUMMER", clue:"Hot season"}, 
  {word:"WINTER", clue:"Cold season"}, {word:"JUNGLE", clue:"Dense forest"}, {word:"FATHER", clue:"Male parent"}, 
  {word:"MOTHER", clue:"Female parent"}, {word:"BREEZE", clue:"Light wind"}, {word:"TICKET", clue:"Pass for travel"}, 
  {word:"FAMILY", clue:"Loved ones"}, {word:"SYSTEM", clue:"Organized structure"}, {word:"BRIGHT", clue:"Shiny"}, 
  {word:"REWARD", clue:"Prize"}, {word:"SUBMIT", clue:"Hand in"}, {word:"ORANGE", clue:"Fruit"}, {word:"POCKET", clue:"Small bag"}, 
  {word:"DAMAGE", clue:"Harm"}, {word:"TUNING", clue:"Adjust sound"}, {word:"CASTOR", clue:"Type of wheel"}, {word:"MARKED", clue:"Labeled"}, {word:"HUNTER", clue:"Chases prey"},
  // 7-letter (30)
  {word:"JOURNEY", clue:"A long trip"}, {word:"MYSTERY", clue:"Something unknown"}, {word:"TREASURE", clue:"Hidden riches"}, 
  {word:"BALANCE", clue:"Evenness"}, {word:"PROJECT", clue:"Planned work"}, {word:"VILLAGE", clue:"Small town"}, 
  {word:"CAPTURE", clue:"Take control"}, {word:"PICTURE", clue:"Visual image"}, {word:"SECRETS", clue:"Hidden info"}, 
  {word:"ANALYZE", clue:"Examine"}, {word:"WEATHER", clue:"Climate"}, {word:"STRETCH", clue:"Extend"}, {word:"FREEDOM", clue:"Liberty"}, 
  {word:"MACHINE", clue:"Mechanical device"}, {word:"PROCESS", clue:"Series of steps"}, {word:"PASSION", clue:"Strong feeling"}, 
  {word:"SUPPORT", clue:"Help"}, {word:"ADJUSTS", clue:"Changes"}, {word:"VILLAGE", clue:"Small town"}, {word:"DYNAMIC", clue:"Constantly changing"}, 
  {word:"BALLOON", clue:"Inflatable"}, {word:"CAPITAL", clue:"City"}, {word:"ENIGMAS", clue:"Puzzles"}, {word:"HORIZON", clue:"Sky meets land"}, 
  {word:"LIBRARY", clue:"Books"}, {word:"GUITARS", clue:"Musical instruments"}, {word:"LANGUAG", clue:"Means of communication"}, {word:"MUSICAL", clue:"Relating to music"}, {word:"TREKKER", clue:"Hiker"}, {word:"SURPRISE", clue:"Unexpected event"},
  // 8-10 letters (20)
  {word:"LABYRINTH", clue:"Confusing maze"}, {word:"TREACHERY", clue:"Betrayal"}, {word:"VIGILANCE", clue:"Careful watch"}, 
  {word:"WILDERNESS", clue:"Untouched nature"}, {word:"XENOPHOBIA", clue:"Fear of strangers"}, {word:"ARCHETYPE", clue:"Original model"}, 
  {word:"MOUNTAINS", clue:"High land"}, {word:"KNIGHTHOOD", clue:"Order of knights"}, {word:"EXCAVATION", clue:"Digging site"}, 
  {word:"TRANSLATOR", clue:"Language converter"}, {word:"HEADQUARTER", clue:"Main office"}, {word:"INSPIRATION", clue:"Motivation"}, 
  {word:"OBSERVATION", clue:"Watching carefully"}, {word:"CELEBRATION", clue:"Festivity"}, {word:"DEVELOPMENT", clue:"Growth"}, 
  {word:"ENLIGHTENED", clue:"Fully aware"}, {word:"CONSEQUENT", clue:"Resulting"}, {word:"INTERACTION", clue:"Communication"}, 
  {word:"REMARKABLE", clue:"Exceptional"}, {word:"CHALLENGING", clue:"Difficult"}
];

let availableWords = [...wordBank]; // track unused words
let currentWord = {};
let level = 1;
let score = 0;
let currentRow = 0;

const frontPage = document.getElementById("frontPage");
const gamePage = document.getElementById("gamePage");
const grid = document.getElementById("grid");
const clueText = document.getElementById("clue");
const scoreText = document.getElementById("score");
const levelText = document.getElementById("level");
const topScoreText = document.getElementById("topScore");
const frontTopScoreText = document.getElementById("frontTopScore");
const input = document.getElementById("guessInput");

function startGame(){
  frontPage.classList.remove("active");
  gamePage.classList.add("active");
  level = 1; score = 0; currentRow = 0;
  scoreText.textContent = "Score: 0";
  levelText.textContent = "Level: 1";
  topScoreText.textContent = localStorage.getItem("guel") || 0;
  availableWords = [...wordBank];
  loadLevel();
}

function loadLevel(){
  if(availableWords.length === 0){
    alert("You've completed all words! Congrats!");
    availableWords = [...wordBank];
  }
  const index = Math.floor(Math.random() * availableWords.length);
  currentWord = availableWords.splice(index, 1)[0]; // remove to avoid repeat
  const wordLen = currentWord.word.length;

  clueText.textContent = "Clue: " + currentWord.clue;
  grid.innerHTML = "";
  grid.style.gridTemplateColumns = `repeat(${wordLen}, 60px)`;
  currentRow = 0;

  for(let i=0;i<6*wordLen;i++){
    const cell = document.createElement("div");
    cell.classList.add("cell");
    grid.appendChild(cell);
  }

  input.maxLength = wordLen;
  input.value = "";
  input.focus();
}

function checkGuess(guess){
  guess = guess.toUpperCase();
  const wordLen = currentWord.word.length;
  if(guess.length !== wordLen) return alert(`Word must be ${wordLen} letters!`);

  for(let i=0;i<wordLen;i++){
    const cell = grid.children[currentRow*wordLen + i];
    cell.textContent = guess[i];
    if(guess[i] === currentWord.word[i]){
      cell.classList.add("correct");
    } else if(currentWord.word.includes(guess[i])){
      cell.classList.add("wrong-place");
    } else{
      cell.classList.add("wrong");
    }
  }

  currentRow++;

  if(guess === currentWord.word){
    score += 10 * level;
    scoreText.textContent = "Score: " + score;
    if(score > (localStorage.getItem("guel") || 0)){
      localStorage.setItem("guel", score);
      topScoreText.textContent = score;
      frontTopScoreText.textContent = score;
    }
    level++;
    levelText.textContent = "Level: " + level;
    setTimeout(loadLevel, 800);
  } else if(currentRow === 6){
    alert(`Game Over! The word was: ${currentWord.word}`);
    if(score > (localStorage.getItem("guel") || 0)){
      localStorage.setItem("guel", score);
    }
    level = 1;
    score = 0;
    scoreText.textContent = "Score: 0";
    levelText.textContent = "Level: 1";
    topScoreText.textContent = localStorage.getItem("guel") || 0;
    setTimeout(loadLevel, 800);
  }
}

// Stopper for extra letters
input.addEventListener("input", function(){
  if(this.value.length > this.maxLength) this.value = this.value.slice(0, this.maxLength);
});

input.addEventListener("keyup", function(e){
  if(e.key === "Enter") { checkGuess(input.value); input.value=""; }
});
</script>

</body>
</html>
