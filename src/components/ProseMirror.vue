<template>
  <div>
    <!-- 自定义菜单栏 -->
    <div class="menu-bar">
      <button @click="insertHeading(1)(view.state, view.dispatch);">插入标题1</button>
      <button @click="insertHeading(2)(view.state, view.dispatch);">插入标题2</button>
      <button @click="insertHeading(3)(view.state, view.dispatch);">插入标题3</button>
      <button @click="toggleBold">加粗</button>
      <button @click="toggleItalic">斜体</button>
      <button @click="toggleBlockquote">引用</button>
    </div>
    <!-- 编辑器容器 -->
    <div ref="editorRef" class="editor-container"></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import { Schema, DOMParser } from 'prosemirror-model';
import { EditorState } from 'prosemirror-state';
import { EditorView } from 'prosemirror-view';
import { toggleMark, setBlockType, wrapIn, baseKeymap } from 'prosemirror-commands';
import { keymap } from "prosemirror-keymap";
// import { schema } from 'prosemirror-schema-basic';

const editorRef = ref(null);
let mySchema = null;
let view = null;

onMounted(() => {
  // 定义自定义 Schema
  mySchema = new Schema({
    nodes: {
      // 文档根节点
      doc: {
        content: "block+",
      },
      // 段落
      paragraph: {
        content: "inline*", // 内容（子节点）包含的类型
        group: "block", // 该结点所属的类型
        parseDOM: [{ tag: "p" }], // 将（初始化或粘贴等插入的）html类型文本转换为JSON
        toDOM: () => ["p", 0], // 将JSON转换为html DOM，0代表子节点插入的位置，不写则不可以插入子节点
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

  // 自定义键盘快捷键
  const keymapPlugin = keymap(baseKeymap);

  const state = EditorState.create({
    doc: mySchema.nodeFromJSON({
      type: "doc",
      content: [
        { type: "paragraph", content: [{ type: "text", text: "这是一个ProseMirror编辑器" }] },
      ]
    }),
    schema: mySchema,
    plugins: [
      keymapPlugin, // 启用自定义键盘快捷键
    ],
  });

  // 初始化编辑器
  view = new EditorView(editorRef.value, { state });
});

// 插入标题
const insertHeading = (level) => {
  return (state, dispatch) => {
    const { $from } = state.selection;
    const headingNode = state.schema.nodes.heading;

    // 创建一个新的 heading 节点
    const tr = state.tr.replaceRangeWith(
      $from.before(),
      $from.after(),
      headingNode.create({ level })
    );

    // 应用事务
    if (dispatch) dispatch(tr.scrollIntoView());
    return true;
  };
};

// 加粗
const toggleBold = () => {
  toggleMark(mySchema.marks.strong)(view.state, view.dispatch);
};
// 斜体
const toggleItalic = () => {
  toggleMark(mySchema.marks.em)(view.state, view.dispatch);
};
// 引用
const toggleBlockquote = () => {
  wrapIn(mySchema.nodes.blockquote)(view.state, view.dispatch);
};

onBeforeUnmount(() => {
  if (view) {
    view.destroy();
  }
});
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