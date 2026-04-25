<!--
    @component
    Account name cell.
-->
<script lang="ts">
  import type { AccountTreeNode } from "../charts/hierarchy.ts";
  import { urlForAccount } from "../helpers.ts";
  import { ChevronDown, ChevronRight } from "@lucide/svelte";
  import { leaf } from "../lib/account.ts";
  import AccountIndicator from "../sidebar/AccountIndicator.svelte";
  import { toggle_account, toggled_accounts } from "../stores/accounts.ts";

  interface Props {
    /** The account node to render the name cell for. */
    node: AccountTreeNode;
  }

  let { node }: Props = $props();

  let { account, children } = $derived(node);
  let is_toggled = $derived($toggled_accounts.has(account));
</script>

<span class="droptarget account-cell" data-account-name={account} style="justify-content: flex-start !important; text-align: left !important;">
  {#if children.length > 0}
    <button
      type="button"
      class="unset"
      onclick={(event) => {
        toggle_account(account, event);
      }}
      aria-label={is_toggled ? "Expand" : "Collapse"}
    >
      {#if is_toggled}
        <ChevronRight size={14} />
      {:else}
        <ChevronDown size={14} />
      {/if}
    </button>
  {:else}
    <span class="toggle-placeholder"></span>
  {/if}
  <a href={$urlForAccount(account)} class="account">
    {leaf(account)}
  </a>
  <AccountIndicator {account} small />
</span>

<style>
  button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 16px;
    height: 16px;
    padding: 0;
    color: var(--treetable-expander);
    background-color: transparent;
    border: none;
    cursor: pointer;
  }

  .toggle-placeholder {
    width: 16px;
    height: 16px;
    flex: 0 0 16px !important;
  }

  a {
    text-align: left !important;
    margin-left: 0 !important;
    flex-grow: 0 !important;
  }

  .account-cell {
    display: flex !important;
    flex: 1;
    align-items: center;
    justify-content: flex-start;
    gap: 0.25em;
    margin-right: auto !important;
  }
</style>
