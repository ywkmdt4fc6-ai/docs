const STORAGE_KEY = 'todo-list-items-v1';

const todoForm = document.getElementById('todo-form');
const todoInput = document.getElementById('todo-input');
const todoList = document.getElementById('todo-list');
const todoTemplate = document.getElementById('todo-item-template');
const clearCompletedBtn = document.getElementById('clear-completed');
const itemsLeftEl = document.getElementById('items-left');
const filterButtons = Array.from(document.querySelectorAll('.filter-btn'));

let tasks = loadTasks();
let currentFilter = 'all';

function loadTasks() {
  try {
    const saved = localStorage.getItem(STORAGE_KEY);
    if (!saved) return [];

    const parsed = JSON.parse(saved);
    return Array.isArray(parsed) ? parsed : [];
  } catch (error) {
    console.error('Failed to load tasks from localStorage:', error);
    return [];
  }
}

function saveTasks() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(tasks));
}

function updateItemsLeft() {
  const remaining = tasks.filter((task) => !task.completed).length;
  const label = `${remaining} item${remaining === 1 ? '' : 's'} left`;
  itemsLeftEl.textContent = label;
}

function getFilteredTasks() {
  if (currentFilter === 'active') {
    return tasks.filter((task) => !task.completed);
  }

  if (currentFilter === 'completed') {
    return tasks.filter((task) => task.completed);
  }

  return tasks;
}

function renderTasks() {
  const filteredTasks = getFilteredTasks();
  todoList.innerHTML = '';

  filteredTasks.forEach((task) => {
    const fragment = todoTemplate.content.cloneNode(true);
    const item = fragment.querySelector('.todo-item');
    const checkbox = fragment.querySelector('.todo-checkbox');
    const text = fragment.querySelector('.todo-text');
    const deleteBtn = fragment.querySelector('.delete-btn');

    item.dataset.id = String(task.id);
    if (task.completed) {
      item.classList.add('completed');
    }

    checkbox.checked = task.completed;
    text.textContent = task.text;

    checkbox.addEventListener('change', () => {
      toggleTask(task.id);
    });

    deleteBtn.addEventListener('click', () => {
      deleteTask(task.id);
    });

    todoList.appendChild(fragment);
  });

  updateItemsLeft();
}

function addTask(text) {
  const trimmed = text.trim();
  if (!trimmed) {
    todoInput.focus();
    return;
  }

  tasks.unshift({
    id: crypto.randomUUID(),
    text: trimmed,
    completed: false,
  });

  saveTasks();
  renderTasks();
  todoInput.value = '';
  todoInput.focus();
}

function toggleTask(id) {
  tasks = tasks.map((task) =>
    task.id === id ? { ...task, completed: !task.completed } : task,
  );

  saveTasks();
  renderTasks();
}

function deleteTask(id) {
  tasks = tasks.filter((task) => task.id !== id);
  saveTasks();
  renderTasks();
}

function clearCompleted() {
  tasks = tasks.filter((task) => !task.completed);
  saveTasks();
  renderTasks();
}

function setFilter(filter) {
  currentFilter = filter;

  filterButtons.forEach((button) => {
    const isActive = button.dataset.filter === filter;
    button.classList.toggle('active', isActive);
  });

  renderTasks();
}

todoForm.addEventListener('submit', (event) => {
  event.preventDefault();
  addTask(todoInput.value);
});

clearCompletedBtn.addEventListener('click', clearCompleted);

filterButtons.forEach((button) => {
  button.addEventListener('click', () => {
    setFilter(button.dataset.filter);
  });
});

renderTasks();
