<script setup>
import { computed } from 'vue';
import { range, isEqual } from "lodash";

const props = defineProps({
  rows: {
    type: Number,
    required: true
  },
  table: {
    type: Array,
    required: true
  },
  winRow: {
    type: Array,
    required: true
  }, 
  currentPlayer: {
    type: String,
    required: true
  }
})

const emit = defineEmits(['update-cell', 'reset-game', 'change-settings']);

const handleCellClick = (cell, row) => {
  const index = getIndex(cell, row);
  emit('update-cell', winnerCombo.value, index)
}

const isDraw = computed(() => !props.winRow.length && props.table.every(item => item !== '') )

const winnerCombo = computed(() => {
  const result = []
  for (let i = 1; i < props.rows + 1; i++) {
    const row = []
    const column = []

    const first_diagonal = []
    const second_diagonal = []

    for (let c = 0; c < props.rows; c++) {
      // for the column we need to check the the current element (i) and increase it by the number of rows 
      // to get the element below the current one. (then we decrase by 1 because table array where this value will be needed starts from 0)
      column.push(i + (props.rows * c) - 1) 

      // the elements in the row are consecutively increased, since (i) starts from 1 our iteration would start from the second row, we move it to the correct row by substracting a row with <- props.row>
      // the result of <(i * props.rows) - props.rows> is the same for each element of the row, because (i) doesn't change until the for loop finishes
      // therefore to get the correct element in the row we need to rely on a second variable (c).
      row.push(c + (i * props.rows) - props.rows) 

      if (i == 1){
        // the first diagonal works in the same way as column, with the difference that for each row we want our element one position ahead the previous element, 
        // we achieve by adding the (c) that increases for each iteration.
        first_diagonal.push(c + (c * props.rows)) 

        // the second diagonal works in a similar way as first diagonal, with the different that starts from the end (so we add to each row the props.rows to get the last element)
        // and then we want our element a position before the previous element, we achieve it by substracting the (c) (then we decrase by 1 because table array where this value will be needed starts from 0).
        second_diagonal.push((c * props.rows) + props.rows - c - 1)
      }
    } 
  
    if (first_diagonal.length == props.rows){
      result.push(first_diagonal)
      result.push(second_diagonal)
    } 

    result.push(row)
    result.push(column)
  }
  return result;
})

const lineClass = computed(() => {
  const min = Math.min(...props.winRow);
  const max = Math.max(...props.winRow);

  const column = [];
  // To check if the winRow is a column we need to increased each element by the lenght of the row (so we get the element right below the current one)
  for (let i = min; i <= max; i += props.rows) {
    column.push(i)
  }
  
  // To check if element are consecutive we just need to check if winRow is equal to the range between min and max
  const isConsecutive = isEqual(range(min, max+1), props.winRow);
  const isColumn = isEqual(column, props.winRow);

  let result;
  if (isConsecutive) result = "straight";
  else if (isColumn) result = "column";
  // If it's not a row or a column we need to check if the min is equal to 0 or the max is equal to table.value.lenght - 1 (array starts from 0 so we need to substract 1)
  // but in our case we just check if it's equal to 0 since all other cases are already checked.
  else if (min == 0) result = "diagonal-1";
  else result = "diagonal-2";

  return result;
})

// All those methods are dynamic, therefore they can't be cached with a computed function.
const getIndex = (cell, row) => {
  return cell + (row * props.rows) - props.rows - 1;
}

const currentValue = (cell, row) => {
  const index = getIndex(cell, row);
  return props.table[index];
}

const headerText = () => {
  let text;
  if (props.winRow.length) text = `${props.table[props.winRow[0]]} won the game!`
  else if (isDraw.value) text = "The game ended in a draw!"
  else text = `Current Player: ${props.currentPlayer}`
  return text;
}

</script>

<template>
  <h1 class="game-header">{{headerText()}}</h1>
  <div class="game-grid">
    <div v-for="row in rows" :key="row" class="row">
      <div v-for="cell in rows" :key="cell" class="cell" @click="handleCellClick(cell, row)">
        <p class="content" :class="['value-' + currentValue(cell, row)]">{{ currentValue(cell, row) }}</p>
        <div v-if="winRow.includes(getIndex(cell, row))" class="line" :class="[lineClass]"></div>
      </div>
    </div>
  </div>
  <div class="action-buttons" :style="{width: `${(rows * 130)}px`}">
    <button @click="emit('change-settings')">Change Settings</button>
    <button @click="emit('reset-game')">Restart</button>
  </div>
</template>

<style lang="scss" scoped>
.game-header {
  text-align: center;
  font-family: 'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;
}
.game-grid {
  margin-top: 20px;
  position: relative;

 .row {
  display: flex;
  justify-content: center;

  .cell {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 130px;
    border: 1px solid black;
    margin: 1px;

    .content {
      margin: 0 auto;
      position: absolute;
      font-size: 40px;
      font-weight: bold;
      font-family: 'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;

      &.value-X {
        color: red;
      }

      &.value-O {
        color: green;
      }
    }

    .line {
      position: absolute;
      height: 3px;
      width: 110%;
      background-color: darkblue;

      &.diagonal-1 {
        width: 150%;
        transform: rotate(42deg);
      }

      &.diagonal-2 {
        width: 150%;
        transform: rotate(138deg);
      }

      &.column {
        transform: rotate(90deg);
      }
    }
  }
  
  .cell::after {
    content: "";
    display: block;
    padding-bottom: 90%;
  }
 }
}
.action-buttons {
  max-width: 100%;
  margin: 0 auto;
  margin-top: 20px;
  display: flex;
  justify-content: space-between;

  button {
    width: 100px;
    border-radius: 20px;
    padding: 10px;
    color: blue;
    cursor: pointer;
    border: 1px solid;
    background-color: white;
  }
}

</style>