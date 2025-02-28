<script setup>
import { ref, onMounted } from "vue";
import { invoke } from "@tauri-apps/api/core";
import Swal from 'sweetalert2'; // 引入 sweetalert2

const greetMsg = ref("");
const name = ref("");
const grid = ref([]);
const selectedCell = ref(null);
const gridSize = ref(4); // 新增难度选择框的响应式变量
const clickedCells = ref([]); // 新增记录点击顺序的数组
const timer = ref("00:00:00.000"); // 新增秒表显示时间的响应式变量
const intervalId = ref(null); // 新增存储 setInterval 返回值的响应式变量

async function greet() {
  // Learn more about Tauri commands at https://tauri.app/develop/calling-rust/
  greetMsg.value = await invoke("greet", { name: name.value });
}

function generateGrid() {
  const size = gridSize.value; // 使用选择框的值作为网格大小
  const newGrid = [];
  for (let i = 0; i < size; i++) {
    const row = [];
    for (let j = 0; j < size; j++) {
      row.push(i * size + j + 1);
    }
    newGrid.push(...row);
  }
  // 打乱数组
  for (let i = newGrid.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [newGrid[i], newGrid[j]] = [newGrid[j], newGrid[i]];
  }
  grid.value = [];
  for (let i = 0; i < size; i++) {
    grid.value.push(newGrid.slice(i * size, (i + 1) * size));
  }
  clickedCells.value = []; // 重置点击顺序数组
  resetTimer(); // 重置秒表
}

function handleCellClick(cell) {
  if (clickedCells.value.length === 0) {
    startTimer(); // 开始秒表计时
  }
  if (clickedCells.value.length === 0 || cell === clickedCells.value[clickedCells.value.length - 1] + 1) {
    clickedCells.value.push(cell);
    selectedCell.value = cell;
    console.log(cell, clickedCells);
    if (clickedCells.value.length === gridSize.value * gridSize.value) {
      stopTimer(); // 停止秒表计时
      // alert("welldone"); // 删除:alert("welldone");
      Swal.fire({ // 新增: 使用 sweetalert2 显示消息
        title: '恭喜',
        text: '你完成了任务！',
        icon: 'success',
        confirmButtonText: '确定'
      });
    }
  } else {
    resetGrid();
  }
}

function resetGrid() {
  clickedCells.value = [];
  selectedCell.value = null;
  resetTimer(); // 重置秒表
}

function startTimer() {
  let milliseconds = 0;
  intervalId.value = setInterval(() => {
    milliseconds+=100;
    timer.value = formatTime(milliseconds);
  }, 100); // 修改: 每毫秒更新一次
}

function stopTimer() {
  clearInterval(intervalId.value);
}

function resetTimer() {
  clearInterval(intervalId.value);
  timer.value = "00:00:000";
}

function formatTime(milliseconds) {
  const pad = (num, size) => String(num).padStart(size, '0');
  const totalSeconds = Math.floor(milliseconds / 1000);
  const minutes = Math.floor(totalSeconds / 60);
  const seconds = totalSeconds % 60;
  const ms = milliseconds % 1000;
  return `${pad(minutes, 2)}:${pad(seconds, 2)}:${pad(ms, 3)}`; // 修改: 添加毫秒显示，确保毫秒是三位数
}

// 在页面挂载时生成一个 4x4 的格子
onMounted(() => {
  generateGrid();
});
</script>

<template>
  <main class="container">
    <h1>舒尔特方格注意力</h1>

    <div class="controls"> <!-- 新增一个容器来包裹难度选择框和生成按钮 -->
      <label for="difficulty">难度:</label>
      <select id="difficulty" v-model="gridSize" class="custom-select">
        <option v-for="size in [3,4, 5, 6, 7, 8]" :key="size" :value="size">{{ size }}</option>
      </select>

      <button @click="generateGrid">生成</button>
    </div>

    <div class="timer"> <!-- 新增秒表示例 -->
      {{ timer }}
    </div>

    <div class="grid" v-if="grid.length">
      <div class="row" v-for="(row, rowIndex) in grid" :key="rowIndex">
        <div class="cell"
             v-for="(cell, cellIndex) in row"
             :key="cellIndex"
             @click="handleCellClick(cell)"
             :class="{ 'selected':  clickedCells.includes(cell) }">
          {{ cell }}
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>
.logo.vite:hover {
  filter: drop-shadow(0 0 2em #747bff);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #249b73);
}

.grid {
  margin-top: 20px;
}

.row {
  display: flex;
  justify-content: center;
}

.cell {
  border: 1px solid #ccc;
  padding: 10px;
  width: 50px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 5px;
  cursor: pointer; /* 添加鼠标指针样式 */
}

.cell.selected {
  background-color: yellow; /* 修改选中时变为黄色 */
}

.custom-select {
  padding: 0.6em 1.2em;
  font-size: 1em;
  font-weight: 500;
  font-family: inherit;
  color: #0f0f0f;
  background-color: #ffffff;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 2px 2px rgba(0, 0, 0, 0.2);
  cursor: pointer;
  transition: border-color 0.25s;
}

.custom-select:hover {
  border-color: #396cd8;
}

.custom-select:focus {
  border-color: #396cd8;
  background-color: #e8e8e8;
}

.controls { /* 新增样式以确保难度选择框和生成按钮在同一行并居中 */
  display: flex;
  align-items: center;
  justify-content: center; /* 新增 */
  margin-bottom: 20px;
}

.controls label {
  margin-right: 10px;
}

.controls .custom-select {
  margin-right: 10px;
}

.timer { /* 新增秒表样式 */
  margin-bottom: 20px;
  font-size: 1.2em;
  font-weight: bold;
}
</style>
<style>
:root {
  font-family: Inter, Avenir, Helvetica, Arial, sans-serif;
  font-size: 16px;
  line-height: 24px;
  font-weight: 400;

  color: #0f0f0f;
  background-color: #f6f6f6;

  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  -webkit-text-size-adjust: 100%;
}

.container {
  margin: 0;
  padding-top: 10vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  text-align: center;
}

.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: 0.75s;
}

.logo.tauri:hover {
  filter: drop-shadow(0 0 2em #24c8db);
}

.row {
  display: flex;
  justify-content: center;
}

a {
  font-weight: 500;
  color: #646cff;
  text-decoration: inherit;
}

a:hover {
  color: #535bf2;
}

h1 {
  text-align: center;
}

input,
button {
  border-radius: 8px;
  border: 1px solid transparent;
  padding: 0.6em 1.2em;
  font-size: 1em;
  font-weight: 500;
  font-family: inherit;
  color: #0f0f0f;
  background-color: #ffffff;
  transition: border-color 0.25s;
  box-shadow: 0 2px 2px rgba(0, 0, 0, 0.2);
}

button {
  cursor: pointer;
}

button:hover {
  border-color: #396cd8;
}
button:active {
  border-color: #396cd8;
  background-color: #e8e8e8;
}

input,
button {
  outline: none;
}

#greet-input {
  margin-right: 5px;
}

@media (prefers-color-scheme: dark) {
  :root {
    color: #f6f6f6;
    background-color: #2f2f2f;
  }

  a:hover {
    color: #24c8db;
  }

  input,
  button {
    color: #ffffff;
    background-color: #0f0f0f98;
  }
  button:active {
    background-color: #0f0f0f69;
  }
}
</style>





