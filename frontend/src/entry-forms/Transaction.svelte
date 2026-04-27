<script lang="ts">
  import { Check, ChevronDown, TriangleAlert, X } from "@lucide/svelte";

  import {
    get_narration_transaction,
    get_narrations,
    get_payee_accounts,
    get_payee_transaction,
  } from "../api/index.ts";
  import AutocompleteInput from "../AutocompleteInput.svelte";
  import type { EntryMetadata, Transaction } from "../entries/index.ts";
  import { Posting } from "../entries/index.ts";
  import { _ } from "../i18n.ts";
  import { move } from "../lib/array.ts";
  import { notify_err } from "../notifications.ts";
  import { payees } from "../stores/index.ts";
  import AddMetadataButton from "./AddMetadataButton.svelte";
  import EntryMetadataSvelte from "./EntryMetadata.svelte";
  import PostingSvelte from "./Posting.svelte";

  interface Props {
    entry: Transaction;
  }

  let { entry = $bindable() }: Props = $props();
  let suggestions: string[] | undefined = $state.raw();
  let isOpen = $state(false);

  function selectFlag(flag: string) {
    entry = entry.set("flag", flag);
    isOpen = false;
  }

  function handleKeyDown(event: KeyboardEvent) {
    if (event.key === "*") {
      selectFlag("*");
    } else if (event.key === "!") {
      selectFlag("!");
    } else if (event.key === "x" || event.key === "X") {
      selectFlag("");
    }
  }

  let payee = $derived(entry.payee);
  $effect(() => {
    if (payee) {
      suggestions = undefined;
      if ($payees.includes(payee)) {
        get_payee_accounts({ payee })
          .then((s) => {
            suggestions = s;
          })
          .catch((error: unknown) => {
            notify_err(
              error,
              (err) =>
                `Fetching account suggestions for payee ${payee} failed: ${err.message}`,
            );
          });
      }
    }
  });

  let narration = $derived(entry.get_narration_tags_links());
  let narration_suggestions: string[] = $state.raw([]);
  $effect(() => {
    get_narrations()
      .then((s) => {
        narration_suggestions = s;
      })
      .catch((error: unknown) => {
        notify_err(
          error,
          (err) => `Fetching narration suggestions failed: ${err.message}`,
        );
      });
  });

  // Autofill complete transactions.
  async function autocompleteSelectPayee() {
    if (entry.narration || entry.postings.some((p) => !p.is_empty())) {
      return;
    }
    const payee_transaction = await get_payee_transaction({
      payee: entry.payee,
    });
    entry = payee_transaction.set("date", entry.date);
  }
  async function autocompleteSelectNarration() {
    if (entry.payee || entry.postings.some((p) => !p.is_empty())) {
      return;
    }
    const data = await get_narration_transaction({ narration });
    entry = data.set("date", entry.date); // Copy to "entry" and preserve the date set in the dialog
    narration = entry.get_narration_tags_links();
  }

  // Always have one empty posting at the end.
  $effect(() => {
    if (!entry.postings.some((p) => p.is_empty())) {
      entry = entry.set("postings", entry.postings.concat(Posting.empty()));
    }
  });
</script>

