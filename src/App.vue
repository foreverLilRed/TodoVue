<script>
import { reactive, onMounted, watch, computed } from 'vue';
import TodoComponent from './components/TodoComponent.vue';

export default {
  components: {
    TodoComponent
  },
  setup() {
    const todos = reactive([]);

    const sortOrder = reactive({
      by: 'diasRestantes', 
      ascending: true
    });

    onMounted(() => {
      const storedTodos = localStorage.getItem('todos');
      if (storedTodos) {
        const parsedTodos = JSON.parse(storedTodos);
        todos.push(...parsedTodos);
      } else {
        todos.push({
          id: 1,
          nombre: 'Realizar una tarea',
          tiempo: '2025-01-01',
          prioridad: 'Baja',
        });
      }
    });
    
    watch(todos, (newTodos) => {
      localStorage.setItem('todos', JSON.stringify(newTodos));
    }, { deep: true });

    const updateTodo = (updatedTodo) => {
      const index = todos.findIndex(todo => todo.id === updatedTodo.id);
      if (index !== -1) {
        todos[index] = updatedTodo;
      }
    };

    const deleteTodo = (todoId) => {
      const todo = todos.find(todo => todo.id === todoId);
      if (todo) {
        const confirmed = confirm(`¿Estás seguro de que deseas eliminar la tarea: "${todo.nombre}"?`);
        if (confirmed) {
          const index = todos.findIndex(todo => todo.id === todoId);
          if (index !== -1) {
            todos.splice(index, 1);
          }
        }
      }
    };

    const calculateDaysRemaining = (dateString) => {
      const today = new Date();
      const targetDate = new Date(dateString);
      const timeDiff = targetDate - today;
      const daysRemaining = Math.ceil(timeDiff / (1000 * 60 * 60 * 24));
      return daysRemaining;
    };

    const sortedTodos = computed(() => {
      return [...todos].sort((a, b) => {
        const aDays = calculateDaysRemaining(a.tiempo);
        const bDays = calculateDaysRemaining(b.tiempo);

        if (sortOrder.ascending) {
          return aDays - bDays;
        } else {
          return bDays - aDays;
        }
      });
    });

    const toggleSortOrder = () => {
      sortOrder.ascending = !sortOrder.ascending;
    };

    const newTodo = reactive({
      nombre: '',
      tiempo: '',
      prioridad: ''
    });

    const addTodo = (event) => {
      event.preventDefault();

      if (!newTodo.nombre || !newTodo.tiempo || !newTodo.prioridad) {
        alert('Todos los campos son obligatorios');
        return;
      }

      const newTask = {
        id: todos.length + 1,
        nombre: newTodo.nombre,
        tiempo: newTodo.tiempo,
        prioridad: newTodo.prioridad,
        estado: 'Iniciado'
      };

      todos.push(newTask);

      newTodo.nombre = '';
      newTodo.tiempo = '';
      newTodo.prioridad = '';
    };

    return { todos, sortedTodos, updateTodo, deleteTodo, newTodo, addTodo, toggleSortOrder };
  }
};
</script>

<template>
  <div class="container mx-auto p-4 mt-10 w-11/12">
    <form @submit="addTodo" class="grid md:grid-cols-4 gap-x-4 sm:grid-cols-2 mb-4">
      <div class="mb-5">
        <label for="nombre" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Nombre</label>
        <input v-model="newTodo.nombre" type="text" id="nombre" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" placeholder="Nueva expo..." required />
      </div>
      <div class="mb-5">
        <label for="limite" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Fecha limite</label>
        <input v-model="newTodo.tiempo" type="date" id="limite" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" required />
      </div>
      <div class="mb-5">
        <label for="prioridad" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Prioridad</label>
        <select v-model="newTodo.prioridad" id="prioridad" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" required>
          <option value="" disabled>Elije una prioridad</option>
          <option value="Baja">Baja</option>
          <option value="Media">Media</option>
          <option value="Alta">Alta</option>
        </select>
      </div>
      <div class="flex items-center">
          <button type="submit" class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm w-full px-5 py-2.5 text-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800">Agregar</button>
      </div>
    </form>
    <div v-if="sortedTodos.length > 0" class="relative overflow-x-auto shadow-md sm:rounded-lg">
      <table class="w-full text-sm text-left rtl:text-right text-gray-500 dark:text-gray-400">
        <thead class="text-xs text-center text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
          <tr>
            <th scope="col" class="px-6 py-3">Tarea</th>
            <th scope="col" class="px-6 py-3">Límite</th>
            <th scope="col" class="px-6 py-3 cursor-pointer" @click="toggleSortOrder">Días Restantes</th>
            <th scope="col" class="px-6 py-3">Prioridad</th>
            <th scope="col" class="px-6 py-3">Estado</th>
            <th scope="col" class="px-6 py-3"><span class="sr-only">Edit</span></th>
            <th scope="col" class="px-6 py-3"><span class="sr-only">Delete</span></th>
          </tr>
        </thead>
        <tbody>
          <TodoComponent
            v-for="todo in sortedTodos"
            :key="todo.id"
            :todo="todo"
            @update-todo="updateTodo"
            @delete-todo="deleteTodo"
          />
        </tbody>
      </table>
    </div>
    <div v-else class="text-center">
      Aun no tienes tareas creadas
    </div>
  </div>
</template>
