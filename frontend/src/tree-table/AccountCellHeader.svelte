<!--
    @component
    Header for the account column.
-->
<script lang="ts">
  import { some } from "d3-array";

  import { _ } from "../i18n.ts";
  import { is_descendant_or_equal } from "../lib/account.ts";
  import { expand_all, toggled_accounts } from "../stores/accounts.ts";
  import { ChevronsDown } from "@lucide/svelte";

  interface Props {
    /** The account this tree is rendered for. */
    account: string;
  }

  let { account }: Props = $props();

  const toggled_children = $derived(
    some($toggled_accounts, is_descendant_or_equal(account)),
  );

  const help_title = _(
    "Hold Shift while clicking to expand all children.\n" +
      "Hold Ctrl or Cmd while clicking to expand one level.",
  );
</script>

<span title={help_title} class="account-cell">
  {#if toggled_children}
    <button
      type="button"
      title={_("Expand all accounts")}
      onclick={() => {
        expand_all(account);
      }}
    >
      <ChevronsDown size={14} />
      <span>{_("Expand all")}</span>
    </button>
  {/if}
</span>

<style>
  span {
    flex: 1;
    display: inline-flex;
    align-items: center;
    height: 24px;
    border: none !important;
  }

  button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 0.25em;
    height: 24px;
    padding: 0;
    margin-left: 0;
    font-size: 0.85em;
    font-weight: normal;
    color: var(--text-color-lighter);
    background-color: transparent;
    border: none;
    cursor: pointer;
    transition: color 0.2s;
    white-space: nowrap;
  }

  button:hover {
    color: var(--text-color);
  }
</style>
