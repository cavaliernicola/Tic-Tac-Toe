<script setup>
import { range } from "lodash";

const emit = defineEmits(['update-row', 'start-game']);
const props = defineProps({
  rows: {
    type: Number,
    required: true
  }
})

</script>

<template>
  <div class="wrapper">
    <h1>How large should the board be?</h1>
    <select :value="props.rows" @input="emit('update-row', $event.target.value)">
      <option v-for="row in range(3, 6)" :key="row" :value="row">{{ `${row} x ${row}`}}</option>
    </select>
    <button @click="emit('start-game')">Confirm</button>
  </div>
</template>

<style lang="scss" scoped>
.wrapper {
  border-radius: 5px;
  width: 700px;
  max-width: 90%;
  border: 1px solid black;
  padding: 50px;
  position: absolute;
  top: 30%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  flex-direction: column;

  h1 {
    text-align: center;
    font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
  }

  select {
    text-align: center;
    margin-bottom: 5px;
    border: 1px solid black;
  }

  button {
    border: 1px solid black;
    border-radius: 2px;
  }
}
</style>