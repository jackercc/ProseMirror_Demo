<template>
  <div>
    <div class="menu-bar">
      <button @click="addHeading">标题</button>
      <button @click="addParagraph">段落</button>
      <button @click="addOrderedList">有序列表</button>
      <button @click="addBulletList">无序列表</button>
      <button @click="addBlockquote">引用</button>
      <button @click="addCodeBlock">代码块</button>
    </div>
    <EditorContent :editor="editor" class="editor-container" />
  </div>
</template>

<script setup>
import { EditorContent, useEditor } from '@tiptap/vue-3';
import StarterKit from '@tiptap/starter-kit';

    const editor = useEditor({
      extensions: [
        StarterKit,
      ],
      content: '<p>这是一个 Tiptap 编辑器</p>',
    });

    const addHeading = () => {
      editor.value.chain().focus().toggleHeading({ level: 1 }).run();
    };

    const addParagraph = () => {
      editor.value.chain().focus().setParagraph().run();
    };

    const addOrderedList = () => {
      editor.value.chain().focus().toggleOrderedList().run();
    };

    const addBulletList = () => {
      editor.value.chain().focus().toggleBulletList().run();
    };

    const addBlockquote = () => {
      editor.value.chain().focus().toggleBlockquote().run();
    };

    const addCodeBlock = () => {
      editor.value.chain().focus().toggleCodeBlock().run();
    };
</script>

<style scoped>
.menu-bar {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}
.editor-container {
  border: 1px solid #ccc;
  padding: 10px;
  min-height: 200px;
}
.editor-container :deep(.ProseMirror) {
  outline: 0;
}
</style>
