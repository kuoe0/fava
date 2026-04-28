<script lang="ts">
  import { Menu, Plus } from "@lucide/svelte";
  import AsideContents from "./AsideContents.svelte";

  /** Whether the sidebar is currently shown. */
  let active = $state(false);
  const toggle = () => {
    active = !active;
  };
</script>

{#if active}
  <div class="overlay" onclick={toggle} aria-hidden="true"></div>
{/if}
<div class:active class="aside-buttons">
  <button type="button" onclick={toggle} aria-label="Toggle Sidebar">
    <Menu size={24} />
  </button>
  <a class="button" href="#add-transaction" aria-label="Add Transaction">
    <Plus size={24} />
  </a>
</div>
<aside class:active>
  <AsideContents />
</aside>

<style>
  aside {
    grid-area: aside;
    padding-top: 0.5rem;
    margin: 0;
    overflow-y: auto;
    color: var(--sidebar-color);
    background-color: var(--sidebar-background);
    border-right: 1px solid var(--sidebar-border);
  }

  .aside-buttons {
    display: none;
  }

  @media (width <= 767px) {
    :root {
      --aside-width: 200px;
    }

    aside {
      position: fixed;
      top: 0;
      bottom: 0;
      z-index: var(--z-index-floating-ui);
      width: var(--aside-width);
      margin-left: calc(-1 * var(--aside-width));
      transition: var(--transitions);
    }

    .overlay {
      position: fixed;
      inset: 0;
      z-index: var(--z-index-floating-ui);
      cursor: pointer;
      background: var(--overlay-wrapper-background);
      transition: var(--transitions);
    }

    aside.active {
      margin-left: 0;
    }

    .aside-buttons {
      position: fixed;
      top: 0;
      left: 0;
      z-index: var(--z-index-floating-ui);
      display: flex;
      flex-direction: column;
      transition: var(--transitions);
    }

    .active.aside-buttons {
      left: var(--aside-width);
    }

    .aside-buttons > * {
      display: inline-flex;
      align-items: center;
      justify-content: center;
      width: 42px;
      height: 42px;
      color: var(--mobile-button-text);
      background-color: var(--sidebar-background);
      border: 1px solid var(--sidebar-border);
      padding: 0;
    }


  }

  @media print {
    aside,
    .aside-buttons {
      display: none;
    }
  }
</style>
