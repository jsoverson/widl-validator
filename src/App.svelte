<script>
  import { onMount } from "svelte";

  import JSONTree from "svelte-json-tree";
  import TabBar from "@smui/tab-bar";
  import Tab from "@smui/tab";
  import Select, { Option } from "@smui/select";

  import Editor from "./Editor.svelte";
  import Codeview from "./Codeview.svelte";

  // import { parse, Writer, Context } from "../widl-js/dist/cjs/index";

  import {
    ModuleVisitor as RustModuleVisitor,
    ScaffoldVisitor as RustScaffoldVisitor,
  } from "widl-codegen/language/rust/index.js";
  import { parse, Writer, Context } from "widl-codegen/widl/index.js";

  import defaultSample from "./default-sample.js";
  export let name = "Widl-Validator";

  let editorValue = defaultSample;
  let codegenValue = "";

  let parsedWidlDoc = null;
  let ast = {};
  let codegen = "";
  let mode = "rust";
  let activeTab = "Codegen";
  let codeview;

  function onChange(evt) {
    editorValue = evt.detail.source;
    updateAst(editorValue);
    updateCodegen(editorValue);
  }

  onMount(() => {
    updateAst(editorValue);
    updateCodegen(editorValue);
  });

  function updateCodegen(editorValue) {
    const writer = new Writer();
    const context = new Context({
      import: "github.com/wapc/languages-tests/tinygo/module",
      module: "module",
    });
    const visitor = new RustModuleVisitor(writer);
    parsedWidlDoc.accept(context, visitor);
    let source = writer.string();
    codegenValue = source;
  }

  function updateAst(src) {
    try {
      parsedWidlDoc = parse(src, { noLocation: true });
      // svelte-json-tree doesn't render constructors well, so we have to
      // force the ast into a POJSO until we replace or fix the component.
      ast = JSON.parse(
        JSON.stringify(parsedWidlDoc, (k, v) =>
          Array.isArray(v) && v.length == 0 ? undefined : v
        )
      );
    } catch (e) {
      ast = { error: "Error parsing widl", message: e.message };
    }
  }
</script>

<div class="app">
  <header>
    <h1>
      <a href="https://github.com/jsoverson/widl-validator">WIDL Validator</a>
    </h1>
  </header>
  <main class="content">
    <div class="left-panel">
      <h3 style="text-align:center">WIDL</h3>
      <div class="panel-content">
        <Editor on:change={onChange} bind:value={editorValue} />
      </div>
    </div>
    <div class="right-panel">
      <div class="invalid-overlay" class:visible={ast.error}>
        <div class="bg" />
        <div class="error">
          <h3>Error: {ast.error}</h3>
          <h4>Error: {ast.message}</h4>
        </div>
      </div>
      <div class="panel-header">
        <div class="tabs">
          <TabBar tabs={["Codegen", "AST"]} let:tab bind:active={activeTab}>
            <Tab {tab}>
              <h3>{tab}</h3>
            </Tab>
          </TabBar>
        </div>
      </div>

      <div class="panel-content">
        <div class="tab-panels">
          <div class="panel" class:selected={activeTab === "AST"}>
            <div
              style="--json-tree-font-size: 13px;--json-tree-font-family:  monospace"
            >
              <JSONTree value={ast} />
            </div>
          </div>
          <div
            class="panel full-width"
            class:selected={activeTab === "Codegen"}
          >
            <Select
              variant="filled"
              value="rust"
              menu$class="codegen-select"
              anchor$class="codegen-select"
            >
              <Option value="rust">Rust</Option>
              <!-- <Option value="go">Go</Option>
              <Option value="javascript">AsmScript(TypeScript)</Option> -->
            </Select>
            <Codeview bind:this={codeview} value={codegenValue} {mode} />
          </div>
        </div>
      </div>
    </div>
  </main>
</div>

<style>
  h1 {
    color: black;
  }
  .app {
    height: 100%;
  }
  header {
    height: 3em;
    line-height: 2em;
    text-align: center;
    font-family: monospace;
  }
  .content {
    height: calc(100vh - 100px);
    display: grid;
    grid-template-columns: 50% 50%;
  }
  .panel {
    display: none;
  }
  .selected {
    display: block !important;
  }
  .invalid {
    background-color: rgb(255, 158, 158);
  }
  .panel-content {
    height: 100%;
  }
  .right-panel .panel-header {
    height: 60px;
    position: relative;
  }
  .invalid-overlay {
    z-index: 100000;
    position: absolute;
    width: 50%;
    display: none;
    height: 100%;
  }
  .invalid-overlay .bg {
    background-color: red;
    opacity: 50%;
    position: absolute;
    width: 100%;
    height: 100%;
    z-index: 9;
  }
  .visible {
    display: block;
  }
  .error {
    position: absolute;
    z-index: 10;
    background-color: white;
    text-align: center;
    margin-top: 5em;
    width: 100%;
  }

  * :global(.codegen-select) {
    width: 100%;
  }
</style>
