<script lang="ts">
  import type { SourceNode } from "../../lib/sources.ts";
  import Sources from "./Sources.svelte";
  import { ChevronDown, ChevronRight } from "@lucide/svelte";
  import { expanded_directories, toggle_directory } from "./stores.ts";

  interface Props {
    is_root?: boolean;
    node: SourceNode;
    on_select: (source: string) => void;
    selected: string;
  }

  let { is_root = false, node, on_select, selected }: Props = $props();

  let is_expanded = $derived.by(() => {
    const result = $expanded_directories.get(node.path);
    // Even though root is always expanded, treat is as being collapsed by default.
    // This allows for expanding everything with one Ctrl/Meta-Click. The subsequent click would then collapse everything.
    return result ?? (!is_root && selected.startsWith(node.path));
  });

  let is_directory = $derived(node.children.length > 0);
  // Show where the selected file would be, if directories are collapsed
  let is_selected = $derived(
    is_directory && !is_expanded && !is_root
      ? selected.startsWith(node.path)
      : selected === node.path,
  );

  let action = (event: MouseEvent) => {
    if (is_directory) {
      toggle_directory(node.path, !is_expanded, event);
    } else {
      on_select(node.path);
    }
    event.stopPropagation();
  };

  let fileName = $derived(node.name.split('/').pop() || '');
  let dirPath = $derived(node.name.substring(0, node.name.lastIndexOf('/') + 1));
</script>

<li class:selected={is_selected} role="menuitem">
  {#if is_root}
    <button
      type="button"
      title="Beancount data root directory
Shift-Click to expand/collapse immediate directories
Ctrl-/Cmd-/Meta-Click to expand/collapse all directories."
      class="unset root-modern"
      onclick={action}
    >
      <div class="file-info">
        <span class="file-name">{fileName}</span>
        <span class="file-path">{dirPath}</span>
      </div>
    </button>
  {:else}
    <p>
      {#if is_directory}
        <button type="button" class="unset toggle" onclick={action} aria-label={is_expanded ? "Collapse" : "Expand"}>
          {#if is_expanded}
            <ChevronDown size={14} />
          {:else}
            <ChevronRight size={14} />
          {/if}
        </button>
      {:else}
        <span class="toggle-placeholder"></span>
      {/if}
      <button type="button" class="unset leaf" onclick={action}>
        {node.name}
      </button>
    </p>
  {/if}
  {#if is_directory && (is_expanded || is_root)}
    <ul>
      {#each node.children as child (child.path)}
        <Sources node={child} {on_select} {selected} />
      {/each}
    </ul>
  {/if}
</li>

<style>
  ul {
    padding: 0;
    margin-left: 8px;
    border-left: 1px solid var(--glass-border);
  }

  p {
    display: flex;
    align-items: center;
    gap: 0.25em;
    padding-right: 0.5em;
    margin: 0;
    overflow: hidden;
  }

  .selected {
    background-color: var(--table-header-background);
  }

  .leaf {
    flex-grow: 1;
    text-align: left;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    min-width: 0;
  }

  .root {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    min-width: 0;
    text-align: left;
  }

  .toggle {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 16px;
    height: 16px;
    padding: 0;
    color: var(--treetable-expander);
  }

  .toggle-placeholder {
    width: 16px;
    height: 16px;
  }

  .root {
    margin: 0 0.25rem;
  }

  .root-modern {
    display: block;
    width: 100%;
    padding: 8px 12px;
    text-align: left;
    cursor: pointer;
  }

  .file-info {
    display: flex;
    flex-direction: column;
    gap: 2px;
  }

  .file-name {
    font-size: 14px;
    font-weight: 600;
    color: var(--text-color);
  }

  .file-path {
    font-size: 11px;
    color: var(--text-color-lightest);
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
</style>
