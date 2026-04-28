<script lang="ts">
  import { Copy, Search } from "@lucide/svelte";

  import { urlFor } from "../../helpers.ts";
  import { _ } from "../../i18n.ts";
  import QueryTable from "../query/QueryTable.svelte";
  import EntriesByType from "./EntriesByType.svelte";
  import {
    postings_per_account_query,
    type StatisticsReportProps,
  } from "./index.ts";
  import UpdateActivity from "./UpdateActivity.svelte";

  let {
    all_balance_directives,
    balances,
    entries_by_type,
    postings_per_account,
  }: Props = $props();
</script>

<div class="left">
  <h3>
    {_("Postings per Account")}
    (<a href={$urlFor("query", { query_string: postings_per_account_query })} style="display: inline-flex; align-items: center; gap: 0.25em;">
      <Search size={16} />
      <span>{_("Query")}</span>
    </a>)
  </h3>
  <QueryTable table={postings_per_account} />
</div>

<div class="left">
  <h3>
    {_("Update Activity")}
    <copyable-text
      class="button right"
      title={_(
        "Click to copy balance directives for accounts (except green ones) to the clipboard.",
      )}
      data-clipboard-text={all_balance_directives}
      style="display: inline-flex; align-items: center; gap: 0.5em;"
    >
      <Copy size={16} />
      <span>{_("Copy balance directives")}</span>
    </copyable-text>
  </h3>
  <UpdateActivity {balances} />
</div>

<div class="left">
  <h3>{_("Entries per Type")}</h3>
  <EntriesByType {entries_by_type} />
</div>

<style>
  /* Retain original styles or keep it minimal */
  .left {
    float: left;
    width: 48%;
    margin-right: 2%;
    margin-bottom: 2em;
  }

  .right {
    float: right;
  }

  h3 {
    margin-top: 0;
  }

  a {
    color: var(--link-color);
    text-decoration: none;
  }

  a:hover {
    text-decoration: underline;
  }

  copyable-text.button {
    display: inline-flex;
    align-items: center;
    gap: 0.5em;
    padding: 0.4em 1em;
    color: var(--background);
    background-color: var(--link-color);
    border: none;
    border-radius: 0;
    font-size: 0.85em;
    font-weight: normal;
    cursor: pointer;
    transition: opacity 0.2s;
  }

  copyable-text.button:hover {
    opacity: 0.9;
  }
</style>
