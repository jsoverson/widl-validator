<script>
  import { onMount, getContext } from "svelte";

  import JSONTree from "svelte-json-tree";
  import TabBar from "@smui/tab-bar";
  import Tab from "@smui/tab";

  import Editor from "./Editor.svelte";
  // import Codeview, { CODEVIEW } from "./Codeview.svelte";

  import { parse } from "../widl-js/dist/cjs/index";

  import defaultSample from "./default-sample.js";
  export let name = "Widl-Validator";

  // const { updateCodeview } = getContext(CODEVIEW);

  let editorValue = defaultSample;

  let ast = {};
  let codegen = "";
  let mode;
  let activeTab = "AST";

  function onChange(evt) {
    updateAst(evt.detail.source);
    updateCodegen();
  }

  onMount(() => {
    updateAst(editorValue);
  });

  function updateCodegen() {
    // updateCodeview(editorValue, mode);
  }

  function updateAst(src) {
    try {
      const parsed = parse(src, { noLocation: true });
      // svelte-json-tree doesn't render constructors well, so we have to
      // force the ast into a POJSO until we replace or fix the component.
      ast = JSON.parse(
        JSON.stringify(parsed, (k, v) =>
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
      <div class="tabs">
        <TabBar tabs={["AST", "Codegen"]} let:tab bind:active={activeTab}>
          <Tab {tab}>
            <h3>{tab}</h3>
          </Tab>
        </TabBar>
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
          <div class="panel" class:selected={activeTab === "Codegen"}>
            [Not implemented yet]
            <!-- <Codeview /> -->
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
  .tabs {
    height: 60px;
  }
</style>
