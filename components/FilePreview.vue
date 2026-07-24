<template>
  <div v-if="show" @click.self="close" class="modal-backdrop file-preview-backdrop">
    <div class="modal file-preview-modal">
      <div class="modal-header file-preview-header">
        <div class="modal-header__title">{{ file?.name }}</div>
        <button type="button" @click="close" aria-label="Close" class="modal-close">✕</button>
      </div>

      <div class="modal-body file-preview-body">
        <img
          v-if="isImage"
          :src="file?.data"
          :alt="file?.name" class="file-preview-content file-preview-content--image" />
        <object
          v-else-if="isPdf"
          :data="file?.data"
          type="application/pdf"
         
         class="file-preview-content file-preview-content--pdf"></object>
        <div v-else class="file-preview-fallback">
          <div class="file-preview-fallback__icon">📄</div>
          <p class="file-preview-fallback__text">Preview not available for this file type.</p>
          <a :href="file?.data" :download="file?.name" class="btn btn-primary">
            Download {{ file?.name }}
          </a>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>

import { computed } from 'vue'

