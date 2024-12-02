<template>
    <div class="row g-1">
        <div class="col-3 text-end">
            <label :for="id" class="form-label">{{ label }}</label>
        </div>
        <div class="col-9 text-start">
            <div v-for="o in possible" :key="o.id" class="badge-container">
                <span 
                    v-if="current.some(c => c.id == o.id)" 
                    class="badge exists" 
                    @click="rm(o.id)">
                    {{ o.name }}
                </span>
                <span 
                    v-else 
                    class="badge" 
                    @click="current.push(o)">
                    {{ o.name }}
                </span>
            </div>
            <input type="hidden" :name="id" :id="id" :value="read">
        </div>
    </div>
</template>

<script setup>
import { gState } from '../state.js';
import { ref, computed, onMounted } from 'vue';

const props = defineProps({
    label: String,
    id: String,
    start: Array,
    all: Array,
});

const current = ref([]);
const possible = ref([]);

onMounted(() => {
    current.value = props.start.map(id => gState.resolve(id));
    possible.value = props.all.toSorted((a, b) => a.name > b.name);
});

function rm(id) {
    current.value.splice(current.value.findIndex(o => o.id == id), 1);
}

const read = computed(() => {
    return JSON.stringify(current.value.map(o => o.id));
});
</script>

<style scoped>
.badge-container {
    display: inline-block;
    margin: 0.2em;
}

.badge {
    display: inline-block;
    padding: 0.5em 1em;
    background-color: #007bff;
    color: white;
    border-radius: 0.25rem;
    cursor: pointer;
    font-size: 0.85rem;
}

.badge.exists {
    background-color: lightblue;
}

.badge:hover {
    background-color: #0056b3;
}
</style>
