<template>
    <div class="row g-1 form-inline align-items-baseline">
        <div class="col-3 text-end">
            <label :for="id" class="form-label">{{ label }}</label>
        </div>
        <div class="col-auto">
            <select class="form-select" :name="id" :id="id" @input="$emit('change', value);">
                <option value="FALL">1er Cuatrimestre</option>
                <option value="SPRING">2º Cuatrimestre</option>
            </select>
            <span>{{value}}</span>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const props = defineProps({
    label: String,
    id: String,
    start: String,
})
defineEmits(['change'])

let value = ref(0)

onMounted(() => {
    value.value = props.start;
})

</script>

<style scoped>
* {
    background-color: lightblue;
}
</style>