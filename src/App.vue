<script setup>
import { ref } from 'vue';
import GridItem from './components/GridItem.vue';
import SettingsItem from "./components/SettingsItem.vue";

const rows = ref(3);
const table = ref([]);
const currentPlayer = ref("X");
const winRow = ref([]);
const gameStarted = ref(false);

function updateCell(winnerCombo, index) {
  if (table.value[index] !== '') return;
  if (winRow.value.length) return;

  table.value[index] = currentPlayer.value;
  checkWinner(winnerCombo);
  currentPlayer.value = currentPlayer.value == "X" ? "O" : "X";
}

const checkWinner = (winnerCombo) => {
  for (let win of winnerCombo) {
    const result = win
                    .map(combo => table.value[combo])
                    .every(item => table.value[win[0]] == item && item !== '')
    if (result) return winRow.value = win;
  }
}

const resetDefaultValues = () => {
  winRow.value = [];
  currentPlayer.value = "X"
}

const resetGame = () => {
  table.value.fill('');
  resetDefaultValues();
}

const updateRowValue = (newValue) => {
  rows.value = parseInt(newValue);
}

const startGame = () => {
  table.value = Array(rows.value**2).fill('');
  resetDefaultValues();
  gameStarted.value = true;
}

</script>

<template>
  <div>
    <SettingsItem v-if="!gameStarted" :rows="rows" @update-row="updateRowValue" @start-game="startGame"/>
    <GridItem v-else :table="table" :rows="rows" :win-row="winRow" :current-player="currentPlayer" @update-cell="updateCell" @reset-game="resetGame" @change-settings="gameStarted = false"/>
  </div>
</template>

<style lang="scss" scoped>
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}
</style>
