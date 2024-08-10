<script setup lang="ts">
import '@/assets/main.css';
import { onMounted, ref } from 'vue';
import type { Schema } from '../../amplify/data/resource';
import { generateClient } from 'aws-amplify/data';

const client = generateClient<Schema>();

// create a reactive reference to the array of todos
const todos = ref<Array<Schema['Todo']["type"]>>([]);

function listTodos() {
  client.models.Todo.observeQuery().subscribe({
    next: ({ items, isSynced }) => {
      todos.value = items
     },
  }); 
}

function createTodo() {
  client.models.Todo.create({
    content: window.prompt("Todo content")
  }).then(() => {
    // After creating a new todo, update the list of todos
    listTodos();
  });
}
      
function deleteTodo(id: string) {
  client.models.Todo.delete({ id })
}

import { uploadData } from "aws-amplify/storage";
function initUpload() {
  const file = document.getElementById("file") as HTMLInputElement;
  const upload = document.getElementById("upload") as HTMLElement;

  upload.addEventListener("click", () => {
    const fileReader = new FileReader();
    // @ts-ignore
    fileReader.readAsArrayBuffer(file.files[0]);

    fileReader.onload = async (event) => {
      // @ts-ignore
      console.log("Complete File read successfully!", event.target.result);
      try {
        await uploadData({
          // @ts-ignore
          data: event?.target?.result,
          // @ts-ignore
          path: `picture-submissions/${file.files[0].name}`
        });
      } catch (e) {
        console.log("error", e);
      }
    };
  });
}

// fetch todos when the component is mounted
 onMounted(() => {
  listTodos();
  initUpload();
});

</script>

<template>
  <main>
    <h1>My pictures</h1>
    <input type="file" id="file"/>
    <button id="upload">Upload</button>
    <h1>My todos</h1>
    <button @click="createTodo">+ new</button>
    <ul>
      <li 
        v-for="todo in todos" 
        :key="todo.id"
        @click="deleteTodo(todo.id)"
        >
        {{ todo.content }}
      </li>
    </ul>
    <div>
      🥳 App successfully hosted. Try creating a new todo.
      <br />
      <a href="https://docs.amplify.aws/gen2/start/quickstart/nextjs-pages-router/">
        Review next steps of this tutorial.
      </a>
    </div>
  </main>
</template>
