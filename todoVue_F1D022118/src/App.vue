<template>
  <div :class="['main-bg', darkMode ? 'dark-bg' : '']">
    <div class="center-container">
      <h1 class="todo-title">TODO LIST</h1>
      <div class="todo-card">
        <form @submit.prevent="addTask" class="todo-form">
          <input v-model="newTask" placeholder="Add new task..." class="todo-input" />
          <button type="submit" class="primary-btn">Add Task</button>
          <select v-model="filter" class="filter-select">
            <option value="all">All</option>
            <option value="done">Done</option>
            <option value="todo">Todo</option>
          </select>
        </form>
        <ul class="task-list">
          <li v-for="(task, index) in filteredTasks" :key="index" class="task-item">
            <input type="checkbox" v-model="task.completed" @change="saveTasks" />
            <div class="task-info">
              <span :class="{ completed: task.completed }" @dblclick="startEdit(index)" title="Double click to edit">
                <template v-if="editIndex === index">
                  <input v-model="editTemp" @keyup.enter="saveEdit(index)" @keyup.esc="cancelEdit" class="edit-input"/>
                  <button @click="saveEdit(index)" class="icon-btn">✔</button>
                  <button @click="cancelEdit" class="icon-btn">✖</button>
                </template>
                <template v-else>
                  {{ task.content }}
                </template>
              </span>
              <div class="task-meta">{{ formatDate(task.timestamp) }}</div>
            </div>
            <button @click="deleteTask(index)" class="icon-btn">🗑️</button>
            <button v-if="editIndex !== index" @click="startEdit(index)" class="icon-btn">✏️</button>
          </li>
        </ul>
        <div v-if="filteredTasks.length === 0" class="empty-text">
          No tasks found!
        </div>
      </div>
      <button class="toggle-theme" @click="darkMode = !darkMode">
        Theme: {{ darkMode ? 'Dark' : 'Light' }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, computed } from 'vue'

// State
const tasks = ref([])
const newTask = ref('')
const filter = ref('all')
const editIndex = ref(-1)
const editTemp = ref('')
const darkMode = ref(false)

// LocalStorage integration
onMounted(() => {
  const saved = localStorage.getItem('tasks')
  if (saved) tasks.value = JSON.parse(saved)
  const mode = localStorage.getItem('darkMode')
  if (mode) darkMode.value = JSON.parse(mode)
})

watch([tasks, darkMode], () => {
  localStorage.setItem('tasks', JSON.stringify(tasks.value))
  localStorage.setItem('darkMode', JSON.stringify(darkMode.value))
}, { deep: true })

function addTask() {
  const val = newTask.value.trim()
  if (val) {
    tasks.value.unshift({
      content: val,
      completed: false,
      timestamp: new Date().getTime(),
    })
    newTask.value = ''
  }
}

function deleteTask(index) {
  tasks.value.splice(index, 1)
  cancelEdit()
}
function startEdit(index) {
  editIndex.value = index
  editTemp.value = tasks.value[index].content
}
function cancelEdit() {
  editIndex.value = -1
  editTemp.value = ''
}
function saveEdit(index) {
  if (editTemp.value.trim()) {
    tasks.value[index].content = editTemp.value.trim()
    editIndex.value = -1
    editTemp.value = ''
  }
}
function saveTasks() {
  localStorage.setItem('tasks', JSON.stringify(tasks.value))
}

function formatDate(ts) {
  const d = new Date(ts)
  return d.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }) + ' ' +
    d.toLocaleDateString([], { day: '2-digit', month: '2-digit', year: 'numeric' })
}

const filteredTasks = computed(() => {
  if (filter.value === 'done') return tasks.value.filter(t => t.completed)
  if (filter.value === 'todo') return tasks.value.filter(t => !t.completed)
  return tasks.value
})
</script>

<style>
:root {
  --border: #eaebf3;
  --bg-card: rgba(244,247,255,0.92);
  --font-main: #48495b;
  --font-meta: #7e8197;
  --bg-form: #f7f9fb;
  --primary-gradient: linear-gradient(90deg,#ea64d3,#6574f7 90%);
  --btn-primary: #656fe7;
  --btn-done: #5cc886;
  --btn-danger: #ff5a96;
  --shadow1: 0 4px 32px #cdcbe9ab;
}
body, html, #app {
  height: 100vh;
  min-height: 100vh;
  margin: 0;
  padding: 0;
  font-family: 'Segoe UI', Arial, sans-serif;
  background: transparent;
}

