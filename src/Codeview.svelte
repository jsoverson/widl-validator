<script>
  import { onMount } from "svelte";
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

  function updateCodemirror() {
    if (cm) cm.getDoc().setValue(value);
  }

  $: updateCodemirror(value);

  onMount(() => {
    const textarea = document.getElementById("codeview");
    textarea.value = value;

    cm = CodeMirror.fromTextArea(textarea, {
      lineNumbers: false,
      readonly: true,
      mode: mode,
    });

    updateCodemirror(value);

    cm.setSize("auto", "100%");
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
