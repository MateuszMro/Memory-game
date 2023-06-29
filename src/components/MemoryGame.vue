<script defer setup lang="js">
import $ from 'jquery'

// Tworzenie planszy gry Memory
function createBoard(size) {
  const symbols = ['A','A','B', 'B','C', 'C','D', 'D','E', 'E','F', 'F','G', 'G','H', 'H']
  const cards = []

  // Generowanie kart
  for (let i = 0; i < size; i++) {
    const symbol = symbols[i % symbols.length]
    cards.push({ id: i, symbol: symbol, isFlipped: false, isMatched: false })
  }

  // Mieszanie kart
  cards.sort(() => Math.random() - 0.5)

  return cards
}

// Inicjalizacja planszy
let boardSize = 16; // Domyślna liczba kart
let board = createBoard(boardSize)

// Inicjalizacja graczy
const players = ['Gracz 1', 'Gracz 2']
let currentPlayerIndex = 0
let playerScores = [0, 0]

// Zliczanie odkrytych i sparowanych kart
let flippedCards = []
let matchedCards = 0
let isCardClickable = true

// Funkcja obsługująca kliknięcie na kartę
function handleCardClick(cardId) {

  if(!isCardClickable){
    return
  }
  const card = board[cardId]

  // Sprawdzanie, czy karta już została odkryta lub sparowana
  if (card.isFlipped || card.isMatched) {
    return
  }

  // Odkrywanie karty
  card.isFlipped = true
  flippedCards.push(cardId)

  // Sprawdzanie, czy odkryto dwa sparowane symbole
  if (flippedCards.length === 2) {

      const card1 = board[flippedCards[0]]
      const card2 = board[flippedCards[1]]

    // Sprawdzanie, czy symbole kart są identyczne
    if (card1.symbol === card2.symbol) {

      card1.isMatched = true
      card2.isMatched = true
      matchedCards += 2


      // Przyznawanie punktu aktualnemu graczowi
      playerScores[currentPlayerIndex]++


      // Sprawdzanie, czy wszystkie karty zostały sparowane
      if (matchedCards === boardSize) {
        setTimeout(()=>{
          alert('Gra zakończona! Wszystkie karty zostały sparowane.');
        },500)
      }
    } else {
      // Czekanie 1 sekundy przed odwróceniem niepasujących kart
      isCardClickable=false
      setTimeout(() => {
        card1.isFlipped = false
        card2.isFlipped = false
        currentPlayerIndex = (currentPlayerIndex + 1) % players.length
        renderBoard()
        isCardClickable= true
      }, 1000)
    }

    // Czyszczenie listy odkrytych kart
    flippedCards = []
  }
  else {
    currentPlayerIndex = (currentPlayerIndex + 1) % players.length;
  }

  renderBoard()
}

// Funkcja renderująca planszę gry
function renderBoard() {
  const boardElement = $('#board')

  // Czyszczenie planszy
  boardElement.empty()

  // Generowanie kart
  board.forEach((card, index) => {
    const cardElement = $('<button>').addClass('card')
    cardElement.text(card.isFlipped || card.isMatched ? card.symbol : 'ODKRYJ')
    cardElement.on('click', () => handleCardClick(index))
    boardElement.append(cardElement)
  })

  // Aktualizowanie wyników graczy
  const scoreElement = $('#scoreboard')
  scoreElement.empty()
  players.forEach((player, index) => {
    const playerScore = $('<p>').text(`${player}: ${playerScores[index]} punktów`)
    scoreElement.append(playerScore)
  })
}


// Inicjalizacja gry
function initializeGame() {
  const sizeInput = $('#board-size')
  const startButton = $('#start-game')
  const restartButton = $('#restart-button')

  startButton.on('click', function() {
    const size = parseInt(sizeInput.val())

    if (size % 4 === 0 && size >= 8) {
      boardSize = size
      board = createBoard(boardSize)
      flippedCards = []
      matchedCards = 0
      currentPlayerIndex = 0
      playerScores = [0, 0]
      renderBoard()
    } else {
      alert('Wielkość planszy musi być parzystą liczbą większą lub równa 8 (z krokiem co 4)')
    }
  })

  restartButton.on('click',function (){
    restartGame()
  })
}
// Funkcja restartująca grę
function restartGame() {
  board = createBoard(boardSize)
  flippedCards = []
  matchedCards = 0
  currentPlayerIndex = 0
  playerScores = [0, 0]
  renderBoard()
}

// Inicjalizacja gry przy załadowaniu dokumentu
$(document).ready(function() {
  initializeGame()
});



</script>

<template>
  <div id="scoreboard"></div>
  <div>
    <label for="board-size">Wielkość planszy (parzysta liczba): </label>
    <input type="number" id="board-size" min="8" step="4">
    <button id="start-game">Rozpocznij grę</button>
    <button id="restart-button">Restartuj grę</button>
  </div>
  <div id="board"></div>

</template>

<style scoped>

#board {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-gap: 5px;
}

#scoreboard{
  margin-bottom: 10px;

}
#restart-button{
  margin-bottom: 15px;
}
#start-game{
  margin-left: 5px;
}

.card {
  width: 100px;
  height: 100px;
  background-color: #999;
  margin: 15px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  cursor: pointer;
  padding-left: 30px;
  padding-right: 30px;
}



</style>