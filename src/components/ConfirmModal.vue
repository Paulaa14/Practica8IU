<template>
    <BaseModal ref="modalRef" id="confirmModal"
      :title="'Confirma acción peligrosa'">
      <template #body>
        <span>{{ text }}</span>
      </template>
      <template #footer>
        <button @click.prevent="() => action()" class="btn btn-primary">
          {{ 'Confirmar acción' }}
        </button>
      </template>
    </BaseModal>
</template>

<script setup>

import BaseModal from './BaseModal.vue';

import { ref } from 'vue'

const props = defineProps({
  text: String,
  action: Function
})

let modalRef = ref(null);

// para que el padre pueda llamar a show (hide no debería hacer falta)
function show() {
  modalRef.value.show();
}
defineExpose({ show });
</script>

<style scoped>
    .name {
        font-weight: 200%;
    }
</style>