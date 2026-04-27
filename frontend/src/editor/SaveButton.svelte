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
    padding: 0.5em 1em;
    color: var(--background);
    background-color: var(--link-color);
    border: none;
    border-radius: 8px;
    font-weight: 600;
    cursor: pointer;
    transition: background-color 0.2s, transform 0.1s, opacity 0.2s;
  }

  button:hover:not(:disabled) {
    background-color: var(--link-hover-color);
  }

  button:active:not(:disabled) {
    transform: scale(0.98);
  }

  button:disabled {
    opacity: 0.6;
    cursor: not-allowed;
    background-color: var(--button-background);
    color: var(--text-color-lighter);
  }
</style>
