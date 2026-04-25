<script lang="ts">
  import type { Document } from "../../entries/index.ts";
  import { _ } from "../../i18n.ts";
  import { is_descendant_or_equal } from "../../lib/account.ts";
  import { basename } from "../../lib/paths.ts";
  import { DateColumn, Sorter, StringColumn } from "../../sort/index.ts";
  import SortHeader from "../../sort/SortHeader.svelte";
  import { selectedAccount } from "./stores.ts";

  interface Props {
    data: Document[];
    selected?: Document | null;
  }

  let { data, selected = $bindable(null) }: Props = $props();

  /**
   * Extract just the latter part of the filename if it starts with a date.
   */
  function name(doc: Document) {
    const base = basename(doc.filename);
    return base.startsWith(doc.date) ? base.substring(11) : base;
  }

  const columns = [
    new DateColumn<Document>(_("Date")),
    new StringColumn<Document>(_("Name"), (d) => name(d)),
  ] as const;
  let sorter = $state(new Sorter(columns[0], "desc"));

  let is_descendant_of_selected = $derived(
    is_descendant_or_equal($selectedAccount),
  );
  let filtered_documents = $derived(
    data.filter((doc) => is_descendant_of_selected(doc.account)),
  );
  let sorted_documents = $derived(sorter.sort(filtered_documents));
</script>

<div class="table-container">
  <table>
    <thead>
      <tr>
        {#each columns as column (column)}
          <SortHeader bind:sorter {column} />
        {/each}
      </tr>
    </thead>
    <tbody>
      {#each sorted_documents as doc (doc.filename)}
        <tr
          class:selected={selected === doc}
          draggable={true}
          title={doc.filename}
          ondragstart={(ev) => {
            ev.dataTransfer?.setData("fava/filename", doc.filename);
          }}
          onclick={() => {
            selected = doc;
          }}
        >
          <td>{doc.date}</td>
          <td>{name(doc)}</td>
        </tr>
      {/each}
    </tbody>
  </table>
</div>

<style>
  .table-container {
    overflow: hidden;
    border-radius: 12px;
    border: 1px solid var(--glass-border);
    background-color: var(--glass-bg);
    backdrop-filter: var(--glass-blur);
    box-shadow: var(--glass-shadow);
  }

  table {
    width: 100%;
    border-collapse: collapse;
  }

  tr {
    cursor: pointer;
    transition: background-color 0.2s;
  }

  .selected,
  tr:hover {
    background-color: rgba(255, 255, 255, 0.05);
  }

  th, td {
    padding: 0.75em 1em;
    border-bottom: 1px solid var(--glass-border);
  }

  tr:last-child td {
    border-bottom: none;
  }
</style>
