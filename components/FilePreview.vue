<template>
  <div v-if="show" @click.self="close" class="modal-backdrop file-preview-backdrop">
    <button type="button" @click="close" aria-label="Close" class="file-preview-close">✕</button>
    <div class="file-preview-title">{{ file?.name }}</div>
    <div class="file-preview-stage">
      <img
        v-if="isImage"
        :src="file?.data"
        :alt="file?.name"
        class="file-preview-content file-preview-content--image"
      />
      <object
        v-else-if="isPdf"
        :data="file?.data"
        type="application/pdf"
        class="file-preview-content file-preview-content--pdf"
      ></object>
      <div v-else class="file-preview-fallback">
        <div class="file-preview-fallback__icon">📄</div>
        <p class="file-preview-fallback__text">Preview not available for this file type.</p>
        <a :href="file?.data" :download="file?.name" class="btn btn-primary">
          Download {{ file?.name }}
        </a>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  show: { type: Boolean, default: false },
  file: { type: Object, default: null },
})

const emit = defineEmits(['close'])

const isImage = computed(() => props.file?.type?.startsWith('image/'))
const isPdf = computed(
  () =>
    props.file?.type === 'application/pdf' ||
    props.file?.name?.toLowerCase().endsWith('.pdf'),
)

function close() {
  emit('close')
}
</script>

