<!--
    Muestra un filtro de la forma {all: '', fields: {}} y un botón de añadir elemento
    El filtro tiene dos modos: normal y por campos, y es posible cambiar entre estos modos.
    - modo normal deja fields: a {}
    - modo avanzado deja all: a ''

    Si no estás en modo avanzado, el modelo es filter.all con el que buscas por 1 unico campo. Sino es filter.fields y desactiva la entrada de texto,
    no te deja escribir.

    El title es el texto alternativo del botón
-->
<template>
  <div class="row">
    <div class="col-auto w-75">
      <div class="input-group">
        <input v-if="!advSearch" v-model="filter.all" type="search" 
          class="form-control" placeholder="Filtrar">
        <input v-else value="(filtrando por campos)" disabled="disabled" type="search" class="form-control"
          placeholder="Filtrar">
        <span class="input-group-text btn-outline-secondary">🔍</span>
        <button type="button" data-bs-toggle="button" class="input-group-text btn btn-outline-secondary b-avanzada"
          @click="toggleAdvanced()" title="Búsqueda avanzada">⚙️</button> <!-- Al pulsar el botón cambia el modo con el toggleAdvanced-->
      </div>
    </div>
    <div class="col-auto"> <!-- BOTÓN DE LIMPIAR FILTROS -->
      <button type="button" data-bs-toggle="button" class="btn btn-outline-secondary" @click="clearFilters()" title="Limpiar filtros">🧹</button>
    </div>
    <div v-if="addBtnTitle" class="col-auto">
      <button type="button" :title="addBtnTitle" @click="$emit('addElement')" class="btn btn-outline-primary">➕</button>
    </div>
  </div>
  <div v-if="advSearch" class="row mt-3">
    <div class="col-auto">
      <div v-for="col in columns" :key="col.key">
        <div class="row g-3 align-items-center">
          <div class="col-auto first">
            <label :for="`${id}_${col.key}`" class="col-form-label">{{ col.display }}</label>
          </div>
          <div class="col-auto">
            <input type="text" v-model="filter.fields[col.key]" :id="`${id}_${col.key}`" 
              @input="updateValue(col.key)" class="form-control">
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, useId } from 'vue'

// eslint generates (bogus) error unless prevented from doing so with this comment:
// eslint-disable-next-line
const filter = defineModel('filter') // {all: '', fields: {keyCol1: '', ...}}

defineProps({
  addBtnTitle: String,
  columns: Array,     // of {key: , display:, type:, [values: ]}; see SortableGrid  
})

defineEmits(['addElement'])

const advSearch = ref(filter.value.fields.length > 0)

const id = useId()

function updateValue(key) {
  const prev = {... filter.value.fields};
  prev[key] = document.getElementById(`${id}_${key}`).value
  filter.value = {all: '', fields: prev}
}

// vacía la parte del filtro que no esté seleccionada
function toggleAdvanced() {
  advSearch.value = !advSearch.value
  if (advSearch.value) {
    filter.value.all = ''
  } else {
    filter.value.fields = {}
  }
}

function clearFilters() {
  filter.value = {all: '', fields: {}};
}

</script>

<style scoped>
.btn.active.b-avanzada {
  background-color: lightblue;
}

.first {
  width: 5em;
}
</style>
