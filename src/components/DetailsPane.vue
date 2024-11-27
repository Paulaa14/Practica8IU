<template>
  <div class="details-container">
    <button class="mobile-nav-button" @click="handleGoBack">
      Volver al listado
    </button>  /*boton volver a arriba*/
  </div>
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

const emit = defineEmits([
  'editUser',
  'rmUser',
  'editSubject',
  'rmSubject',
  'editGroup',
  'rmGroup',
  'goBack'
])

defineProps({
  element: Object // a User, Subject, Group, Slot or Location; use {id: -1} for "nothing"
})

const handleGoBack = () => {/*boton volver a arriba*/
  if (typeof window.scrollTo === 'function') {
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }
  emit('goBack');
};

</script>

<style scoped>
.mobile-nav-button{/*boton volver a arriba*/
  position: fixed;
  bottom: 10px;
  right: 10px;
  z-index: 1000;
  padding: 10px 20px;
  background-color: #007bff;
  color:white;
  border: none;
  border-radius: 5px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  font-size: 14px;
  cursor: pointer;
}

.mobile-nav-button:hover{/*boton volver a arriba*/
  background-color: #0056b3;
}

.details-container{/*boton volver a arriba*/
  padding:1em;
  font-size: 16px;
}

tr>th {
  width: 10em;
  text-align: right;
}

.empty {/*Tamaño pantalla, q se ajuste a un móvil*/
  margin: 2em;
  text-align: center;
  font-size: 1rem;
}

@media (max-width: 600px){/*Tamaño pantalla, q se ajuste a un móvil*/
  .details-container{
    font-size: 14px;
  }

  .empty{
    font-size: 0.9rem;
    padding: 20px;
  }

  h5{
    font-size: 1.1rem;
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