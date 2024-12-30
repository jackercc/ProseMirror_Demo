<template>
  <div>
    <!-- 自定义菜单栏 -->
    <div class="menu-bar">
      <button @click="toggleBold">加粗</button>
      <button @click="toggleItalic">斜体</button>
      <button @click="toggleOrderedList">有序列表</button>
      <button @click="toggleBulletList">无序列表</button>
      <button @click="toggleBlockquote">引用</button>
      <button @click="toggleCodeBlock">代码块</button>
    </div>
    <!-- 编辑器容器 -->
    <div ref="editor" class="editor-container"></div>
  </div>
</template>

<script>
import { Schema, DOMParser } from 'prosemirror-model';
import { EditorState } from 'prosemirror-state';
import { EditorView } from 'prosemirror-view';
import { addListNodes, wrapInList } from 'prosemirror-schema-list';
import { toggleMark, setBlockType, wrapIn } from 'prosemirror-commands';
// import { schema } from 'prosemirror-schema-basic';

export default {
  data() {
    return {
      view: null,
      mySchema: null,
    };
  },
  mounted() {
    // 定义自定义 Schema
    this.mySchema = new Schema({
      nodes: {
        // 文档根节点
        doc: {
          content: "block+",
        },
        // 段落
        paragraph: {
          content: "inline*",
          group: "block",
          parseDOM: [{ tag: "p" }],
          toDOM: () => ["p", 0],
        },
        // 标题
        heading: {
          attrs: { level: { default: 1 } },
          content: "inline*",
          group: "block",
          defining: true,
          parseDOM: [
            { tag: "h1", attrs: { level: 1 } },
            { tag: "h2", attrs: { level: 2 } },
            { tag: "h3", attrs: { level: 3 } },
          ],
          toDOM: (node) => [`h${node.attrs.level}`, 0],
        },
        // 引用
        blockquote: {
          content: "block+",
          group: "block",
          defining: true,
          parseDOM: [{ tag: "blockquote" }],
          toDOM: () => ["blockquote", 0],
        },
        // 代码块
        code_block: {
          content: "text*",
          group: "block",
          marks: "",
          code: true,
          defining: true,
          parseDOM: [{ tag: "pre", preserveWhitespace: "full" }],
          toDOM: () => ["pre", ["code", 0]],
        },
        // 文本节点
        text: {
          group: "inline",
        },
      },
      marks: {
        // 加粗
        strong: {
          parseDOM: [
            { tag: "strong" },
            { tag: "b", getAttrs: (node) => node.style.fontWeight !== "normal" && null },
            {
              style: "font-weight",
              getAttrs: (value) => /^(bold(er)?|[5-9]\d{2,})$/.test(value) && null,
            },
          ],
          toDOM: () => ["strong", 0],
        },
        // 斜体
        em: {
          parseDOM: [
            { tag: "i" },
            { tag: "em" },
            {
              style: "font-style",
              getAttrs: (value) => value === "italic" && null,
            },
          ],
          toDOM: () => ["em", 0],
        },
      },
    });

    // 初始化 EditorView
    this.view = new EditorView(this.$refs.editor, {
      state: EditorState.create({
        doc: DOMParser.fromSchema(this.mySchema).parse(document.createElement('div')),
        plugins: [],
      }),
    });
  },
  methods: {
    // 加粗
    toggleBold() {
      toggleMark(this.mySchema.marks.strong)(this.view.state, this.view.dispatch);
    },
    // 斜体
    toggleItalic() {
      toggleMark(this.mySchema.marks.em)(this.view.state, this.view.dispatch);
    },
    // 有序列表
    toggleOrderedList() {
      wrapInList(this.mySchema.nodes.ordered_list)(this.view.state, this.view.dispatch);
    },
    // 无序列表
    toggleBulletList() {
      wrapInList(this.mySchema.nodes.bullet_list)(this.view.state, this.view.dispatch);
    },
    // 引用
    toggleBlockquote() {
      wrapIn(this.mySchema.nodes.blockquote)(this.view.state, this.view.dispatch);
    },
    // 代码块
    toggleCodeBlock() {
      setBlockType(this.mySchema.nodes.code_block)(this.view.state, this.view.dispatch);
    },
  },
  beforeUnmount() {
    if (this.view) {
      this.view.destroy();
    }
  },
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
</style>
