<script context="module">
  export const CODEVIEW = {};
</script>

<script>
  import { onMount, setContext } from "svelte";
  import CodeMirror from "codemirror";
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  import "codemirror/mode/rust/rust";
  import "codemirror/mode/go/go";
  import "codemirror/mode/javascript/javascript";
  import "../node_modules/codemirror/lib/codemirror.css";

  let cm = null;
  export let value;
  export let mode;

  onMount(() => {
    const textarea = document.getElementById("codeview");
    textarea.value = value;

    window.cm = cm = CodeMirror.fromTextArea(textarea, {
      lineNumbers: true,
      readonly: true,
      mode: mode,
    });
    cm.getWrapperElement();

    cm.setSize("auto", "100%");
  });
  setContext(CODEVIEW, {
    updateCodeview(src, mode) {
      cm.setOption("mode", mode);
      cm.getDoc().setValue(src);
    },
  });
</script>

<div class="code">
  <div class="editor">
    <textarea id="codeview" {value} />
  </div>
</div>

<style>
  .editor {
    height: 100%;
  }

  .code {
    height: calc(100% - 10px);
  }
</style>
