<script lang="ts">
  import AsideContents from "./AsideContents.svelte";

  interface Props {
    active: boolean;
    toggle: () => void;
  }
  let { active, toggle }: Props = $props();
</script>

{#if active}
  <div class="overlay" onclick={toggle} aria-hidden="true"></div>
{/if}
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
    background-color: var(--background);
    border-right: 1px solid var(--sidebar-border);
  }



  @media (width <= 767px) {
    :root {
      --aside-width: 200px;
    }

    aside {
      position: fixed;
      top: 0;
      bottom: 0;
      z-index: 3000;
      width: var(--aside-width);
      margin-left: calc(-1 * var(--aside-width));
      transition: var(--transitions);
    }

    .overlay {
      position: fixed;
      inset: 0;
      z-index: 2999;
      cursor: pointer;
      background: var(--overlay-wrapper-background);
      transition: var(--transitions);
    }

    aside.active {
      margin-left: 0;
    }


  }

  @media print {
    aside {
      display: none;
    }
  }
</style>
