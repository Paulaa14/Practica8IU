<!-- 
  Una tabla que permite filtrar sus filas y ordenar según columnas.
  Los datos son de la forma [{col1: valor1, ...}, ...] - cada fila, un objeto con valor para cada columna
  Para ello, necesita saber de qué tipo son las columnas - ver documantación de prop 'columns'
  Los filtros son los que genera FilterOrAddBox - prop 'filter'
-->
<template>
  <div class="page-container">
    <!-- Tabla -->
    <div class="sortable-grid">
      <table v-if="filteredData.length">
        <thead>
          <tr>        
            <th v-for="col in columns" :key="col.key"
            @click="sortBy(col.key)"
            :class="sorter.length && sorter.slice(-1)[0].key == col.key ? 'ghead active' : 'ghead'">
            <span v-if="sorter.find(o => o.key==col.key && o.order==1)" class="arrow asc"></span>
            <span v-if="sorter.find(o => o.key==col.key && o.order==-1)" class="arrow dsc"></span>
            {{ col.display }}
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="entry in filteredData" :key="entry.id" @click="selectRow(entry)">
            <td
              v-for="(col, index) in columns"
              :key="`${entry.id}_${col.key}`"
              class="text-start"
            >
              <template v-if="col.type === 'Enum'">
                {{ col.displayValues[entry[col.key]]}}
              </template>
              <template v-else-if="col.type === 'ObjectIds' ">
                <span v-for="o in entry[col.key].map(v => gState.resolve(v))" :key="o.id"
                class="small"
                  :data-bs-toggle="col?.title ? 'tooltip' : null"
                  :data-bs-html="col?.title ? true : null"
                  :data-bs-title="col?.title ? col.title(entry, col.key, o) : null"
                >
                  {{ o.name }}
                </span>
              </template>
              <template v-else>
                <span
                  :class="index == 0 ? 'name' : null"
                  :data-bs-toggle="col?.title ? 'tooltip' : null"
                  :data-bs-html="col?.title ? true : null"
                  :data-bs-title="col?.title ? col.title(entry, col.key) : null"
                >
                  {{ entry[col?.key]}}
                </span>
              </template>
            </td>
          </tr>
        </tbody>
      </table>
      <p v-else>(No hay nada que mostrar)</p>
    </div>

    <!-- Panel de detalles -->
