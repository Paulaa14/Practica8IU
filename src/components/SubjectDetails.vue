<template>
<h4>Asignatura <span class="name">{{ subject.short }}</span></h4>
    <table>
      <tbody>
        <tr>
          <th>Nombre</th>
          <td>{{ subject.name }} </td>
        </tr>
        <tr>
          <th>Créditos</th>
          <td>{{ subject.credits }} </td>
        </tr>
        <tr>
          <th>Cuatrimestre</th>
          <td>{{ subject.semester }} </td>
        </tr>
        <tr>
          <th>Códigos GEA</th>
          <td>{{ subject.codes.replaceAll('-', ', ') }} </td>
        </tr>
        <tr>
          <th>Grupos</th>
          <td v-if="subject.groups.length">
            {{ subject.groups.map(g => gState.resolve(g).name).join(' ') }}
          </td>
          <td v-else> (ninguno) </td>
        </tr>
      </tbody>
    </table>

    <h5>Acciones</h5>
    <div class="btn-group">
      <button @click="$emit('editSubject')" class="btn btn-outline-success" title = "Editar asignatura">✏️</button>
      <button @click="confirmDelete()" :disabled="subject.groups.length !== 0" class="btn btn-outline-danger" title = "Eliminar asignatura">🗑️</button>
    </div>
</template>

<script setup>

import { gState } from '../state.js';

const emit = defineEmits([
  'editSubject',
  'rmSubject',
])

const props = defineProps({
  subject: Object // see definition of Subject in ../model.js
})

const confirmDelete = () => {
  if (confirm(`¿Estás seguro de que quieres borrar la asignatura ${props.subject.name}?`)) {
    emit('rmSubject')
    alert('Asignatura borrada con exito')
  }
}

</script>