<div class="flex-row">
  <input
    type="date"
    bind:value={
      () => entry.date,
      (date: string) => {
        entry = entry.set("date", date);
      }
    }
    required
  />
  <div class="flag-selector" class:open={isOpen}>
    <button
      type="button"
      onclick={() => (isOpen = !isOpen)}
      onkeydown={handleKeyDown}
      onblur={() => setTimeout(() => (isOpen = false), 200)}
      aria-label="Select Flag"
    >
      {#if entry.flag === '*'}
        <Check size={16} />
      {:else if entry.flag === '!'}
        <TriangleAlert size={16} />
      {:else}
        <X size={16} />
      {/if}
      <ChevronDown size={12} />
    </button>
    {#if isOpen}
      <ul class="options">
        <li onclick={() => { selectFlag('*'); }} class:selected={entry.flag === '*'}>
          <Check size={16} />
        </li>
        <li onclick={() => { selectFlag('!'); }} class:selected={entry.flag === '!'}>
          <TriangleAlert size={16} />
        </li>
        <li onclick={() => { selectFlag(''); }} class:selected={entry.flag === '' || !entry.flag}>
          <X size={16} />
        </li>
      </ul>
    {/if}
  </div>
  <label>
    <span class="hide-on-desktop">{_("Payee")}:</span>
    <AutocompleteInput
      placeholder={_("Payee")}
      bind:value={
        () => entry.payee,
        (payee: string) => {
          entry = entry.set("payee", payee);
        }
      }
      suggestions={$payees}
      onSelect={autocompleteSelectPayee}
      --autocomplete-wrapper-flex="1"
    />
  </label>
  <label class="narration">
    <span class="hide-on-desktop">{_("Narration")}:</span>
    <AutocompleteInput
      placeholder={_("Narration")}
      bind:value={narration}
      suggestions={narration_suggestions}
      onSelect={autocompleteSelectNarration}
      onEnter={() => {
        entry = entry.set_narration_tags_links(narration);
      }}
      onBlur={() => {
        entry = entry.set_narration_tags_links(narration);
      }}
      --autocomplete-wrapper-flex="2"
    />
    <AddMetadataButton
      bind:meta={
        () => entry.meta,
        (meta: EntryMetadata) => {
          entry = entry.set("meta", meta);
        }
      }
    />
  </label>
</div>
<EntryMetadataSvelte
  bind:meta={
    () => entry.meta,
    (meta: EntryMetadata) => {
      entry = entry.set("meta", meta);
    }
  }
/>
<div class="flex-row hide-on-desktop">
  <span class="label">{_("Postings")}:</span>
</div>
{#each entry.postings, index (index)}
  <!-- Using the indexed access (instead of `as posting` in the each) seems to track
         the reactivity differently and avoids cursor jumping on the posting inputs. -->
  {@const posting = entry.postings[index]}
  {#if posting}
    <PostingSvelte
      bind:posting={
        () => posting,
        (posting: Posting) => {
          entry = entry.set("postings", entry.postings.with(index, posting));
        }
      }
      {index}
      {suggestions}
      date={entry.date}
      move={({ from, to }: { from: number; to: number }) => {
        entry = entry.set("postings", move(entry.postings, from, to));
      }}
      remove={() => {
        entry = entry.set("postings", entry.postings.toSpliced(index, 1));
      }}
    />
  {/if}
{/each}

<style>
  .flag-selector {
    position: relative;
    display: inline-block;
    width: 3.5em;
    height: 32px;
  }

  .flag-selector button {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    height: 100%;
    padding: 0 4px;
    background-color: var(--background-darker);
    border: 1px solid var(--border-darker);
    border-radius: 8px;
    color: var(--text-color);
    cursor: pointer;
    box-sizing: border-box;
    transition: border-color 0.2s;
  }

  .flag-selector button:focus {
    border-color: var(--color-focus, #2684ff);
    outline: none;
  }

  .flag-selector .options {
    position: absolute;
    top: 100%;
    left: 0;
    z-index: 10;
    width: 140px;
    margin: 4px 0 0;
    padding: 4px;
    background-color: var(--glass-bg);
    border: 1px solid var(--glass-border);
    border-radius: 8px;
    box-shadow: var(--glass-shadow);
    list-style: none;
    backdrop-filter: var(--glass-blur);
  }

  .flag-selector .options li {
    display: flex;
    align-items: center;
    gap: 0.5em;
    padding: 6px 8px;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.2s;
  }

  .flag-selector .options li:hover {
    background-color: rgba(255, 255, 255, 0.05);
  }

  .flag-selector .options li.selected {
    background-color: var(--link-color);
    color: white;
  }

  input[type="date"] {
    height: 32px;
    box-sizing: border-box;
  }

  .hide-on-desktop {
    display: none;
  }

  @media (width <= 767px) {
    .hide-on-desktop {
      display: initial;
      width: 100%;
    }
  }
</style>
