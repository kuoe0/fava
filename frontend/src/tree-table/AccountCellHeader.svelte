<!--
    @component
    Header for the account column.
-->
<script lang="ts">
  import { ChevronsDown } from "@lucide/svelte";
  import { some } from "d3-array";

  import { _ } from "../i18n.ts";
  import { is_descendant_or_equal } from "../lib/account.ts";
  import { expand_all, toggled_accounts } from "../stores/accounts.ts";

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

<span title={help_title}>
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
    min-width: 14em;
    max-width: 30em;
    display: inline-flex;
    align-items: center;
  }

  button {
    display: inline-flex;
    align-items: center;
    gap: 0.25em;
    margin-left: 1em;
    font-weight: normal;
    color: inherit;
    opacity: 0.5;
    background: transparent;
    border: none;
    cursor: pointer;
  }

  button:hover {
    opacity: 1;
    color: var(--link-color);
    background-color: transparent !important;
    box-shadow: none !important;
  }
</style>
