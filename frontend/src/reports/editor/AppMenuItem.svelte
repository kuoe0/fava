<!--
  @component
  A single top level menu item in an app menu.

  The default slot should filled with its vertically arranged sub-items.
-->
<script lang="ts">
  import { ChevronDown } from "@lucide/svelte";
  import type { Snippet } from "svelte";

  interface Props {
    /** The name of the menu item. */
    name: string;
    children: Snippet;
  }

  let { name, children }: Props = $props();

  let open = $state(false);
</script>

<span
  class:open
  tabindex="0"
  role="menuitem"
  onblur={() => {
    open = false;
  }}
  onkeydown={(event) => {
    if (event.key === "Escape") {
      open = false;
    } else if (event.key === "ArrowDown") {
      open = true;
    }
  }}
>
  {name}
  <ChevronDown size={12} />
  <ul role="menu">
    {@render children()}
  </ul>
</span>

<style>
  span {
    display: inline-flex;
    align-items: center;
    gap: 0.25em;
    position: relative;
    padding: 0.7em 0.5em;
    cursor: pointer;
  }

  span.open,
  span:hover {
    background-color: var(--background-darkest);
  }

  ul {
    position: absolute;
    top: 100%;
    left: 0;
    z-index: var(--z-index-floating-ui);
    display: none;
    width: max-content;
    min-width: 250px;
    max-width: 90vw;
    max-height: 400px;
    margin: 0;
    overflow-y: auto;
    background-color: var(--background);
    border: 1px solid var(--border);
    box-shadow: var(--box-shadow-dropdown);
    padding: 0.5rem 0;
  }

  span.open > ul,
  span:hover > ul {
    display: block;
  }
</style>
