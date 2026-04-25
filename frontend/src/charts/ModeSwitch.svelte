<script lang="ts" generics="T extends string">
	import type { LocalStoreSyncedStore } from "../lib/store.ts";
	import { LineChart, AreaChart, BarChart2, BarChart, Network, PieChart, Settings, Moon, Sun } from "@lucide/svelte";

	interface Props {
		/** The store to show a switch for. */
		store: LocalStoreSyncedStore<T>;
	}

	let { store }: Props = $props();

	function getIcon(option: string) {
		const opt = option.toLowerCase();
		if (opt.includes("line")) return LineChart;
		if (opt.includes("area")) return AreaChart;
		if (opt.includes("stacked")) return BarChart2;
		if (opt.includes("single")) return BarChart;
		if (opt.includes("treemap")) return Network;
		if (opt.includes("sunburst")) return PieChart;
		if (opt.includes("icicle")) return BarChart; // Fallback
		if (opt.includes("light dark")) return Settings;
		if (opt === "dark") return Moon;
		if (opt === "light") return Sun;
		return null;
	}
</script>

<span class="button-group">
	{#each store.values() as [option, name] (option)}
		{@const Icon = getIcon(option)}
		<label class="switch-option" class:selected={$store === option}>
			<input type="radio" bind:group={$store} value={option} />
			{#if Icon}
				<Icon size={16} />
			{/if}
			<span>{name}</span>
		</label>
	{/each}
</span>

<style>
	input {
		display: none;
	}

	.button-group {
		display: inline-flex;
		gap: 2px;
		padding: 2px !important;
		background-color: var(--glass-bg);
		backdrop-filter: var(--glass-blur);
		border: 1px solid var(--glass-border);
		border-radius: 12px;
		box-shadow: var(--glass-shadow);
		overflow: hidden;
	}

	.switch-option {
		display: flex !important;
		align-items: center;
		gap: 0.5em;
		padding: 0.2em 1em !important;
		height: 28px !important;
		color: var(--text-color-lighter);
		background-color: transparent;
		border: none;
		border-radius: 8px;
		cursor: pointer;
		transition: background-color 0.2s, color 0.2s;
	}

	.switch-option:hover {
		color: var(--text-color);
		background-color: rgba(255, 255, 255, 0.1);
	}

	.switch-option.selected {
		color: var(--background);
		background-color: var(--link-color);
	}

	@media print {
		.button-group {
			display: none;
		}
	}
</style>
