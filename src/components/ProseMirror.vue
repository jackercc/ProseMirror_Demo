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
import { schema } from 'prosemirror-schema-basic';
import { addListNodes, wrapInList } from 'prosemirror-schema-list';
import { toggleMark, setBlockType, wrapIn } from 'prosemirror-commands';

export default {
  data() {
    return {
      view: null,
      mySchema: null,
    };
  },
  mounted() {
    // 扩展 Schema，支持列表和其他块级内容
    this.mySchema = new Schema({
      nodes: addListNodes(schema.spec.nodes, 'paragraph block*', 'block'),
      marks: schema.spec.marks,
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
