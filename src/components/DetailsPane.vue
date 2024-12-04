<template>
  <div v-if="!element || element.id == -1" class="empty">
    (selecciona una fila para ver sus detalles)
  </div>
  <div v-else-if="typeof element.id == 'string'">
    <LocationDetails :location="element.id.split('_')[0]" :semester="element.id.split('_')[1]" />
  </div>
  <div v-else-if="element.firstName">
    <UserDetails :user="element" 
    @editUser="$emit('editUser')" @rmUser="$emit('rmUser')" />
  </div>
  <div v-else-if="element.codes">
    <SubjectDetails :subject="element" 
    @editSubject="$emit('editSubject')" @rmSubject="$emit('rmSubject')" />
  </div>
  <div v-else-if="element.slots">
    <GroupDetails :group="element" 
    @editGroup="$emit('editGroup')" @rmGroup="$emit('rmGroup')" />
  </div>
</template>

<script setup>

import LocationDetails from './LocationDetails.vue';
import UserDetails from './UserDetails.vue';
import SubjectDetails from './SubjectDetails.vue';
import GroupDetails from './GroupDetails.vue';

defineEmits([
  'editUser',
  'rmUser',
  'editSubject',
  'rmSubject',
  'editGroup',
  'rmGroup',
  'goBack'
]);

defineProps({
  element: Object // a User, Subject, Group, Slot or Location; use {id: -1} for "nothing"
});

</script>

<style scoped>

tr > th {
  width: 10em;
  text-align: right;
}


/* Estilo de la sección vacía */
.empty {
  margin: 2em;
  text-align: center;
  font-size: 1rem;
  padding: 15px;
  line-height: 1.5;
}

/* Media query para pantallas móviles */
@media (max-width: 600px) {
  .details-container {
    font-size: 14px;
    padding: 0.8em;
  }

  .empty {
    font-size: 0.9rem;
    padding: 15px;
    margin: 1.5em;
  }

  h5 {
    font-size: 1.2rem;
  }

  tr > th {
    width: auto; /* Asegura que las celdas de las tablas se ajusten al contenido */
  }

  td, th {
    font-size: 14px; /* Asegura que las celdas de las tablas tengan una fuente más pequeña */
  }
}

.name {
  font-weight: 1000;
}

.dayOfWeek {
  display: inline-block;
  width: 4em;
}

td,
th {
  padding: 4px;
}

h5 {
  margin-top: 1em;
}
</style>