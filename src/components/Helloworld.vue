<template>
  <div>
    <MarkdownEditor v-model="markdownContent" />
  </div>
</template>

<script setup lang="ts">
import MarkdownEditor from './MarkdownEditor.vue';
import { ref } from 'vue';
const markdownContent = ref<string>('');
</script>