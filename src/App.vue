<script setup>
import { ref, onMounted, computed, watch } from "vue";
import "./main.css";

const todos = ref([]);
const filteredtodos = ref([]);
const todo = ref("");
// const pendingTodos=localStorage.getItem('pendingTodos')||todos.length;

const todo_content = ref("");

//Sorting the array from latest added
const todos_asc = computed(() =>
  filteredtodos.value.sort((a, b) => {
    return b.createdAt - a.createdAt;
  })
);

//Creating a todo
const createTodo = () => {
  if (todo_content.value.trim() === "" || todo_content.value === null) {
    return;
  }
  console.log("added todo: " + todo_content.value);

  todos.value.push({
    content: todo_content.value,
    completed: false,
    createdAt: new Date().getTime(),
  });
  todo_content.value = "";
};

const deleteTodo = (createdAt) => {
  todos.value = todos.value.filter((todo) => todo.createdAt !== createdAt);
  filteredtodos.value = todos.value;
};

watch(
  todos,
  (newTodo) => {
    localStorage.setItem("todos", JSON.stringify(newTodo));
  },
  { deep: true }
);

onMounted(() => {
  todos.value = JSON.parse(localStorage.getItem("todos")) || [];
});

const checkCompleted = () => {
  const pending = todos.value.filter((todo) => {
    return todo.completed === false;
  });
  localStorage.setItem("pendingTodos", pending.length);
};

//Sorting on completed status
const filterStatus = (status) => {
  filteredtodos.value = todos.value.filter((todo) => {
    return status === ""
      ? true
      : todo.completed === status || todo.completed === "";
  });
};

const clearCompleted = () => {
  todos.value = todos.value.filter((todo) => {
    return todo.completed === false;
  });
  filteredtodos.value = todos.value;
};

const onDragStart = (event, todoitem) => {
  console.log(todoitem);
  event.dataTransfer.dropEffect = "move";
  event.dataTransfer.effectAllowed = "move";
  event.dataTransfer.setData("createdAt", todoitem.createdAt);
};

const onDrop = (event, itempos) => {
  const createdAt = event.dataTransfer.getData("createdAt");
  const item = item.value.find((item) => item.createdAt == createdAt);
  itempos = item;
};

//switch to light mode
const switchToLightMode = () => {
  document.getElementById('mainContainer').style.cssText="background-image: url('./src/assets/bg-desktop-light.jpg');";
  document.getElementById('todomain').style.cssText="background-color:hsl(236, 33%, 92%);";
  document.getElementById('displayTodos').style.cssText="background-color:hsl(0, 0%, 98%);";
  document.getElementById('filterMobile').style.cssText="border-top: 10px solid hsl(236, 33%, 92%);";
  document.getElementById('createtodoInput').style.cssText="background-color:hsl(0, 0%, 98%);";
  document.getElementById('sun').style.cssText="display:none"
  document.getElementById('moon').style.cssText="display:block"
};
//switch to dark mode
const switchToDarkMode = () => {
  document.getElementById('mainContainer').style.cssText="background-image: url('./src/assets/bg-desktop-dark.jpg');";
  document.getElementById('todomain').style.cssText="background-color:hsl(235, 21%, 11%);";
  document.getElementById('displayTodos').style.cssText="background-color:hsl(235, 24%, 19%);";
  document.getElementById('filterMobile').style.cssText="border-top: 10px solid hsl(235, 21%, 11%);";
  document.getElementById('createtodoInput').style.cssText="background-color: hsl(235, 24%, 19%)";
  document.getElementById('sun').style.cssText="display:block"
  document.getElementById('moon').style.cssText="display:none"
};
//Dan 2022 nov
</script>

<template>
  <div id="todomain"></div>
  <div id="mainContainer" >
    <div id="heading">
      <h3>T O D O</h3>
      <svg xmlns="http://www.w3.org/2000/svg" id="sun" width="26" height="26" @click="switchToLightMode">
        <path
          fill="#FFF"
          fill-rule="evenodd"
          d="M13 21a1 1 0 011 1v3a1 1 0 11-2 0v-3a1 1 0 011-1zm-5.657-2.343a1 1 0 010 1.414l-2.121 2.121a1 1 0 01-1.414-1.414l2.12-2.121a1 1 0 011.415 0zm12.728 0l2.121 2.121a1 1 0 01-1.414 1.414l-2.121-2.12a1 1 0 011.414-1.415zM13 8a5 5 0 110 10 5 5 0 010-10zm12 4a1 1 0 110 2h-3a1 1 0 110-2h3zM4 12a1 1 0 110 2H1a1 1 0 110-2h3zm18.192-8.192a1 1 0 010 1.414l-2.12 2.121a1 1 0 01-1.415-1.414l2.121-2.121a1 1 0 011.414 0zm-16.97 0l2.121 2.12A1 1 0 015.93 7.344L3.808 5.222a1 1 0 011.414-1.414zM13 0a1 1 0 011 1v3a1 1 0 11-2 0V1a1 1 0 011-1z"
        />
      </svg>
      <svg xmlns="http://www.w3.org/2000/svg" id="moon" width="26" height="26" @click="switchToDarkMode"><path fill="#FFF" fill-rule="evenodd" d="M13 0c.81 0 1.603.074 2.373.216C10.593 1.199 7 5.43 7 10.5 7 16.299 11.701 21 17.5 21c2.996 0 5.7-1.255 7.613-3.268C23.22 22.572 18.51 26 13 26 5.82 26 0 20.18 0 13S5.82 0 13 0z"/></svg>

    </div>
    <div class="createTodo">
      <form @submit.prevent="createTodo">
        <input
          type="text"
          v-model="todo_content"
          placeholder="Create a new todo..."
          id="createtodoInput"
        />
        {{ todo }}
      </form>
    </div>
    <div
      id="displayTodos"
      @drop="onDrop($event, itempos)"
      @dragenter.prevent
      @dragover.prevent
    >
      <div
        v-for="todo in todos_asc"
        :class="`todo`"
        draggable="true"
        @dragenter.prevent
        @dragover.prevent
        @dragstart="onDragStart($event, todo.createdAt)"
      >
        <label>
          <input
            type="checkbox"
            v-model="todo.completed"
            id="checkbox"
            @click="checkCompleted"
          /> </label
        >
          <div class="todoinfo">
            <div id="updateTodo">
              {{todo.content}}
              <!-- <input id="updateTodo" type="text" v-model="todo.content" class="textbox"/> -->
            </div>
 
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="18"
            height="18"
            @click="deleteTodo(todo.createdAt)"
          >
            <path
              fill="#494C6B"
              fill-rule="evenodd"
              d="M16.97 0l.708.707L9.546 8.84l8.132 8.132-.707.707-8.132-8.132-8.132 8.132L0 16.97l8.132-8.132L0 .707.707 0 8.84 8.132 16.971 0z"
            />
          </svg>
          </div>

      </div>
      <div id="todosAbout">
        <div class="uncompleted">{{ todos.length }} items left</div>
        <div id="filter">
          <a @click="filterStatus('')">All</a>
          <a @click="filterStatus(false)">Active</a>
          <a @click="filterStatus(true)">Completed</a>
        </div>
        <div class="clearCompleted">
          <a @click="clearCompleted">Clear Completed</a>
        </div>
      </div>
      <div id="filterMobile">
        <a @click="filterStatus('')">All</a>
        <a @click="filterStatus(false)">Active</a>
        <a @click="filterStatus(true)">Completed</a>
      </div>
    </div>
  </div>
  <div></div>
</template>