.main-bg {
  position: fixed;
  inset: 0;
  width: 100vw;
  min-height: 100vh;
  height: 100vh;
  background: radial-gradient(ellipse 70% 70% at 50% 45%, #ffe0f5 1%, #e0e7ff 80%);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1;
  overflow: auto;
}
.dark-bg {
  background: radial-gradient(ellipse 80% 80% at 50% 50%, #222237 0%, #29294f 35%, #1c1d2a 100%);
}

.center-container {
  z-index: 2;
  width: 100%;
  min-height: 70vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.todo-title {
  font-size: 2.1em;
  font-weight: 700;
  color: var(--font-main);
  margin-top: 12px;
  margin-bottom: 24px;
  letter-spacing: 0.04em;
  text-align: center;
}
.dark-bg .todo-title { color: #e7e9fd; }

.todo-card {
  min-width: 310px;
  max-width: 410px;
  width: 100%;
  background: var(--bg-card);
  border-radius: 21px;
  box-shadow: var(--shadow1);
  padding: 24px 18px 30px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
}
.dark-bg .todo-card {
  background: rgba(37, 37, 57, 0.96);
  box-shadow: 0 8px 38px #111b2240;
}

.todo-form {
  width: 100%;
  display: flex;
  gap: 8px;
  align-items: center;
  flex-wrap: wrap;
}
.todo-input {
  flex: 2 1 180px;
  padding: 11px 16px;
  border-radius: 11px;
  background: var(--bg-form);
  border: 1.5px solid var(--border);
  font-size: 1em;
  color: var(--font-main);
}
.todo-input:focus { border: 1.8px solid #ea64d3; }
.dark-bg .todo-input { background: #232441; color: #eee; border: 1.4px solid #36385a; }
.primary-btn {
  flex: 1 1 100px;
  padding: 10px 18px;
  border-radius: 11px;
  border: none;
  background: var(--primary-gradient);
  color: #fff;
  font-weight: 600;
  font-size: 1em;
}
.filter-select {
  flex: 1 1 80px;
  padding: 10px;
  border-radius: 7px;
  border: 1px solid var(--border);
  background: var(--bg-form);
  color: var(--font-main);
  font-size: 1em;
}
.dark-bg .filter-select { background: #232441; color: #eee; border: 1.4px solid #36385a; }

.task-list {
  width: 100%;
  padding: 0;
  list-style: none;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.task-item {
  display: flex;
  align-items: center;
  gap: 12px;
  background: #fdf7ff;
  box-shadow: 0 1px 7px #ded1f033;
  border-radius: 13px;
  padding: 10px 17px 10px 12px;
  margin-bottom: 0;
  font-size: 1.03em;
  transition: box-shadow 0.13s, background 0.12s;
  width: 100%;
  box-sizing: border-box;
}

.task-item span {
  color: var(--font-main); /* pakai warna teks utama */
  font-weight: 500;
  letter-spacing: 0.01em;
  transition: color 0.18s;
}

.dark-bg .task-item span {
  color: #e8eafd;
}

.completed {
  text-decoration: line-through;
  color: #a5a5b3 !important;  /* pastikan hanya task selesai yang pudar */
  font-style: italic;
}

.dark-bg .completed {
  color: #636b98 !important;
}

.task-item:hover { box-shadow: 0 3px 12px #ded1f066; background: #f7f4fe; }
.dark-bg .task-item { background: #251f30; color: #eee; }

input[type="checkbox"] {
  height: 20px;
  width: 20px;
  margin-right: 3px;
  accent-color: #ff4c8b;
}

.task-info {
  flex: 7;
  min-width: 0;
  display: flex;
  flex-direction: column;
}
.completed {
  text-decoration: line-through;
  color: #a5a5b3;
  font-style: italic;
}
.dark-bg .completed { color: #636b98; }

.task-meta {
  font-size: 0.93em;
  color: var(--font-meta);
}
.dark-bg .task-meta {
  color: #b7bad9;
}
.edit-input {
  flex: 1;
  padding: 8px 11px;
  border-radius: 9px;
  background: var(--bg-form);
  border: 1.7px solid #d7d8de;
  margin-right: 5px;
  font-size: 1em;
  color: var(--font-main);
}
.dark-bg .edit-input { background: #29294f; color: #eee; border: 1.8px solid #595c6e; }

.icon-btn {
  background: transparent;
  border: none;
  font-size: 1.22em;
  margin-left: 6px;
  margin-right: 2px;
  cursor: pointer;
  color: var(--btn-primary);
}
.icon-btn:active { color: #fe36b5; }

.empty-text {
  margin: 22px auto 7px;
  color: #c8a2db;
  font-size: 1.09em;
  text-align: center;
}
.dark-bg .empty-text { color: #9073c4; }

.toggle-theme {
  margin-top: 24px;
  padding: 9px 29px;
  border-radius: 12px;
  border: none;
  background: var(--primary-gradient);
  color: #fff;
  font-weight: 600;
  font-size: 1em;
  box-shadow: 0 2px 10px #e1d3f155;
}

@media (max-width: 560px) {
  .todo-card { min-width: 96vw; max-width: 99vw; padding: 10px 2vw 22px; }
  .center-container { margin-top: 0; }
  .task-form, .task-list { width: 100%; }
  .task-item { flex-direction: column; align-items: flex-start; padding: 14px 9px 11px 10px;}
}
</style>