<script setup lang="ts">
import { ref, onMounted, computed } from "vue";

// consts
const length = 15;
const minesCount = 15;

// refs
const grid = ref(Array(length).fill(Array(length).fill(null)));
const hiddenGrid = ref(Array(length).fill(Array(length).fill(false)));
const canPlay = ref(false);

// fonctions
function generateRandomPosition() {
  return {
    x: Math.floor(Math.random() * length),
    y: Math.floor(Math.random() * length),
  };
}
function displayCell(indexRow: number, indexCol: number) {
  if (grid.value[indexRow][indexCol] === null) {
    return " ";
  }
  if (grid.value[indexRow][indexCol] === "flag") {
    return "🚩";
  }
  if (grid.value[indexRow][indexCol] === false) {
    return " ";
  }
  if (grid.value[indexRow][indexCol] === "bomb") {
    return "💀";
  }
  return grid.value[indexRow][indexCol];
}
function clickCell(indexRow: number, indexCol: number) {
  console.log(indexRow, indexCol);
  if (hiddenGrid.value[indexRow][indexCol] === true) {
    alert("You loose !");
    revealAllBombs();
    return;
  }
  if (grid.value[indexRow][indexCol] === null) {
    setCell(indexRow, indexCol, false);
    setBombAroundCount(indexRow, indexCol);
    if (hasWon.value === true) {
      alert("You won !");
      return;
    }
  }
}
function setCell(
  indexRow: number,
  indexCol: number,
  value: boolean | string | null
) {
  grid.value[indexRow] = [...grid.value[indexRow]];
  grid.value[indexRow][indexCol] = value;
}
function setBombAroundCount(indexRow: number, indexCol: number) {
  let count = 0;
  if (
    indexRow - 1 >= 0 &&
    indexCol - 1 >= 0 &&
    hiddenGrid.value[indexRow - 1][indexCol - 1] === true
  ) {
    count++;
  }
  if (indexRow - 1 >= 0 && hiddenGrid.value[indexRow - 1][indexCol] === true) {
    count++;
  }
  if (
    indexCol + 1 < length &&
    hiddenGrid.value[indexRow][indexCol + 1] === true
  ) {
    count++;
  }
  if (
    indexRow + 1 < length &&
    hiddenGrid.value[indexRow + 1][indexCol] === true
  ) {
    count++;
  }
  if (
    indexRow + 1 < length &&
    indexCol + 1 < length &&
    hiddenGrid.value[indexRow + 1][indexCol + 1] === true
  ) {
    count++;
  }
  if (indexCol - 1 >= 0 && hiddenGrid.value[indexRow][indexCol - 1] === true) {
    count++;
  }
  if (
    indexRow + 1 < length &&
    indexCol - 1 >= 0 &&
    hiddenGrid.value[indexRow + 1][indexCol - 1] === true
  ) {
    count++;
  }
  if (
    indexRow - 1 >= 0 &&
    indexCol + 1 < length &&
    hiddenGrid.value[indexRow - 1][indexCol + 1] === true
  ) {
    count++;
  }

  if (count === 0) {
    clickCell(indexRow, indexCol);
    revealAroundCell(indexRow, indexCol);
    return;
  }
  setCell(indexRow, indexCol, String(count));
}
function revealAroundCell(indexRow: number, indexCol: number) {
  if (
    indexRow - 1 >= 0 &&
    indexCol - 1 >= 0 &&
    grid.value[indexRow - 1][indexCol - 1] === null
  ) {
    setBombAroundCount(indexRow - 1, indexCol - 1);
  }
  if (indexRow - 1 >= 0 && grid.value[indexRow - 1][indexCol] === null) {
    setBombAroundCount(indexRow - 1, indexCol);
  }
  if (indexCol + 1 < length && grid.value[indexRow][indexCol + 1] === null) {
    setBombAroundCount(indexRow, indexCol + 1);
  }
  if (indexRow + 1 < length && grid.value[indexRow + 1][indexCol] === null) {
    setBombAroundCount(indexRow + 1, indexCol);
  }
  if (
    indexRow + 1 < length &&
    indexCol + 1 < length &&
    grid.value[indexRow + 1][indexCol + 1] === null
  ) {
    setBombAroundCount(indexRow + 1, indexCol + 1);
  }
  if (indexCol - 1 >= 0 && grid.value[indexRow][indexCol - 1] === null) {
    setBombAroundCount(indexRow, indexCol - 1);
  }
  if (
    indexRow + 1 < length &&
    indexCol - 1 >= 0 &&
    grid.value[indexRow + 1][indexCol - 1] === null
  ) {
    setBombAroundCount(indexRow + 1, indexCol - 1);
  }
  if (
    indexRow - 1 >= 0 &&
    indexCol + 1 < length &&
    grid.value[indexRow - 1][indexCol + 1] === null
  ) {
    setBombAroundCount(indexRow - 1, indexCol + 1);
  }
}
function getFontColorByCount(indexRow: number, indexCol: number) {
  const outputColor = {
    ".": "#eee",
    F: "#1000f0",
    " ": "#666",
    "1": "#009964",
    "2": "#00a61f",
    "3": "#92ba00",
    "4": "#b09300",
    "5": "#b36200",
    "6": "#a83e00",
    "7": "#a83300",
    "8": "#ab2000",
  };
  return outputColor[grid.value[indexRow][indexCol]] ?? "";
}
function getBgColorByValue(indexRow: number, indexCol: number) {
  if (
    grid.value[indexRow][indexCol] === null ||
    grid.value[indexRow][indexCol] === "flag"
  ) {
    return "#666";
  } else {
    return "#ccc";
  }
}
function revealAllBombs() {
  hiddenGrid.value.forEach((row, indexRow) => {
    row.forEach((col, indexCol) => {
      if (col === true) {
        setCell(indexRow, indexCol, "bomb");
      }
    });
  });
}
function flagCell(event: any, indexRow: number, indexCol: number) {
  if (grid.value[indexRow][indexCol] === null) {
    setCell(indexRow, indexCol, "flag");
    event.preventDefault();
    return;
  }
  if (grid.value[indexRow][indexCol] === "flag") {
    setCell(indexRow, indexCol, null);
    event.preventDefault();
    return;
  }
}

