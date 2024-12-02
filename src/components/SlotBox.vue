<!--
    Pensado para campo de formulario modal, permite gestionar horarios (slots)
    para un grupo
-->
<template>
  <div class="row g-1">
    <div class="col-3 text-end">
      <label :for="id" class="form-label">{{ label }}</label>
      <span> {{ groupId }}</span>
    </div>
    <div class="col-9 text-start">
      <div v-for="slot in current" :key="slot.id" class="caja">
        <select :id="`${id}-${slot.id}-day`" :name="`${id}-${slot.id}-day`" @input="setDay(slot)">
          <option v-for="d in Object.keys(weekDayNames)" :key="d" :value="d"
            :selected="slot.weekDay == d ? 'selected' : null">
            {{ weekDayNames[d] }}
          </option>
        </select>
        de
        <input type="time" :id="`${id}-${slot.id}-start`" :name="`${id}-${slot.id}-start`"
          :value="formatTime(slot.startTime)" @input="setStart(slot)" />
        a
        <input type="time" :id="`${id}-${slot.id}-end`" :name="`${id}-${slot.id}-end`" :value="formatTime(slot.endTime)"
          @input="setEnd(slot)" />
        en
        <input type="text" :id="`${id}-${slot.id}-location`" :name="`${id}-${slot.id}-location`" :value="slot.location"
          @input="setLocation(slot)" />
        <button type="button" @click="rm(slot)">🗑️</button>
      </div>
      <button type="button" @click="add">➕</button>
      <button type="button especial" @click.prevent="addSpecial">(➕)</button>
      <input type="hidden" :name="id" :id="id" :value="read">
    </div>
  </div>
</template>

<script setup>
import { gState, weekDayNames } from '../state.js'

import { ref, computed, onMounted } from 'vue'

const props = defineProps({
  label: String,
  id: String,
  start: Array,
  groupId: Number,
})

const current = ref([])

let lastId = -1;

onMounted(() => {
  current.value = props.start.map(id => gState.resolve(id));
})

function setDay(slot) {
  slot.weekDay = document.getElementById(`${props.id}-${slot.id}-day`).value
}

function setStart(slot) {
  slot.startTime = timeToHundreds(document.getElementById(`${props.id}-${slot.id}-start`).value)
}

function setEnd(slot) {
  slot.endTime = timeToHundreds(document.getElementById(`${props.id}-${slot.id}-end`).value)
}

function setLocation(slot) {
  slot.location = document.getElementById(`${props.id}-${slot.id}-location`).value
}

function setLastLocation(slot) {
  slot.location = document.querySelector('[id*="-location"]').value
}

function add() {
  // negative ids to be able to create the slots later
  current.value.push(new gState.model.Slot(
    --lastId, Object.keys(gState.model.WeekDay)[0], 900, 1000,
    "???", Object.keys(gState.model.WeekDay)[0], -1));
}

function addSpecial() {
  console.log("Hola mundo")
  let weekDay = Object.keys(gState.model.WeekDay)[0];
  let startTime = 900;
  let endTime = 2000;
  let nextId = --lastId;
  let valid = false;
  let s = null;
  for (let start = startTime; ! valid && start < endTime; start += 100) {
    let group = gState.resolve(props.groupId);
    console.log("group is", group)
    let subject = gState.resolve(group.subjectId);
    console.log("subject is", subject)
    s = new gState.model.Slot(nextId, weekDay, start, start + 100,
      "???" /*location*/, 
      subject.semester /*semester*/, 
      props.groupId /*groupId*/);
    setLastLocation(s);
    valid = !gState.model.overlapsOtherSlots(s, gState.model.getSlots());
  }
  current.value.push(s);
}

function rm(slot) {
  current.value.splice(current.value.findIndex(o => o.id == slot.id), 1);
}

const read = computed(() => {
  return JSON.stringify(current.value);
})

// note that 900 => "09:00" - this is what the input type="time" requires, it errors with "9:00" (!)
const formatTime = t => `0${Math.floor(t / 100)}`.slice(-2) + ':' + `0${t % 100}`.slice(-2)
// reverse operation: "09:00" => 900, "12:34" => 1234
const timeToHundreds = t => {
  const [h, m] = t.split(":")
  return (+h) * 100 + (+m)
}

</script>

<style scoped>
.exists {
  background-color: lightblue;
}

.special {
  border: 2px solid green;
}

.caja {
  display: inline-block;
  padding: 2px;
}
</style>