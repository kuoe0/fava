<script lang="ts">
  import { ChevronDown, Menu, Plus,RotateCw } from "@lucide/svelte";

  import { keyboardShortcut } from "../keyboard-shortcuts.ts";
  import { router } from "../router.ts";
  import { ledgerData } from "../stores/index.ts";
  import { ledger_title } from "../stores/options.ts";
  import FilterForm from "./FilterForm.svelte";
  import HeaderIcon from "./HeaderIcon.svelte";
  import { has_changes } from "./page-title.ts";
  import PageTitle from "./PageTitle.svelte";

  interface Props {
    toggle: () => void;
  }
  let { toggle }: Props = $props();

  let other_ledgers = $derived($ledgerData.other_ledgers);
  let has_dropdown = $derived(other_ledgers.length);
</script>

<header>
  <button type="button" class="unset mobile-menu-toggle" onclick={toggle} aria-label="Toggle Sidebar">
    <Menu size={24} />
  </button>
  <HeaderIcon />
  <h1>
    {$ledger_title}{#if has_dropdown}&nbsp;<ChevronDown size={16} style="display: inline; vertical-align: middle;" />{/if}<PageTitle />
    {#if has_dropdown}
      <div class="beancount-files">
        <ul>
          {#each other_ledgers as [name, url] (url)}
            <li>
              <a href={url} data-remote>{name}</a>
            </li>
          {/each}
        </ul>
      </div>
    {/if}
  </h1>
  {#if $has_changes}
    <button
      type="button"
      class="reload-page"
      {@attach keyboardShortcut("r")}
      onclick={router.reload}
      aria-label="Reload Page"
    >
      <RotateCw size={16} />
    </button>
  {/if}
  <span class="spacer"></span>
  <a class="button add-txn" href="#add-transaction" aria-label="Add Transaction">
    <Plus size={16} />
  </a>
  <FilterForm />
</header>

<style>
  .mobile-menu-toggle {
    display: none;
    color: var(--header-color);
  }

  .add-txn {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 32px;
    height: 32px;
    padding: 0;
    margin-right: 0.5em;
    background-color: var(--glass-bg);
    border: 1px solid var(--glass-border);
    border-radius: 50%;
    box-shadow: var(--glass-shadow);
  }

  @media (width <= 767px) {
    .mobile-menu-toggle {
      display: block;
    }
  }

  .reload-page {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 32px;
    height: 32px;
    padding: 0;
    color: var(--dark-gray);
    background-color: var(--warning);
    border-radius: 8px;
  }

  h1 {
    display: inline-block;
    padding: 0.5rem;
    margin: 0;
    overflow: hidden;
    font-size: 16px;
    font-weight: normal;
  }

  a:hover,
  a:link,
  a:visited {
    color: inherit;
  }

  .beancount-files {
    position: absolute;
    z-index: var(--z-index-floating-ui);
    display: none;
    width: 20em;
    margin-top: 0.25em;
    color: var(--link-color);
    background-color: var(--background);
    border: 1px solid var(--border);
    box-shadow: var(--box-shadow-dropdown);
  }

  .beancount-files a {
    display: block;
    padding: 8px 12px 8px 28px;
    cursor: pointer;
  }

  h1:hover .beancount-files {
    display: block;
  }

  .beancount-files ul {
    max-height: 400px;
    margin-bottom: 0;
    overflow-y: auto;
  }

  .beancount-files a:hover {
    color: var(--background);
    background-color: var(--link-color);
  }
</style>
