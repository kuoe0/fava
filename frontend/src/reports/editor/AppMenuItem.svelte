<script lang="ts">
  import { ChevronDown } from "@lucide/svelte";
  import type { Snippet } from "svelte";
  import { onMount } from "svelte";

  interface Props {
    /** The name of the menu item. */
    name: string;
    children: Snippet;
    open?: boolean;
  }

  let { name, children, open = $bindable(false) }: Props = $props();

  let spanElement: HTMLElement;

  onMount(() => {
    const handleClickOutside = (event: MouseEvent) => {
      if (open && !spanElement.contains(event.target as Node)) {
        open = false;
      }
    };
    document.addEventListener("click", handleClickOutside);
    return () => {
      document.removeEventListener("click", handleClickOutside);
    };
  });
</script>

<span
  bind:this={spanElement}
  class:open
  tabindex="0"
  role="menuitem"
  onclick={() => {
    open = !open;
  }}
  onblur={(e) => {
    if (!e.currentTarget.contains(e.relatedTarget as Node)) {
      open = false;
    }
  }}
  onkeydown={(event) => {
    if (event.key === "Escape") {
      open = false;
    } else if (event.key === "ArrowDown") {
      open = true;
    }
  }}
  style="display: inline-flex; align-items: center; gap: 0.25em; position: relative;"
>
  {name}
  <ChevronDown size={12} />
  <ul role="menu" class:visible={open}>
    {@render children()}
  </ul>
</span>

<style>
  span {
    padding: 0.7em 0.5em;
    cursor: pointer;
  }

  span.open,
  span:hover {
    background-color: var(--background-darkest);
  }

  span::after {
    /* Removed content since we use Lucide icon */
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
    margin: 0.5rem 0 0 0;
    overflow-y: auto;
    background-color: var(--glass-bg);
    backdrop-filter: var(--glass-blur);
    border: 1px solid var(--glass-border);
    border-radius: 12px;
    box-shadow: var(--glass-shadow);
    padding: 0.5rem;
  }

  ul.visible {
    display: block;
  }
</style>
