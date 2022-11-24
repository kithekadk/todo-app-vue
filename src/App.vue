<script setup>
import { ref, onMounted, computed, watch } from "vue";
import "./main.css";

const todos = ref([]);
const todo = ref("");
// const pendingTodos=localStorage.getItem('pendingTodos')||todos.length;

const todo_content = ref("");

const todos_asc = computed(() =>
  todos.value.sort((a, b) => {
    return b.createdAt - a.createdAt;
  })
);

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

const checkCompleted=()=>{
  const pending = todos.value.filter((todo)=>{
    return todo.completed === false;
  })
  localStorage.setItem('pendingTodos', pending.length)
}

const onDragStart=(event, todoitem)=>{
  console.log(todoitem)
  event.dataTransfer.dropEffect ="move"
  event.dataTransfer.effectAllowed = "move"
  event.dataTransfer.setData('createdAt', todoitem.createdAt)
}

const onDrop = (event, itempos)=>{
  const createdAt= event.dataTransfer.getData('createdAt');
  const item = item.value.find((item)=>item.createdAt == createdAt)
  itempos= item
}
</script>

<template>
  <div id="todomain"></div>
  <div id="mainContainer">
    <div id="heading">
      <h3>T O D O</h3>
      <svg xmlns="http://www.w3.org/2000/svg" width="26" height="26"><path fill="#FFF" fill-rule="evenodd" d="M13 21a1 1 0 011 1v3a1 1 0 11-2 0v-3a1 1 0 011-1zm-5.657-2.343a1 1 0 010 1.414l-2.121 2.121a1 1 0 01-1.414-1.414l2.12-2.121a1 1 0 011.415 0zm12.728 0l2.121 2.121a1 1 0 01-1.414 1.414l-2.121-2.12a1 1 0 011.414-1.415zM13 8a5 5 0 110 10 5 5 0 010-10zm12 4a1 1 0 110 2h-3a1 1 0 110-2h3zM4 12a1 1 0 110 2H1a1 1 0 110-2h3zm18.192-8.192a1 1 0 010 1.414l-2.12 2.121a1 1 0 01-1.415-1.414l2.121-2.121a1 1 0 011.414 0zm-16.97 0l2.121 2.12A1 1 0 015.93 7.344L3.808 5.222a1 1 0 011.414-1.414zM13 0a1 1 0 011 1v3a1 1 0 11-2 0V1a1 1 0 011-1z"/></svg>

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
    <div id="displayTodos" @drop="onDrop($event, itempos)"
      @dragenter.prevent
      @dragover.prevent>
      <div
        v-for="todo in todos_asc"
        :class="`todo ${todo.completed && 'completed'}`"
        draggable="true"
        @dragenter.prevent
      @dragover.prevent
        @dragstart="onDragStart($event, todo.createdAt)"
      >
        <label>
          <input type="checkbox" v-model="todo.completed" id="checkbox" @click="checkCompleted"/>
        </label>&nbsp;&nbsp;&nbsp;
        <div>
          <input type="text" v-model="todo.content" id="updatetodo" />
        </div>
      </div>
      <div id="todosAbout">
        <div class="uncompleted">{{ todos.value }} items left</div>
        <div id="filter">
          <a href="">All</a>
          <a href="">Active</a>
          <a href="">Completed</a>
        </div>
        <div class="clearCompleted">
          <a href="">Clear Completed</a>
        </div>
      </div>
    </div>
  </div>
  <div></div>
</template>
