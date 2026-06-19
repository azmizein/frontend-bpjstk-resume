<script setup>
import { onBeforeUnmount, watch } from 'vue'
import { useEditor, EditorContent } from '@tiptap/vue-3'
import StarterKit from '@tiptap/starter-kit'
import Underline from '@tiptap/extension-underline'
import Link from '@tiptap/extension-link'
import Placeholder from '@tiptap/extension-placeholder'

const props = defineProps({
  modelValue: {
    type: String,
    default: ''
  },
  placeholder: {
    type: String,
    default: 'Write something...'
  }
})

const emit = defineEmits(['update:modelValue'])

const setLink = () => {
  if (!editor.value) return
  const previousUrl = editor.value.getAttributes('link').href
  const url = window.prompt('URL', previousUrl)
  
  if (url === null) return
  
  if (url === '') {
    editor.value.chain().focus().extendMarkRange('link').unsetLink().run()
    return
  }
  
  editor.value.chain().focus().extendMarkRange('link').setLink({ href: url }).run()
}

const editor = useEditor({
  content: props.modelValue,
  extensions: [
    StarterKit,
    Underline,
    Link.configure({
      openOnClick: false,
    }),
    Placeholder.configure({
      placeholder: props.placeholder,
    })
  ],
  onUpdate: () => {
    emit('update:modelValue', editor.value.getHTML())
  },
})

watch(() => props.modelValue, (value) => {
  const isSame = editor.value.getHTML() === value
  if (!isSame) {
    editor.value.commands.setContent(value, false)
  }
})

onBeforeUnmount(() => {
  if (editor.value) {
    editor.value.destroy()
  }
})
</script>

<template>
  <div class="rich-text-editor">
    <div class="rte-toolbar">
      <div class="rte-tools">
        <button class="tool-btn font-bold" :class="{ 'is-active': editor?.isActive('bold') }" @click.prevent="editor?.chain().focus().toggleBold().run()">B</button>
        <button class="tool-btn font-italic" :class="{ 'is-active': editor?.isActive('italic') }" @click.prevent="editor?.chain().focus().toggleItalic().run()">I</button>
        <button class="tool-btn font-underline" :class="{ 'is-active': editor?.isActive('underline') }" @click.prevent="editor?.chain().focus().toggleUnderline().run()">U</button>
        <button class="tool-btn font-strike" :class="{ 'is-active': editor?.isActive('strike') }" @click.prevent="editor?.chain().focus().toggleStrike().run()">S</button>
        <div class="divider"></div>
        <button class="tool-btn" :class="{ 'is-active': editor?.isActive('bulletList') }" @click.prevent="editor?.chain().focus().toggleBulletList().run()"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="8" y1="6" x2="21" y2="6"></line><line x1="8" y1="12" x2="21" y2="12"></line><line x1="8" y1="18" x2="21" y2="18"></line><line x1="3" y1="6" x2="3.01" y2="6"></line><line x1="3" y1="12" x2="3.01" y2="12"></line><line x1="3" y1="18" x2="3.01" y2="18"></line></svg></button>
        <button class="tool-btn" :class="{ 'is-active': editor?.isActive('orderedList') }" @click.prevent="editor?.chain().focus().toggleOrderedList().run()"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="10" y1="6" x2="21" y2="6"></line><line x1="10" y1="12" x2="21" y2="12"></line><line x1="10" y1="18" x2="21" y2="18"></line><path d="M4 6h1v4"></path><path d="M4 10h2"></path><path d="M6 18H4c0-1 2-2 2-3s-1-1.5-2-1"></path></svg></button>
        <div class="divider"></div>
        <button class="tool-btn" :class="{ 'is-active': editor?.isActive('link') }" @click.prevent="setLink"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg></button>
      </div>
      <button class="pre-written-btn">Pre-written phrases <span class="plus-circle">+</span></button>
    </div>
    <editor-content :editor="editor" />
  </div>
</template>

<style scoped>
/* Rich Text Editor */
.rich-text-editor {
  background-color: var(--input-bg, #f7f9fa);
  border-radius: var(--radius-md, 8px);
  overflow: hidden;
  border: 2px solid transparent;
  transition: background-color 0.2s, border-color 0.2s;
}

.rich-text-editor:focus-within {
  background-color: #f7f9fa;
  border-bottom: 2px solid var(--primary-color, #1a73e8);
}

.rte-toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  background-color: transparent;
}

.rte-tools {
  display: flex;
  align-items: center;
  gap: 4px;
}

.tool-btn {
  background: none;
  border: none;
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  cursor: pointer;
  color: #4a5568;
  font-family: serif;
  font-size: 16px;
  transition: all 0.2s;
}

.tool-btn:hover {
  background-color: rgba(0,0,0,0.05);
  color: #000;
}

.tool-btn.is-active {
  background-color: #e2e8f0;
  color: #000;
}

.font-bold { font-weight: bold; }
.font-italic { font-style: italic; }
.font-underline { text-decoration: underline; }
.font-strike { text-decoration: line-through; }

.divider {
  width: 1px;
  height: 16px;
  background-color: #cbd5e1;
  margin: 0 8px;
}

.pre-written-btn {
  background: none;
  border: none;
  color: var(--text-secondary, #4a5568);
  font-size: 14px;
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  transition: color 0.2s;
}

.pre-written-btn:hover {
  color: var(--primary-color, #1a73e8);
}

.plus-circle {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 18px;
  height: 18px;
  background-color: var(--primary-color, #1a73e8);
  color: white;
  border-radius: 50%;
  font-size: 14px;
  font-weight: bold;
  line-height: 1;
}

.rich-text-editor :deep(.tiptap) {
  width: 100%;
  min-height: 150px;
  padding: 0 16px 16px;
  border: none;
  background: transparent;
  font-family: inherit;
  font-size: 15px;
  color: var(--text-primary, #1a202c);
  line-height: 1.6;
  outline: none;
}

.rich-text-editor :deep(.tiptap p.is-editor-empty:first-child::before) {
  color: #a0aab8;
  content: attr(data-placeholder);
  float: left;
  height: 0;
  pointer-events: none;
}
</style>
