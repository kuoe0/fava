<script lang="ts">
  import { Play } from "@lucide/svelte";

  import { attach_editor } from "../../codemirror/dom.ts";
  import type { CodemirrorBql } from "../../codemirror/types.ts";
  import { _ } from "../../i18n.ts";
  import { keyboardShortcut } from "../../keyboard-shortcuts.ts";

  interface Props {
    value: string;
    submit: () => void;
    codemirror_bql: CodemirrorBql;
  }

  let { value = $bindable(), submit, codemirror_bql }: Props = $props();

  // svelte-ignore state_referenced_locally
  const editor = codemirror_bql.init_query_editor(
    value,
    (state) => {
      value = state.sliceDoc();
    },
    _("…enter a BQL query. 'help' to list available commands."),
    () => submit,
  );

  $effect(() => {
    if (value !== editor.state.sliceDoc()) {
      editor.dispatch(codemirror_bql.replace_contents(editor.state, value));
    }
  });
</script>

<form
  onsubmit={(event) => {
    event.preventDefault();
    submit();
  }}
>
  <div {@attach attach_editor(editor)}></div>
  <button type="submit" {@attach keyboardShortcut("Control+Enter")} aria-label={_("Submit")}>
    <Play size={16} />
    <span>{_("Submit")}</span>
  </button>
</form>

<style>
  form {
    display: flex;
    align-items: center;
    padding-bottom: 1em;
  }

  button {
    display: inline-flex;
    align-items: center;
    gap: 0.5em;
    height: 32px;
    padding: 0 1em;
    margin: 0;
    background-color: var(--link-color);
    color: var(--background);
    border: none;
    border-radius: 8px;
    cursor: pointer;
    transition: background-color 0.2s;
  }

  button:hover {
    background-color: var(--link-hover-color);
  }

  div {
    flex-grow: 1;
    width: 100%;
    height: auto;
    margin-right: 0.5em;
    font-size: 16px;
    border: 1px solid var(--border);
  }

  form :global(.cm-editor) {
    width: 100%;
  }
</style>