<div class="details-pane" :class="{ visible: selectedRow !== null }">
  <button class="close-button" @click="closeDetails">Cerrar</button>
  <div v-if="selectedRow">
    <h3>Detalles</h3>
    <table>
      <tbody>
        <tr v-for="(value, key) in getDetailsForRow(selectedRow)" :key="key">
          <td><strong>{{ key }}</strong></td>
          <td>
            <div v-if="Array.isArray(value)">
              <table v-if="key === 'schedule'">
                <thead>
                  <tr>
                    <th>Hora</th>
                    <th>Actividad</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(entry, index) in value" :key="index">
                    <td>{{ entry.time }}</td>
                    <td>{{ entry.activity }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
            <div v-else>
              <template v-if="key === 'Enum'">
                {{ value.displayValues[value] || 'N/A' }}
              </template>
              <template v-else-if="Array.isArray(value)">
                <span v-for="item in value" :key="item.id">{{ item.name || 'Sin nombre' }}</span>
              </template>
              <span v-else>{{ value || 'N/A' }}</span>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</div>

  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import { gState } from '../state.js';
import { mkValueSorter, mkKeySorter, mkArraySorter, mkTimeSorter, mkReverseSorter} from '../util.js';



// Estado del ordenamiento
const sorter = ref([]); // [{ key: colName, order: 1|-1 } ...]

// Props
const props = defineProps({
  data: Array, // Datos de las filas [{ colName1: colValue1, ... }, ...]
  filter: Object, // Filtros { all: String, fields: Object }
  columns: Array, // Columnas [{ key: String, display: String, type: Enum | ObjectIds | String | Number }]
});

// Fila seleccionada
const selectedRow = ref(null);

// Resolver objetos (para `ObjectIds`)
function resolveObjects(ids) {
  if (!Array.isArray(ids)) return [];
  return ids.map(id => gState.resolve(id)).filter(Boolean);
}

/**
   * Column objects:
   *  key - the column key. Each element of data must have a value for each of these keys
   *  display - the user-friendly name of the column. Used in the column header, and in filter fields for that column
   *  type - the type of the column. Used to choose how to sort it and how to display its values. Can be one of:
   *    String - strings
   *    Number - numbers
   *    ObjectIds - an array of objectIds, which when looked up via gState.resolve(oId) results in actual model objects
   *    Enum - one of a (closed) set of values, which use a special order. For example, days of the week. 
   *  values - (only for type Enum) - the set of values in their correct order. Used for sorting
   *  title - (only for type ObjectIds) - a function that generates html tooltips for elements of this type
   *    will be called passing in (dataRow, colName, objectValue), where the objectValue is the result of
   *    calling gState.resolve(dataRow[colName]). If absent, no tooltip
   */
  
const filteredData = computed(() => {
  let { data, filter, columns} = props
    if (filter.all) {
      const needle = filter.all.toLowerCase()
      const keys = columns.map(o => o.key)
      const rowText = (row) => keys.map(k => row[k]).join('$').toLowerCase();
      data = data.filter(row => rowText(row).indexOf(needle) > -1)
    } else if (Object.keys(filter.fields).length) {
      for (let [k, v] of Object.entries(filter.fields)) {
        if (v.length == 0) continue;
        const needle = v.toLowerCase()
        data = data.filter(row => (""+row[k]).toLowerCase().indexOf(needle) > -1)
      }
    }

    if (sorter.value.length) {
      data = data.slice()
      for (let s of sorter.value) {
        const sortCol = columns.find(c => c.key == s.key);
        if ( ! sortCol) {
          console.log("ERROR: cannot sort by ", s, "cols are", columns)
          continue;
        }
        let sorter = null;
        switch (sortCol.type) {
          case "Enum": 
            sorter = mkKeySorter(sortCol.values, s.key); break;
          case "ObjectIds":
            sorter = mkArraySorter(s.key); break;
          case "Time":
            sorter = mkTimeSorter(s.key); break;
          default: 
            sorter = mkValueSorter(s.key);
        }
        data = data.sort(s.order == 1 ? sorter : mkReverseSorter(sorter))
      }
    }
    return data
  })


  /**
   * Se puede ordenar por varias columnas. La última es la que más ordena.
   * 1er click en col: ordena ascendente
   * 2do click en col: ordena descendente
   * 3er click en col: deja de ordenar (siguiente será 1ero)
   * @param key columna cuyo orden alterar
   */ 
// Manejo del orden
function sortBy(key) {
    const prev = sorter.value.find(o => o.key == key)
    if (prev && prev.order == 1) {
      prev.order = -1;
    } else if (prev) {
      // equivalent to sorter = sorter.filter(o => o.key != key)
      // but we cannot re-asign v-models (or properties), so we use this instead
      sorter.value.splice(sorter.value.findIndex(o => o.key == key), 1);      
    } else {
      sorter.value.push({key, order: 1})
    }
  }

// Seleccionar fila
function selectRow(row) {
  selectedRow.value = row;
}

// Cerrar panel
function closeDetails() {
  selectedRow.value = null;
}

// Función para obtener detalles específicos de la fila seleccionada
function getDetailsForRow(row) {
  if (!row) return {};

  const details = {};

  // Aquí puedes recorrer las columnas para mapear los detalles, igual que en la tabla
  for (let col of props.columns) {
    const value = row[col.key];

    if (col.type === 'Enum') {
      details[col.display] = col.displayValues[value] || 'N/A';
    } else if (col.type === 'ObjectIds' && Array.isArray(value)) {
      details[col.display] = resolveObjects(value);
    } else {
      details[col.display] = value || 'N/A';
    }
  }

  return details;
}
</script>

<style>
.page-container {
  position: relative;
}

.sortable-grid {
  margin: 20px 0;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 10px;
  border: 1px solid #ddd;
  text-align: left;
}

th.ghead {
  white-space: nowrap;
}

.arrow.asc::after {
  content: "↓";
  position: relative;
  left: 4px;
}

.arrow.dsc::after {
  content: "↑";
  position: relative;
  left: 4px;
}

.details-pane {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.8);
  color: white;
  padding: 20px;
  display: none;
  z-index: 9999;
  overflow-y: auto;
}

.details-pane.visible {
  display: block;
}

.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
  background-color: #ff6b6b;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
}

.close-button:hover {
  background-color: #e63946;
}

h3 {
  font-size: 24px;
  margin-bottom: 20px;
}

h4 {
  font-size: 20px;
  margin-top: 10px;
}

small {
  font-size: 0.8em;
}

span {
  font-weight: normal;
  font-size: 1rem;
}

span.name {
  font-weight: 1000;
}
span.small {
  padding-right: 2px;
}
table {
  margin-top: 10px;

}
thead>tr {
  border-bottom: 1px solid gray;
  color: rgb(104, 103, 103);
}
</style>
