<script lang="ts">
  import type { Component, Snippet } from "svelte";

  import { urlFor } from "../helpers.ts";
  import type { KeySpec } from "../keyboard-shortcuts.ts";
  import { keyboardShortcut } from "../keyboard-shortcuts.ts";
  import { pathname } from "../stores/url.ts";

  interface Props {
    /** Report to generate the URL for. */
    report: string;
    /** Name to display for this link. */
    name: string;
    /** Icon component to display. */
    icon?: Component;
    /** Key combination for the link. */
    key?: KeySpec;
    /** Whether this is a remote link (for which we do not intercept clicks). */
    remote?: true;
    /** Show a bubble with a number */
    bubble?: [number, "error" | "info"];
    actions?: Snippet;
    children?: Snippet;
  }

  let { report, name, icon, key, remote, bubble, actions, children }: Props = $props();

  let href = $derived(remote ? report : $urlFor(`${report}/`));
  let selected = $derived(remote ? false : href.includes($pathname));
  // eslint-disable-next-line @typescript-eslint/naming-convention
  let Icon = $derived(icon);

  let has_right_content = $derived((bubble && bubble[0] > 0) || actions);
</script>

<li style="display: flex; flex-direction: column; width: 100%;">
  <div class="link-row" style="display: flex; align-items: center; width: 100%;">
    <a class:selected {href} {@attach keyboardShortcut(key)} data-remote={remote} style="flex: 1;">
      {#if Icon}
        <Icon size={16} />
      {/if}
      <span>{name}</span>
    </a>
    {#if has_right_content}
      <div class="right-actions" style="margin-left: auto; min-width: 36px; display: inline-flex; justify-content: center; align-items: center;">
        {#if bubble && bubble[0] > 0}
          <span class="bubble" class:error={bubble[1] === "error"}>
            {bubble[0]}
          </span>
        {/if}
        {@render actions?.()}
      </div>
    {/if}
  </div>
  <div class="submenu-area" style="width: 100%;">
    {@render children?.()}
  </div>
</li>

<style>
  a {
    display: flex;
    align-items: center;
    gap: 0.5em;
    padding: 0.25em 0.5em 0.25em 1em;
    color: inherit;
    text-decoration: none;
    white-space: nowrap;
  }

  a.selected,
  a:hover {
    color: var(--sidebar-hover-color);
    background-color: var(--sidebar-border);
  }

  li {
    display: flex;
    flex-direction: column;
    width: 100%;
  }

  .link-row {
    display: flex;
    align-items: center;
    width: 100%;
  }

  li:last-child {
    margin-bottom: 0;
    border: none;
  }

  li a:first-child {
    flex: 1;
  }

  .bubble {
    width: 24px;
    height: 24px;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    font-size: 0.8em;
    font-weight: bold;
    color: var(--sidebar-color);
    background-color: var(--sidebar-border);
    border-radius: 50%;
  }

  .error.bubble {
    color: white;
    background-color: var(--error);
  }
</style>
