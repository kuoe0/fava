<script lang="ts" module>
  import { 
    Asterisk, Check, Coins, FileText, FolderClosed, FolderOpen, Link, List,
    Lock, Scale, Search, Settings, StickyNote, Tag, Terminal, TriangleAlert, X 
  } from "@lucide/svelte";

  import { _, format } from "../../i18n.ts";
  import type { KeySpec } from "../../keyboard-shortcuts.ts";
  import { keyboardShortcut } from "../../keyboard-shortcuts.ts";
  import { toggle } from "../../lib/set.ts";
  import { journal_show } from "../../stores/journal.ts";

  const toggleText = _("Toggle %(type)s entries");

  const buttons: [
    type: string,
    button_text: string,
    title: string | null,
    shortcut: KeySpec,
    supertype?: string,
  ][] = [
    ["open", "Open", null, "s o"],
    ["close", "Close", null, "s c"],
    ["transaction", "Transaction", null, "s t"],
    ["cleared", "*", _("Cleared transactions"), "t c", "transaction"],
    ["pending", "!", _("Pending transactions"), "t p", "transaction"],
    ["other", "x", _("Other transactions"), "t o", "transaction"],
    ["balance", "Balance", null, "s b"],
    ["note", "Note", null, "s n"],
    ["document", "Document", null, "s d"],
    ["discovered", "D", _("Documents with a #discovered tag"), "d d", "document"],
    ["linked", "L", _("Documents with a #linked tag"), "d l", "document"],
    ["pad", "Pad", null, "s p"],
    ["query", "Query", null, "s q"],
    ["custom", "Custom", null, "s C"],
    ["budget", "B", _("Budget entries"), "s B", "custom"],
    ["metadata", _("Metadata"), _("Toggle metadata"), "m"],
    ["postings", _("Postings"), _("Toggle postings"), "p"],
  ];

  function getIcon(type: string) {
    switch (type) {
      case "open": return FolderOpen;
      case "close": return FolderClosed;
      case "transaction": return FileText;
      case "cleared": return Asterisk;
      case "pending": return TriangleAlert; // Will be overridden in template
      case "other": return X;
      case "balance": return Scale;
      case "note": return StickyNote;
      case "document": return FileText;
      case "discovered": return Search;
      case "linked": return Link;
      case "pad": return Lock;
      case "query": return Terminal;
      case "custom": return Settings;
      case "budget": return Coins;
      case "metadata": return Tag;
      case "postings": return List;
      default: return null;
    }
  }
</script>

<script lang="ts">
  let shownSet = $derived(new Set($journal_show));

  function toggle_type(type: string) {
    journal_show.update((show) => {
      const set = new Set(show);
      toggle(set, type);
      buttons.filter((b) => b[4] === type).forEach((b) => toggle(set, b[0]));
      return [...set].sort();
    });
  }

  let active = $derived((type: string, supertype?: string): boolean =>
    supertype != null
      ? shownSet.has(type) && shownSet.has(supertype)
      : shownSet.has(type),
  );
</script>

<form class="flex-row">
  {#each buttons as [type, button_text, title, shortcut, supertype] (type)}
    {@const FilterIcon = getIcon(type)}
    <button
      type="button"
      title={title ?? format(toggleText, { type: button_text })}
      {@attach keyboardShortcut(shortcut)}
      class:inactive={!active(type, supertype)}
      onclick={() => {
        toggle_type(type);
      }}
      style="display: inline-flex; align-items: center; gap: 0.25em;"
      aria-label={title ?? button_text}
    >
      {#if type === "pending"}
        <span style="font-weight: bold; font-size: 1.1em;">!</span>
      {:else if FilterIcon}
        <FilterIcon size={14} />
      {/if}
      {#if !["cleared", "pending", "other"].includes(type)}
        <span>{button_text}</span>
      {/if}
    </button>
  {/each}
</form>

<style>
  form {
    justify-content: flex-end;
    display: flex;
    flex-wrap: wrap;
    gap: 2px;
  }

  button {
    height: 26px; /* Set fixed height for consistency */
    padding: 0 8px;
    font-size: 0.85em;
    font-weight: normal;
    color: var(--background);
    background-color: var(--link-color);
    border: none;
    border-radius: 0; /* Traditional square style */
    cursor: pointer;
    transition: background-color 0.2s, opacity 0.2s;
  }

  button:hover {
    opacity: 0.9;
  }

  button.inactive {
    color: var(--text-color);
    background-color: var(--sidebar-border, #e0e0e0);
  }

  button.inactive:hover {
    background-color: var(--table-header-background, #d0d0d0);
  }
</style>
