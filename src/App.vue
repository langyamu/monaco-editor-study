<script setup lang="ts">
import * as monaco from "monaco-editor";
import editorWorker from "monaco-editor/esm/vs/editor/editor.worker?worker";
import JsonWorker from 'monaco-editor/esm/vs/language/json/json.worker?worker';
import tsWorker from "monaco-editor/esm/vs/language/typescript/ts.worker?worker";
import { nextTick, onMounted, ref } from "vue";
// 注册 TypeScript 语言的语法解释器，提供语法高亮等，是 TypeScript 语言的基础能力
import "monaco-editor/esm/vs/basic-languages/typescript/typescript.contribution";
import "monaco-editor/esm/vs/language/json/monaco.contribution";
import prettier from 'prettier';
import parserTS from 'prettier/parser-typescript';
// @ts-ignore
self.MonacoEnvironment = {
  getWorker(_: string, label: string) {
    if (label === "json") {
      return new JsonWorker();
    }
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
  console.log(monaco)
  // validation settings
  monaco.languages.typescript.typescriptDefaults.setDiagnosticsOptions({
    noSemanticValidation: true,
    noSyntaxValidation: false
  });

  // compiler options
  monaco.languages.typescript.typescriptDefaults.setCompilerOptions({
    target: monaco.languages.typescript.ScriptTarget.ES2015,
    allowNonTsExtensions: true,
    allowJs: true
  });
  //  导入 mongodb 类型
  //  注意 typescript 版本与   monaco.languages.typescript.typescriptVersion 对应
  const mongodb_type_text = await (await fetch("./mongodb.d.ts")).text()
  const lib_source = "ts:filename/mongodb.d.ts"
  monaco.languages.typescript.typescriptDefaults.addExtraLib(mongodb_type_text, lib_source)
  monaco.editor.createModel(mongodb_type_text, 'typescript', monaco.Uri.parse(lib_source))

  // 格式化逻辑
  function spliceSemiAndDoubleQuote(val: string) {
    return prettier.format(val, {
      parser: 'typescript',
      "arrowParens": "always",
      "bracketSameLine": false,
      "bracketSpacing": true,
      "embeddedLanguageFormatting": "auto",
      "htmlWhitespaceSensitivity": "css",
      "insertPragma": false,
      "jsxSingleQuote": false,
      "printWidth": 80,
      "proseWrap": "preserve",
      "quoteProps": "as-needed",
      "requirePragma": false,
      "semi": true,
      "singleQuote": false,
      "tabWidth": 4,
      "trailingComma": "all",
      "useTabs": false,
      "vueIndentScriptAndStyle": false,
      plugins: [parserTS],
    })
  }

  monaco.languages.registerDocumentFormattingEditProvider('typescript', {
    provideDocumentFormattingEdits(model, options, token) {
      return [{
        text: spliceSemiAndDoubleQuote(model.getValue()),
        range: model.getFullModelRange()
      }]
    }
  })


  editorIns = monaco.editor.create(
    editor_dom.value as HTMLElement,
    {
      value: "db('@huli-patient').collection('patient').find({}).sort({}).limit(\"${limit}\").skip(\"${skip}\").project({_id:0})",
      language: "typescript",
      theme: "vs-dark",
      readOnly: false,
      scrollBeyondLastLine: false, // 为 false 时 格式化后不会滚动条顶部不会跳转到行末
      minimap: {
        enabled: false // 不要小地图
      },
    }
  );

  // editorIns.getAction("editor.action.formatDocument")?.run()

  editorIns.onDidChangeModelContent(function (e) {
    editorIns?.layout();
  });

  // onUnmounted(() => {
  //   editorIns.dispose();  // 销毁编辑器
  // });
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
