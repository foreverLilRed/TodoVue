<script>
import { reactive, toRefs, computed } from 'vue';

export default {
  props: {
    todo: Object,
  },
  emits: ['update-todo'],
  setup(props, { emit }) {
    const { todo } = toRefs(props);
    const editing = reactive({
      nombre: false,
      tiempo: false,
      prioridad: false,
      estado: false,
    });

    const toggleEdit = (field) => {
      editing[field] = true;
    };

    const saveEdit = (field, event) => {
      todo.value[field] = event.target.value;
      editing[field] = false;
      emit('update-todo', todo.value);
    };

    const availableTime = computed(() => {
      const today = new Date();
      const deadline = new Date(todo.value.tiempo);
      const timeDifference = deadline - today;
      const daysDifference = Math.ceil(timeDifference / (1000 * 60 * 60 * 24));
      return daysDifference;
    });

    const colorPriority = computed(()=> {
      const priority = todo.value.prioridad;
      switch(priority){
        case "Baja":
          return 'text-green-600';
        case "Media":
          return 'text-blue-700';
        case "Alta":
          return 'text-red-600';
      }
    });

    const colorStatus = computed(()=> {
      const status = todo.value.estado;
      switch(status){
        case "Iniciado":
          return 'text-green-600';
        case "En proceso":
          return 'text-blue-700';
        case "Terminado":
          return 'text-red-600';
      }
    });


    const colorTime = computed(() => {
      const days = availableTime.value;
      if (days <= 7) {
        return 'text-red-600';
      } else if (days > 7 && days < 30) {
        return 'text-blue-700';
      } else {
        return 'text-green-600';
      }
    });

    const deleteTodo = () => {
      emit('delete-todo', todo.value.id);
    };

    return { todo, editing, toggleEdit, saveEdit, availableTime, deleteTodo, colorTime, colorPriority, colorStatus };
  }
}
</script>

<template>
  <tr class="bg-white border-b dark:bg-gray-800 dark:border-gray-700 hover:bg-gray-50 dark:hover:bg-gray-600">
    <th scope="row" class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap dark:text-white">
      <div v-if="editing.nombre">
        <input v-model="todo.nombre" @blur="saveEdit('nombre', $event)" class="border p-1"/>
      </div>
      <div v-else @dblclick="toggleEdit('nombre')">
        {{ todo.nombre }}
      </div>
    </th>
    <td class="px-6 py-4">
      <div v-if="editing.tiempo">
        <input v-model="todo.tiempo" @blur="saveEdit('tiempo', $event)" class="border p-1"/>
      </div>
      <div v-else @dblclick="toggleEdit('tiempo')">
        {{ todo.tiempo }}
      </div>
    </td>
    <td class="px-6 py-4" :class="colorTime">
      <b>{{ availableTime }} días</b>
    </td>
    <td class="px-6 py-4">
      <div v-if="editing.prioridad">
        <select v-model="todo.prioridad" @blur="saveEdit('prioridad', $event)" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" required>
          <option value="Baja">Baja</option>
          <option value="Media">Media</option>
          <option value="Alta">Alta</option>
        </select>
      </div>
      <div v-else @dblclick="toggleEdit('prioridad')" :class="colorPriority">
        <b>{{ todo.prioridad }}</b>
      </div>
    </td>
    <td class="px-6 py-4">
      <div v-if="editing.estado">
        <select v-model="todo.estado" @blur="saveEdit('estado', $event)" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" required>
          <option value="Iniciado">Iniciado</option>
          <option value="En proceso">En proceso</option>
          <option value="Terminado">Terminado</option>
        </select>
      </div>
      <div v-else @dblclick="toggleEdit('estado')" :class="colorStatus">
        <b>{{ todo.estado }}</b>
      </div>
    </td>
    <td class="px-6 py-4 text-right">
      <svg @click="deleteTodo" class="cursor-pointer" fill="#ff0000" width="20px" height="20px" viewBox="0 0 32 32" version="1.1" xmlns="http://www.w3.org/2000/svg" stroke="#ff0000"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <title>cancel2</title> <path d="M19.587 16.001l6.096 6.096c0.396 0.396 0.396 1.039 0 1.435l-2.151 2.151c-0.396 0.396-1.038 0.396-1.435 0l-6.097-6.096-6.097 6.096c-0.396 0.396-1.038 0.396-1.434 0l-2.152-2.151c-0.396-0.396-0.396-1.038 0-1.435l6.097-6.096-6.097-6.097c-0.396-0.396-0.396-1.039 0-1.435l2.153-2.151c0.396-0.396 1.038-0.396 1.434 0l6.096 6.097 6.097-6.097c0.396-0.396 1.038-0.396 1.435 0l2.151 2.152c0.396 0.396 0.396 1.038 0 1.435l-6.096 6.096z"></path> </g></svg>
    </td>
  </tr>
</template>
