<script setup>
import {ref, onMounted, onUnmounted} from 'vue';

// Configurações do jogo (5 letras, 6 tentativas)
const wordLength = 5;
const maxGuesses = 6;

// Banco de palavras fictício (idealmente seriam todas maiúsculas e sem acento para simplificar)
const wordBank = ['TERMO', 'VERDE', 'PENSAR', 'ALUNO', 'PORTA', 'NOITE', 'FUEGO', 'MUNDO', 'LOGICA', 'VUEJS'];
// Filtrando apenas as que têm 5 letras para garantir
const validWords = wordBank.filter(w => w.length === wordLength);

// Estados Reativos
const targetWord = ref('');
const guesses = ref([]); // Array de strings (ex: ['TERMO', 'LETRA'])
const currentGuess = ref('');
const gameOver = ref(false);
const gameWon = ref(false);

// Layout do Teclado
const keyboardRows = [
  ['Q', 'W', 'E', 'R', 'T', 'Y', 'U', 'I', 'O', 'P'],
  ['A', 'S', 'D', 'F', 'G', 'H', 'J', 'K', 'L', 'Ç'],
  ['ENTER', 'Z', 'X', 'C', 'V', 'B', 'N', 'M', 'BACK']
];

// Inicializa ou reinicia o jogo
const resetGame = () => {
  const randomIndex = Math.floor(Math.random() * validWords.length);
  targetWord.value = validWords[randomIndex].toUpperCase();
  guesses.value = [];
  currentGuess.value = '';
  gameOver.value = false;
  gameWon.value = false;
};

// Retorna a letra a ser exibida na grade
const getLetterValue = (rowIndex, colIndex) => {
  if (rowIndex < guesses.value.length) {
    return guesses.value[rowIndex][colIndex];
  }
  if (rowIndex === guesses.value.length) {
    return currentGuess.value[colIndex] || '';
  }
  return '';
};

// Define as cores das letras na grade (Lógica principal)
const getLetterClass = (rowIndex, colIndex) => {
  if (rowIndex >= guesses.value.length) return 'bg-light';

  const guessWord = guesses.value[rowIndex];
  const letter = guessWord[colIndex];
  const target = targetWord.value;

  // 1. Correta (Verde)
  if (target[colIndex] === letter) {
    return 'text-bg-success';
  }
  // 2. Existe na palavra, mas posição errada (Amarelo)
  if (target.includes(letter)) {
    return 'text-bg-warning';
  }
  // 3. Não existe (Cinza)
  return 'text-bg-secondary';
};

// Estiliza o teclado virtual com base no progresso
const getKeyClass = (key) => {
  if (key === 'ENTER' || key === 'BACK') return 'functional';

  let status = '';
  for (const guess of guesses.value) {
    for (let i = 0; i < guess.length; i++) {
      if (guess[i] === key) {
        if (targetWord.value[i] === key) {
          return 'text-bg-success'; // Prioridade máxima: verde
        } else if (targetWord.value.includes(key)) {
          status = 'text-bg-warning'; // Amarelo
        } else if (!status) {
          status = 'text-bg-secondary opacity-25'; // Cinza
        }
      }
    }
  }
  return status || 'btn-outline-secondary';
};

// Gerencia a digitação (Teclado físico e virtual)
const handleKeyPress = (key) => {
  if (gameOver.value) return;

  const upperKey = key.toUpperCase();

  if (upperKey === 'ENTER') {
    submitGuess();
  } else if (upperKey === 'BACK' || upperKey === 'BACKSPACE') {
    currentGuess.value = currentGuess.value.slice(0, -1);
  } else if (/^[A-ZÇ]$/.test(upperKey) && currentGuess.value.length < wordLength) {
    currentGuess.value += upperKey;
  }
};

// Lógica de envio da palavra
const submitGuess = () => {
  if (currentGuess.value.length !== wordLength) {
    alert('A palavra deve ter 5 letras!');
    return;
  }

  guesses.value.push(currentGuess.value);

  if (currentGuess.value === targetWord.value) {
    gameWon.value = true;
    gameOver.value = true;
  } else if (guesses.value.length >= maxGuesses) {
    gameOver.value = true;
  }

  currentGuess.value = '';
};

// Escuta o teclado físico do computador
const handleKeyDown = (e) => {
  if (e.key === 'Backspace') handleKeyPress('BACK');
  else if (e.key === 'Enter') handleKeyPress('ENTER');
  else handleKeyPress(e.key);
};

onMounted(() => {
  resetGame();
  window.addEventListener('keydown', handleKeyDown);
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown);
});
</script
>

<template>
  <div class="container mt-5">
    <h1 class="text-center mb-5">Descubra a palavra</h1>

    <!-- Grade do Jogo -->
    <div class="row justify-content-center mb-4">
      <div class="col-auto">
        <div
            v-for="(row, rowIndex) in maxGuesses"
            :key="rowIndex"
            class="d-flex mb-1 gap-1"
        >
          <div
              v-for="(col, colIndex) in wordLength"
              :key="colIndex"
              class="letter-box rounded-3 border border-secondary d-flex align-items-center justify-content-center fw-bold fs-3"
              :class="getLetterClass(rowIndex, colIndex)"
          >
            {{ getLetterValue(rowIndex, colIndex) }}
          </div>
        </div>
      </div>
    </div>
    <!-- Teclado Virtual Simplificado -->
    <div class="row justify-content-center mb-4">
      <div class="col-md-8 col-lg-6">
        <div v-for="row in keyboardRows" :key="row" class="d-flex justify-content-center mb-1 gap-1">
          <button
              v-for="key in row"
              :key="key"
              @click="handleKeyPress(key)"
              class="btn btn-sm btn-outline-secondary key-btn flex-grow-1"
              :class="getKeyClass(key)"
          >
            {{ key }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.letter-box {
  width: 60px;
  height: 60px;
}

.key-btn {
  font-size: 1.1rem;
}

.key-btn.functional {
  font-size: 0.8rem;
}
</style>
