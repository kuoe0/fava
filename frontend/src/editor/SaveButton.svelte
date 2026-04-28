<script lang="ts">
  import { Save } from "@lucide/svelte";

  import { _ } from "../i18n.ts";
  import { keyboardShortcut } from "../keyboard-shortcuts.ts";

  interface Props {
    /** Whether anything is changed - the button is disabled otherwise. */
    changed: boolean;
    /** Whether the contents are currently being saved. */
    saving: boolean;
  }

  let { changed, saving }: Props = $props();
</script>

<button
  type="submit"
  disabled={!changed}
  {@attach keyboardShortcut({ key: "Control+s", mac: "Meta+s" })}
  aria-label={_("Save")}
>
  {#if saving}
    <span>{_("Saving…")}</span>
  {:else}
    <Save size={16} />
    <span>{_("Save")}</span>
  {/if}
</button>

<style>
  button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 0.5em;
    padding: 0.4em 1em;
    color: var(--text-color);
    background-color: var(--sidebar-border, #e0e0e0);
    border: none;
    border-radius: 0; /* Traditional square style */
    font-weight: normal;
    cursor: pointer;
    transition: background-color 0.2s;
  }

  button:hover:not(:disabled) {
    background-color: var(--table-header-background, #d0d0d0);
  }

  button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
</style>
