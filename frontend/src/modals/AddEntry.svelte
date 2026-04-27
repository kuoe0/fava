<script lang="ts">
  import { Save } from "@lucide/svelte";

  import { save_entries } from "../api/index.ts";
  import { Balance, Note, Transaction } from "../entries/index.ts";
  import Entry from "../entry-forms/Entry.svelte";
  import { todayAsString } from "../format.ts";
  import { _ } from "../i18n.ts";
  import { router } from "../router.ts";
  import { addEntryContinue } from "../stores/editor.ts";
  import { hash } from "../stores/url.ts";
  import ModalBase from "./ModalBase.svelte";

  /** The entry types which have support adding in the modal. */
  const entryTypes = [
    [Transaction, _("Transaction")],
    [Balance, _("Balance")],
    [Note, _("Note")],
  ] as const;

  // For the first entry to be added, use today as the default date.
  let entry: Transaction | Balance | Note = $state.raw(
    Transaction.empty(todayAsString()),
  );

  async function submit(event: SubmitEvent) {
    event.preventDefault();
    await save_entries([entry]);
    const added_entry_date = entry.date;
    // Reuse the date of the entry that was just added.
    // @ts-expect-error all these entries have that static method, but TS is not able to determine that
    // eslint-disable-next-line @typescript-eslint/no-unsafe-assignment, @typescript-eslint/no-unsafe-call
    entry = entry.constructor.empty(added_entry_date);
    if (!$addEntryContinue) {
      router.close_overlay();
    }
  }

  let shown = $derived($hash === "add-transaction");
</script>

<ModalBase {shown} focus=".payee input" showCloseButton={false}>
  <form onsubmit={submit} class="flex-column modal-form">
    <div class="modal-header">
      <h3>{_("Add Entry")}</h3>
      <div class="header-actions">
        <div class="pill-container">
          {#each entryTypes as [Cls, displayName] (displayName)}
            <button
              type="button"
              class="pill-button"
              class:selected={entry instanceof Cls}
              onclick={() => {
                entry = Cls.empty(entry.date);
              }}
            >
              {displayName}
            </button>
          {/each}
        </div>

      </div>
    </div>

    <div class="modal-content">
      <Entry bind:entry />
    </div>

    <div class="modal-actions">
      <label class="continue-label">
        <input type="checkbox" bind:checked={$addEntryContinue} />
        <span class="switch"></span>
        <span>{_("continue")}</span>
      </label>
      <span class="spacer"></span>
      <button type="button" class="muted" onclick={() => { router.close_overlay(); }}>
        {_("Cancel")}
      </button>
      <button type="submit" class="primary-btn">
        <Save size={16} />
        <span>{_("Save")}</span>
      </button>
    </div>
  </form>
</ModalBase>

<style>
  .modal-form {
    gap: 0;
  }

  .modal-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 1rem;
    border-bottom: 1px solid var(--glass-border);
  }

  h3 {
    margin: 0;
    font-size: 1.2rem;
  }

  .header-actions {
    display: flex;
    align-items: center;
    gap: 1rem;
  }

  .pill-container {
    display: inline-flex;
    gap: 4px;
    padding: 4px !important;
    background-color: rgba(0, 0, 0, 0.1);
    border-radius: 10px;
    overflow: hidden;
  }

  .pill-button {
    display: flex !important;
    align-items: center;
    justify-content: center;
    gap: 0.5em;
    padding: 0.4em 1em !important;
    height: 32px !important;
    font-size: 0.9em;
    font-weight: 500;
    color: var(--text-color-lighter);
    background-color: transparent !important;
    box-shadow: none !important;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    transition: background-color 0.2s, color 0.2s, transform 0.1s;
  }

  .pill-button:hover {
    color: var(--text-color);
    background-color: rgba(255, 255, 255, 0.05) !important;
  }

  .pill-button.selected {
    color: var(--background) !important;
    background-color: var(--link-color) !important;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1) !important;
  }

  .pill-button:active {
    transform: scale(0.95);
  }

  .modal-content {
    padding: 1.5rem 1rem;
  }

  .modal-actions {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 1rem;
    border-top: 1px solid var(--glass-border);
  }

  .continue-label {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    color: var(--text-color-lighter);
    cursor: pointer;
  }

  .continue-label input {
    display: none;
  }

  .continue-label .switch {
    position: relative;
    display: inline-block;
    width: 40px;
    height: 22px;
    background-color: rgba(128, 128, 128, 0.2);
    border-radius: 11px;
    transition: background-color 0.3s ease, box-shadow 0.2s ease;
    box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.1);
  }

  .continue-label .switch::after {
    content: "";
    position: absolute;
    width: 18px;
    height: 18px;
    border-radius: 50%;
    background-color: white;
    top: 2px;
    left: 2px;
    transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  }

  .continue-label input:checked + .switch {
    background-color: var(--link-color);
    box-shadow: inset 0 1px 2px rgba(0, 0, 0, 0.1);
  }

  .continue-label input:checked + .switch::after {
    transform: translateX(18px);
  }

  .primary-btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 0.5em;
    padding: 0.6em 1.2em;
    color: var(--background);
    background-color: var(--link-color);
    border: none;
    border-radius: 10px;
    font-weight: 600;
    cursor: pointer;
    transition: background-color 0.2s, transform 0.1s, box-shadow 0.2s;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  .primary-btn:hover {
    background-color: var(--link-hover-color);
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
  }

  .primary-btn:active {
    transform: scale(0.98);
  }

  .muted {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 0.5em;
    padding: 0.6em 1.2em;
    background-color: rgba(255, 255, 255, 0.05);
    border: 1px solid var(--glass-border);
    border-radius: 10px;
    color: var(--text-color-lighter);
    font-weight: 500;
    cursor: pointer;
    transition: background-color 0.2s, color 0.2s, transform 0.1s;
  }

  .muted:hover {
    background-color: rgba(255, 255, 255, 0.1);
    color: var(--text-color);
  }

  .muted:active {
    transform: scale(0.98);
  }
  @media (max-width: 600px) {
    .modal-header {
      flex-direction: column;
      align-items: flex-start;
      gap: 1rem;
    }

    .header-actions {
      width: 100%;
      justify-content: space-between;
    }

    .pill-container {
      flex: 1;
      display: flex;
    }

    .pill-button {
      flex: 1;
      text-align: center;
    }

    .modal-actions {
      flex-wrap: wrap;
      gap: 0.5rem;
    }

    .modal-actions .spacer {
      display: none;
    }

    .continue-label {
      width: 100%;
    }

    .primary-btn, .muted {
      flex: 1;
      justify-content: center;
    }
  }
</style>
