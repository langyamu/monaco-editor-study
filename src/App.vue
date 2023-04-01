<script setup lang="ts">
import * as monaco from "monaco-editor/esm/vs/editor/editor.api";
import editorWorker from "monaco-editor/esm/vs/editor/editor.worker?worker";
import tsWorker from "monaco-editor/esm/vs/language/typescript/ts.worker?worker";
import { nextTick, onMounted, onUnmounted, ref } from "vue";
// 注册 TypeScript 语言的语法解释器，提供语法高亮等，是 TypeScript 语言的基础能力
import "monaco-editor/esm/vs/basic-languages/typescript/typescript.contribution";
// @ts-ignore
self.MonacoEnvironment = {
  getWorker(_: string, label: string) {
    if (label === "typescript" || label === "javascript") {
      return new tsWorker();
    }
    return new editorWorker();
  },
};

const editor_dom = ref<HTMLElement>();
let editorIns: monaco.editor.IStandaloneCodeEditor;

onMounted(async () => {
  await nextTick();

  editorIns = monaco.editor.create(
    editor_dom.value as HTMLElement,
    {
      value: "function(){}",
      language: "typescript",
      theme: "vs",
      readOnly: false,
    }
  );

  // editorIns.getAction("editor.action.formatDocument")?.run()

  editorIns.onDidChangeModelContent(function (e) {
    editorIns?.layout();
  });

  onUnmounted(() => {
    //  ( editor_ins.value as monaco.editor.IStandaloneCodeEditor).dispose();  // 销毁
  });
});
</script>

<template>
  <section class="editor-container">
    <div class="editor" ref="editor_dom"></div>
  </section>
</template>

<style scoped>
.editor {
  width: 100vw;
  height: 100vh;
}
</style>
