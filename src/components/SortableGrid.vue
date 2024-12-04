<template>
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
      <tr v-for="entry in filteredData" :key="entry.id" 
          @click="$emit('selectOne', entry.id)">
        <td v-for="(col, index) in columns" :key="`${entry.id}_${col.key}`"
          class="text-start">
          <template v-if="col.type == 'Enum'">
            {{ col.displayValues[entry[col.key]] }}
          </template>
          <template v-else-if="col.type == 'ObjectIds'">
            <span 
              v-for="o in entry[col.key].map(v => gState.resolve(v))" 
              :key="o.id" 
              class="badge badge-primary">
              {{ o.name }}
            </span>
          </template>
          <template v-else>
            <span 
              :class="index == 0 ? 'name' : null">
              {{ entry[col.key] }}
            </span>
          </template>
        </td>
      </tr>
    </tbody>
  </table>
  <p v-else>(no hay nada que mostrar)</p>
</template>

<script setup>
import { computed } from 'vue';
import { gState } from '../state.js';
import { mkValueSorter, mkKeySorter, mkArraySorter, mkTimeSorter, mkReverseSorter } from '../util.js';

// eslint generates (bogus) error unless prevented from doing so with this comment:
// eslint-disable-next-line
const sorter = defineModel('sorter'); // [{key: colName, order: 1 |-1} ...]

const props = defineProps({
  data: Array,       // [{colName1: colValue1, ...}, ...]
  filter: Object,    // { all: String, filter: [ {colName: String} ... ] } - see FilterOrAddBox
  columns: Array,    // [{key: colName, display:, type: String|Number|ObjectIds|Enum, } ...]
});

defineEmits(['selectOne']);

const filteredData = computed(() => {
  let { data, filter, columns } = props;
  if (filter.all) {
    const needle = filter.all.toLowerCase();
    const keys = columns.map(o => o.key);
    const rowText = (row) => keys.map(k => row[k]).join('$').toLowerCase();
    data = data.filter(row => rowText(row).indexOf(needle) > -1);
  } else if (Object.keys(filter.fields).length) {
    for (let [k, v] of Object.entries(filter.fields)) {
      if (v.length == 0) continue;
      const needle = v.toLowerCase();
      data = data.filter(row => ("" + row[k]).toLowerCase().indexOf(needle) > -1);
    }
  }

  if (sorter.value.length) {
    data = data.slice();
    for (let s of sorter.value) {
      const sortCol = columns.find(c => c.key == s.key);
      if (!sortCol) {
        console.log("ERROR: cannot sort by ", s, "cols are", columns);
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
      data = data.sort(s.order == 1 ? sorter : mkReverseSorter(sorter));
    }
  }
  return data;
});

/**
 * Se puede ordenar por varias columnas. La última es la que más ordena.
 * 1er click en col: ordena ascendente
 * 2do click en col: ordena descendente
 * 3er click en col: deja de ordenar (siguiente será 1ero)
 * @param key columna cuyo orden alterar
 */
function sortBy(key) {
  const prev = sorter.value.find(o => o.key == key);
  if (prev && prev.order == 1) {
    prev.order = -1;
  } else if (prev) {
    sorter.value.splice(sorter.value.findIndex(o => o.key == key), 1);
  } else {
    sorter.value.push({ key, order: 1 });
  }
}
</script>

<style scoped>
th.ghead {
  white-space: nowrap;
  text-align: center; /* Centra el texto en los encabezados */
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

span.name {
  font-weight: 1000;
}

.badge {
  display: inline-block;
  padding: 0.5em 1em;
  margin: 0.2em;
  border-radius: 0.25rem;
  color: white;
  font-size: 0.85rem;
}

.badge-primary {
  background-color: #007bff;
}

/* Añadido padding para filas y celdas */
table {
  margin-top: 10px;
}

thead > tr {
  border-bottom: 1px solid gray;
  color: rgb(104, 103, 103);
}

tbody > tr {
  border-bottom: 1px solid #ccc; /* Línea de separación entre filas */
}

tr {
  padding: 10px 0; /* Padding vertical */
}

td {
  padding: 10px; /* Padding general */
}

</style>
