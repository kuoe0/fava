<script lang="ts" generics="T extends string">
  import { ChartArea, ChartLine, ChartNoAxesColumn, ChartNoAxesColumnIncreasing, ChartPie, Moon, Network, Settings, Sun } from "@lucide/svelte";

  import type { LocalStoreSyncedStore } from "../lib/store.ts";

  interface Props {
    /** The store to show a switch for. */
    store: LocalStoreSyncedStore<T>;
  }

  let { store }: Props = $props();

  function getIcon(option: string) {
    const opt = option.toLowerCase();
    if (opt.includes("line")) { return ChartLine; }
    if (opt.includes("area")) { return ChartArea; }
    if (opt.includes("stacked")) { return ChartNoAxesColumn; }
    if (opt.includes("single")) { return ChartNoAxesColumnIncreasing; }
    if (opt.includes("treemap")) { return Network; }
    if (opt.includes("sunburst")) { return ChartPie; }
    if (opt.includes("icicle")) { return ChartNoAxesColumnIncreasing; } // Fallback
    if (opt.includes("light dark")) { return Settings; }
    if (opt === "dark") { return Moon; }
    if (opt === "light") { return Sun; }
    return null;
  }

</script>

<span>
  {#each store.values() as [option, name] (option)}
    {@const ChartIcon = getIcon(option)}
    <label class="button" class:muted={$store !== option} style="display: inline-flex; align-items: center; gap: 0.25em;">
      <input type="radio" bind:group={$store} value={option} />
      {#if ChartIcon}
        <ChartIcon size={14} />
      {/if}
      <span>{name}</span>
    </label>
  {/each}
</span>

<style>
  input {
    display: none;
  }

  label + label {
    margin-left: 0.125rem;
  }

  @media print {
    label {
      display: none;
    }
  }
</style>
