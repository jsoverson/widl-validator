<script>
  import { onMount } from "svelte";

  import JSONTree from "svelte-json-tree";
  import TabBar from "@smui/tab-bar";
  import Tab from "@smui/tab";
  import Select, { Option } from "@smui/select";

  import Editor from "./Editor.svelte";
  import Codeview from "./Codeview.svelte";

  import {
    ModuleVisitor as RustModuleVisitor,
    ScaffoldVisitor as RustScaffoldVisitor,
  } from "@wapc/widl-codegen/rust";
  import {
    ModuleVisitor as TinyGoModuleVisitor,
    ScaffoldVisitor as TinyGoScaffoldVisitor,
  } from "@wapc/widl-codegen/tinygo";
  import {
    ModuleVisitor as AssemblyScriptModuleVisitor,
    ScaffoldVisitor as AssemblyScriptScaffoldVisitor,
  } from "@wapc/widl-codegen/assemblyscript";

  import { parse, ast as AST } from "@wapc/widl";

  const { Writer, Context } = AST;

  import defaultSample from "./default-sample.js";
  export let name = "Widl-Validator";

  let hashValue = location.hash ? atob(location.hash.slice(1)) : "";

  let editorValue = hashValue || defaultSample;
  let codegenValue = "";
  let selectEl;

  let parsedWidlDoc = null;
  let ast = {};
  let mode = "rust";
  let activeTab = "Codegen";
  let codeview;

  function onChange(evt) {
    update(evt.detail.source);
  }

  function onLangChange(evt) {
    update(editorValue);
  }

  onMount(() => {
    update(editorValue);
    selectEl.$on("MDCSelect:change", () => {
      update(editorValue);
    });
  });

  function update(src) {
    try {
      parsedWidlDoc = parse(src, undefined, { noLocation: true });
      // svelte-json-tree doesn't render constructors well, so we have to
      // force the ast into a POJSO until we replace or fix the component.
      ast = parseAst(parsedWidlDoc);
      location.hash = btoa(src);
    } catch (e) {
      ast = { error: "Error parsing widl", message: e.message };
    }

    codegenValue = updateCodegen(parsedWidlDoc);
  }

  function updateCodegen(parsedWidlDoc) {
    const writer = new Writer();
    const context = new Context({
      import: "github.com/wapc/languages-tests/tinygo/module",
      module: "module",
    });

    const visitor = ((mode) => {
      switch (mode) {
        case "rust":
          return new RustModuleVisitor(writer);
        case "tinygo":
          return new TinyGoModuleVisitor(writer);
        case "assemblyscript":
          return new AssemblyScriptModuleVisitor(writer);
      }
    })(mode);
    parsedWidlDoc.accept(context, visitor);
    return writer.string();
  }

  function parseAst(doc) {
    try {
      // svelte-json-tree doesn't render constructors well, so we have to
      // force the ast into a POJSO until we replace or fix the component.
      return JSON.parse(
        JSON.stringify(doc, (k, v) =>
          Array.isArray(v) && v.length == 0 ? undefined : v
        )
      );
    } catch (e) {
      return { error: "Error parsing widl", message: e.message };
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
              bind:this={selectEl}
              variant="filled"
              bind:value={mode}
              menu$class="codegen-select"
              anchor$class="codegen-select"
              on:change={onLangChange}
            >
              <Option value="rust">Rust</Option>
              <Option value="tinygo">TinyGo</Option>
              <Option value="assemblyscript">AssemblyScript</Option>
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
    margin-top: 0;
  }
  .app {
    height: 100%;
    max-height: 100%;
    /* overflow: hidden; */
  }
  header {
    height: 3em;
    line-height: 2em;
    text-align: center;
    font-family: monospace;
  }
  .content {
    height: calc(100% - 3em);
    display: grid;
    grid-template-columns: 50% 50%;
  }
  .panel {
    display: none;
  }
  .selected {
    display: block !important;
  }
  .panel-content {
    height: 100%;
    overflow: auto;
  }
  .left-panel,
  .right-panel {
    height: 100%;
    overflow: hidden;
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

  * :global(.mdc-select) {
    width: 100%;
  }
</style>
