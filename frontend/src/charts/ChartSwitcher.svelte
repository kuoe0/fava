<script lang="ts">
	import { _ } from "../i18n.ts";
	import type { KeySpec } from "../keyboard-shortcuts.ts";
	import { keyboardShortcut } from "../keyboard-shortcuts.ts";
	import { lastActiveChartName } from "../stores/chart.ts";
	import { show_charts } from "../stores/url.ts";
	import Chart from "./Chart.svelte";
	import { chartContext } from "./context.ts";
	import ConversionAndInterval from "./ConversionAndInterval.svelte";
	import type { ParsedFavaChart } from "./index.ts";
	import { BarChart2, PieChart, Network, BarChart } from "@lucide/svelte";

	interface Props {
		charts: readonly ParsedFavaChart[];
	}

	let { charts }: Props = $props();

	let active_chart = $derived(
		charts.find((c) => c.label === $lastActiveChartName) ?? charts[0],
	);

	// Get the shortcut key for jumping to the previous chart.
	let shortcutPrevious = $derived((index: number): KeySpec | undefined => {
		const current = active_chart ? charts.indexOf(active_chart) : -1;
		return index === (current - 1 + charts.length) % charts.length
			? { key: "C", note: _("Previous") }
			: undefined;
	});
	// Get the shortcut key for jumping to the next chart.
	let shortcutNext = $derived((index: number): KeySpec | undefined => {
		const current = active_chart ? charts.indexOf(active_chart) : -1;
		return index === (current + 1 + charts.length) % charts.length
			? { key: "c", note: _("Next") }
			: undefined;
	});

	function getIcon(label: string) {
		if (label.includes("Treemap")) return Network;
		if (label.includes("Sunburst")) return PieChart;
		if (label.includes("Stacked Bars")) return BarChart2;
		if (label.includes("Single Bars")) return BarChart;
		return null;
	}
</script>

{#if active_chart}
	<Chart chart={active_chart.with_context($chartContext)}>
		<ConversionAndInterval />
	</Chart>
	<div hidden={!$show_charts} class="chart-switcher-container" class:many-items={charts.length > 10}>
		{#each charts as chart, index (chart.label)}
			{@const Icon = getIcon(chart.label)}
			<button
				type="button"
				class="unset"
				class:selected={chart === active_chart}
				onclick={() => {
					$lastActiveChartName = chart.label;
				}}
				{@attach keyboardShortcut(shortcutPrevious(index))}
				{@attach keyboardShortcut(shortcutNext(index))}
				aria-label={chart.label}
			>
				{#if Icon}
					<Icon size={16} />
				{/if}
				<span>{chart.label}</span>
			</button>
		{/each}
	</div>
{/if}

<style>
	.chart-switcher-container {
		display: flex;
		justify-content: center;
		width: max-content;
		max-width: 100%;
		gap: 4px;
		padding: 4px !important;
		margin: 0 auto 1em auto;
		background-color: var(--glass-bg);
		backdrop-filter: var(--glass-blur);
		border: 1px solid var(--glass-border);
		border-radius: 12px;
		box-shadow: var(--glass-shadow);
	}

	/* For few items, force no wrap and allow scrolling if needed */
	.chart-switcher-container:not(.many-items) {
		flex-wrap: nowrap;
		overflow-x: auto;
	}

	/* For many items, allow wrapping to see all */
	.chart-switcher-container.many-items {
		flex-wrap: wrap;
	}

	button {
		display: flex !important;
		align-items: center;
		gap: 0.5em;
		padding: 0.2em 1em !important;
		height: 28px !important;
		color: var(--text-color-lighter);
		background-color: transparent;
		border: none;
		border-radius: 8px;
		transition: background-color 0.2s, color 0.2s;
		flex-shrink: 0;
		white-space: nowrap;
	}

	button:hover {
		color: var(--text-color);
		background-color: rgba(255, 255, 255, 0.1);
	}

	button.selected {
		color: var(--background);
		background-color: var(--link-color);
	}

	@media print {
		.chart-switcher-container {
			display: none;
		}
	}
</style>