// computeds
const hasWon = computed(() => {
  return (
    grid.value.filter((row) => {
      return row.filter((col) => {
        col === false;
      });
    }).length ===
    length * length - minesCount
  );
});

// lifecycle
onMounted(() => {
  for (let i = 0; i < minesCount; i++) {
    while (true) {
      let randomPosition = generateRandomPosition();
      if (hiddenGrid.value[randomPosition.x][randomPosition.y] === false) {
        hiddenGrid.value[randomPosition.x] = [
          ...hiddenGrid.value[randomPosition.x],
        ];
        hiddenGrid.value[randomPosition.x][randomPosition.y] = true;
        break;
      }
    }
  }
  canPlay.value = true;
});
</script>

<template>
  <h1>Démineur</h1>
  <button @click="revealAllBombs()">ici</button>
  <div>{{ hasWon }}</div>
  <div v-if="canPlay">
    <div v-for="(row, indexRow) in grid" :key="indexRow" class="demineur-row">
      <div
        v-for="(col, indexCol) in row"
        :key="indexCol"
        class="demineur-col"
        @click="clickCell(indexRow, indexCol)"
        @contextmenu="(event) => flagCell(event, indexRow, indexCol)"
        :style="{
          color: getFontColorByCount(indexRow, indexCol),
          'background-color': getBgColorByValue(indexRow, indexCol),
        }"
      >
        {{ displayCell(indexRow, indexCol) }}
      </div>
    </div>
  </div>

  <div v-else>Chargement en cours</div>
</template>

<style scoped>
.demineur-row {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
}

.demineur-col {
  text-align: center;
  font-weight: bolder;
  font-size: 20px;
  background-color: rgb(221, 221, 221);
  border: 1px solid black;
  height: 20px;
  width: 20px;
}

.demineur-col:hover {
  background-color: #888 !important;
}
</style>
