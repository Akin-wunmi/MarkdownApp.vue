<template>
  <div class="markdown-editor">
    <div class="toolbar" aria-label="Markdown toolbar">
      <button type="button" @click="wrapWith('**', '**')"><strong>B</strong></button>
      <button type="button" @click="wrapWith('_', '_')"><em>I</em></button>
      <button type="button" @click="insertSnippet('# ', '')">H1</button>
      <button type="button" @click="insertSnippet('- ', '')">List</button>
      <button type="button" @click="insertSnippet('```\\n', '\\n```')">Code</button>
      <button type="button" @click="togglePreview">{{ showPreview ? 'Hide' : 'Show' }} Preview</button>
    </div>

    <div class="editor-preview" :class="{ twoPane: twoPane }">
      <textarea
        v-model="markdown"
        class="editor"
        rows="15"
        placeholder="Write Markdown here..."
      ></textarea>

      <div v-if="showPreview" class="preview" v-html="htmlContent"></div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import MarkdownIt from 'markdown-it';

const props = defineProps<{
  modelValue?: string;
  twoPane?: boolean;
  showPreview?: boolean;
}>();

const emit = defineEmits<{ (e: 'update:modelValue', value: string): void }>();

const markdown = ref<string>(props.modelValue ?? '');
const md = new MarkdownIt({ breaks: true, html: false });

watch(markdown, (val) => {
  emit('update:modelValue', val);
});

const htmlContent = computed(() => md.render(markdown.value));

const showPreview = ref<boolean>(props.showPreview ?? true);
const twoPane = ref<boolean>(props.twoPane ?? true);

function wrapWith(prefix: string, suffix: string) {
  const input = document.querySelector<HTMLTextAreaElement>('textarea.editor');
  if (!input) return;
  const start = input.selectionStart ?? 0;
  const end = input.selectionEnd ?? 0;
  const before = markdown.value.substring(0, start);
  const selected = markdown.value.substring(start, end);
  const after = markdown.value.substring(end);
  const replacement = prefix + (selected || 'text') + suffix;
  markdown.value = before + replacement + after;
  requestAnimationFrame(() => {
    const pos = start + replacement.length;
    input.focus();
    input.setSelectionRange(pos, pos);
  });
}

function insertSnippet(prefix: string, suffix: string) {
  const input = document.querySelector<HTMLTextAreaElement>('textarea.editor');
  if (!input) return;
  const start = input.selectionStart ?? 0;
  const end = input.selectionEnd ?? 0;
  const before = markdown.value.substring(0, start);
  const selected = markdown.value.substring(start, end) || '';
  const after = markdown.value.substring(end);
  const insertion = prefix + selected + suffix;
  markdown.value = before + insertion + after;
  requestAnimationFrame(() => {
    const pos = before.length + insertion.length;
    input.focus();
    input.setSelectionRange(pos, pos);
  });
}

function togglePreview() {
  showPreview.value = !showPreview.value;
}
</script>

<style scoped>
.markdown-editor {
  background: #2a2a2a;
  border: 1px solid #555;
  border-radius: 8px;
  padding: 12px;
  max-width: 800px;
  margin: 40px auto;
  color: #eee;
}
.toolbar {
  display: flex;
  gap: 8px;
  margin-bottom: 8px;
}
.toolbar button {
  background: #333;
  color: #fff;
  border: 1px solid #555;
  border-radius: 6px;
  padding: 6px 10px;
  cursor: pointer;
}
.editor {
  width: 100%;
  height: 260px;
  resize: vertical;
  background: #1e1e1e;
  color: #fff;
  border: 1px solid #555;
  padding: 8px;
  border-radius: 6px;
  outline: none;
}
.preview {
  border: 1px solid #555;
  background: #fff;
  color: #111;
  padding: 12px;
  min-height: 120px;
  border-radius: 6px;
}
.editor-preview.twoPane {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}
</style>