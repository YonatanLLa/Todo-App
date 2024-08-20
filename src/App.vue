<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import { Todo } from './types/todo'

const todos = ref<Todo[]>([])
const input_content = ref<string>('')
const editingTodo = ref<Todo | null>(null) 

// Guarda las tareas en localStorage cuando cambian
watch(todos, (newVal) => {
  localStorage.setItem('todos', JSON.stringify(newVal))
}, { deep: true })

const addOrUpdateTodo = (): void => {
  if (input_content.value.trim() === '') return
  if (editingTodo.value) {
    editingTodo.value.content = input_content.value
    editingTodo.value.editable = false 
    editingTodo.value = null 
  } else {
    todos.value.push({
      content: input_content.value,
      done: false,
      editable: false, 
      createdAt: new Date().getTime()
    })
  }

  input_content.value = '' 
}

// Elimina una tarea de la lista
const removeTodo = (todo: Todo): void => {
  todos.value = todos.value.filter((t) => t !== todo)
}

// Activa el modo de edición
const editTodo = (todo: Todo): void => {
  editingTodo.value = todo
  input_content.value = todo.content
  todo.editable = true
}

// Carga las tareas desde localStorage al montar el componente
onMounted((): void => {
  const savedTodos = localStorage.getItem('todos')
  if (savedTodos) {
    todos.value = JSON.parse(savedTodos) as Todo[]
  }
})
</script>

<template>
  <main class="p-4 max-w-2xl mx-auto bg-gray-800 min-h-screen">
    <section class="bg-white p-6 rounded-lg shadow-md mb-6">
      <h3 class="text-xl font-semibold mb-4 text-center">{{ editingTodo ? 'EDITAR TAREA' : 'CREAR UNA TAREA' }}</h3>
      <form @submit.prevent="addOrUpdateTodo" class="flex flex-col gap-4">
        <input 
          type="text" 
          placeholder="Nueva tarea"
          v-model="input_content"
          class="p-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500" />
        <button 
          type="submit" 
          class="bg-blue-500 text-white py-2 px-4 rounded-md hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500">
          {{ editingTodo ? 'Guardar Cambios' : 'Agregar' }}
        </button>
      </form>
    </section>

    <section class="bg-white p-6 rounded-lg shadow-md">
      <h3 class="text-xl font-semibold mb-4 text-center">LISTA DE TAREAS</h3>
      <div class="space-y-4">
        <div v-for="todo in todos" :key="todo.createdAt" class="flex flex-col justify-between items-center p-4 border border-gray-300 rounded-md bg-gray-50">
          <div class="flex items-center justify-between w-full pb-4">
            <input 
                type="checkbox" 
                v-model="todo.done"
                class="custom-checkbox " />
            <div :class="{'line-through text-gray-500': todo.done}" class="flex-1 pl-4">{{ todo.content }}</div>
          </div>
          <div class="h-[1px] w-[50%] bg-gray-300">
             
          </div>
          <div class="mt-2 sm:mt-0 flex items-center pt-2  space-x-2">
            <button 
              @click="editTodo(todo)"
              class="bg-yellow-500 text-white py-1 px-3 rounded-md hover:bg-yellow-600 focus:outline-none focus:ring-2 focus:ring-yellow-500">
              Editar
            </button>
            <button 
              @click="removeTodo(todo)"
              class="bg-red-500 text-white py-1 px-3 rounded-md hover:bg-red-600 focus:outline-none focus:ring-2 focus:ring-red-500">
              Eliminar
            </button>
        
          </div>
        </div>
      </div>
    </section>
  </main>
</template>


<style>
.custom-checkbox {
  appearance: none;
  background-color: #fff;
  border: 2px solid #d1d5db;
  border-radius: 50%;
  width: 24px;
  height: 24px;
  position: relative;
  cursor: pointer;
  outline: none;
}

.custom-checkbox:checked {
  background-color: #2563eb;
  border-color: #2563eb;
}

.custom-checkbox:checked::before {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background-color: #fff;
}
</style>

