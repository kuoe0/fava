<!--
  @component
   A modal dialog.

   This tries to follow https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/.
-->
<script lang="ts">
  import { X } from "@lucide/svelte";
  import type { Snippet } from "svelte";
  import type { Attachment } from "svelte/attachments";

  import { attemptFocus, getFocusableElements } from "../lib/focus.ts";
  import { router } from "../router.ts";

  interface Props {
    shown: boolean;
    focus?: string;
    closeHandler?: () => void;
    showCloseButton?: boolean;
    children: Snippet;
  }

  let {
    shown,
    focus,
    closeHandler = router.close_overlay,
    showCloseButton = true,
    children,
  }: Props = $props();

  /**
   * A Svelte action to handle focus within a modal.
   */
  const handleFocus: Attachment<HTMLDivElement> = (el) => {
    const keydown = (ev: KeyboardEvent) => {
      if (ev.key === "Tab") {
        const focusable = getFocusableElements(el);
        const first = focusable[0];
        const last = focusable[focusable.length - 1];
        if (ev.shiftKey && document.activeElement === first && last) {
          ev.preventDefault();
          attemptFocus(last);
        } else if (!ev.shiftKey && document.activeElement === last && first) {
          ev.preventDefault();
          attemptFocus(first);
        }
      } else if (ev.key === "Escape") {
        ev.preventDefault();
        closeHandler();
      }
    };

    document.addEventListener("keydown", keydown);

    const selectorFocusEl = focus != null ? el.querySelector(focus) : undefined;
    const focusEl = selectorFocusEl ?? getFocusableElements(el)[0];
    if (focusEl) {
      attemptFocus(focusEl);
    }

    return () => {
      document.removeEventListener("keydown", keydown);
    };
  };
</script>

{#if shown}
  <div class="overlay">
    <div class="background" onclick={closeHandler} aria-hidden="true"></div>
    <div class="content" role="dialog" aria-modal="true" {@attach handleFocus}>
      {@render children()}
      {#if showCloseButton}
        <button type="button" class="close" onclick={closeHandler} aria-label="Close">
          <X size={16} />
        </button>
      {/if}
    </div>
  </div>
{/if}

<style>
  :global(body:has(.overlay)) {
    overflow: hidden;
  }

  .background {
    position: fixed;
    inset: 0;
    width: 100%;
    height: 100%;
    cursor: pointer;
    background: var(--overlay-wrapper-background);
  }

  .overlay {
    position: fixed;
    inset: 0;
    z-index: var(--z-index-overlay);
    display: flex;
    align-items: start;
    justify-content: center;
    width: 100vw;
    height: 100vh;
    overflow: auto;
  }

  .content {
    position: relative;
    display: flex;
    width: 100%;
    max-width: 767px;
    padding: 1em;
    margin: 0.5em;
    margin-top: 10vh;
    background: var(--glass-bg);
    backdrop-filter: var(--glass-blur);
    border: 1px solid var(--glass-border);
    border-radius: 16px;
    box-shadow: var(--glass-shadow);
  }

  .close {
    position: absolute;
    top: 0.5em;
    right: 0.5em;
    width: 24px !important;
    height: 24px !important;
    margin: 0;
    padding: 0;
    display: inline-flex !important;
    align-items: center;
    justify-content: center;
    color: var(--text-color-lighter);
    background-color: transparent !important;
    border: none !important;
    border-radius: 50% !important;
    box-shadow: none !important;
    transition: background-color 0.2s, color 0.2s;
  }

  .close:hover {
    color: var(--text-color);
    background-color: rgba(255, 255, 255, 0.1) !important;
  }

  .content :global(form),
  .content > :global(div) {
    width: 100%;
  }

  @media (width <= 767px) {
    /* Show the modal as a bottom sheet on mobile. */
    .overlay {
      height: 100%;
      align-items: flex-end;
    }

    .background {
      /* Keep the dim background */
      background: var(--overlay-wrapper-background);
    }

    .content {
      height: auto;
      max-height: 85vh;
      margin: 0;
      border-radius: 24px 24px 0 0;
      box-shadow: 0 -5px 20px rgba(0, 0, 0, 0.2);
      overflow-y: auto;
    }
  }
</style>
