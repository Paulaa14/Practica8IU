<template>
  <div class="row">
    <div class="col-auto w-75">
      <div class="input-group">
        <input v-if="!advSearch" v-model="filter.all" type="search" 
          class="form-control" placeholder="Filtrar">
        <input v-else value="(filtrando por campos)" disabled type="search" 
          class="form-control" placeholder="Filtrar">
        
        <!-- Lupa sin parecer botón -->
        <span class="input-group-text lupa-icon">🔍</span>

        <button type="button" class="input-group-text btn btn-outline-secondary b-avanzada"
          @click="toggleAdvanced()" title="Búsqueda avanzada">⚙️</button>
      </div>
    </div>
    <div class="col-auto">
      <button type="button" class="btn btn-outline-secondary" 
        @click="clearFilters()" title="Limpiar filtros">🧹</button>
    </div>
    <div v-if="addBtnTitle" class="col-auto">
      <button type="button" :title="addBtnTitle" 
        @click="$emit('addElement')" class="btn btn-outline-primary">➕</button>
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

// eslint genera un error falso a menos que se evite con este comentario:
// eslint-disable-next-line
const filter = defineModel('filter') // {all: '', fields: {keyCol1: '', ...}}

defineProps({
  addBtnTitle: String,
  columns: Array,     // de {key: , display:, type:, [values: ]}; ver SortableGrid  
})

defineEmits(['addElement'])

const advSearch = ref(filter.value.fields.length > 0)
const id = useId()

function updateValue(key) {
  const prev = { ...filter.value.fields }
  prev[key] = document.getElementById(`${id}_${key}`).value
  filter.value = { all: '', fields: prev }
}

// Vacía la parte del filtro que no esté seleccionada
function toggleAdvanced() {
  advSearch.value = !advSearch.value
  if (advSearch.value) {
    filter.value.all = ''
  } else {
    filter.value.fields = {}
  }
}

function clearFilters() {
  filter.value = { all: '', fields: {} }
}
</script>

<style scoped>
/* Resalta el botón de búsqueda avanzada */
.btn.active.b-avanzada {
  background-color: lightblue;
}

/* Icono de la lupa */
.lupa-icon {
  background-color: white !important;
  border: 1px solid #ced4da; /* Coincide con el borde de Bootstrap */
  border-left: none; /* Quita el borde izquierdo */
  color: #495057; /* Color del texto */
  padding: 0.5rem 0.75rem;
  font-size: 1.25rem;
  cursor: default;
}

/* Alineación de la etiqueta */
.first {
  width: 5em;
}
</style>