<script lang="ts">
	import { urlFor } from "../../helpers.ts";
	import { _ } from "../../i18n.ts";
	import QueryLinks from "../query/QueryLinks.svelte";
	import QueryTable from "../query/QueryTable.svelte";
	import type { HoldingsReportProps } from "./index.ts";
	import { Search } from "@lucide/svelte";

	let {
		aggregation_key,
		query_string,
		query_result_table,
	}: HoldingsReportProps = $props();
</script>

<div class="headerline">
	<div class="segmented-control">
		<a href={$urlFor("holdings/")} class:selected={aggregation_key === "all"}>
			{_("Holdings")}
		</a>
		<a href={$urlFor("holdings/by_account/")} class:selected={aggregation_key === "by_account"}>
			{_("Holdings by")} {_("Account")}
		</a>
		<a href={$urlFor("holdings/by_currency/")} class:selected={aggregation_key === "by_currency"}>
			{_("Holdings by")} {_("Currency")}
		</a>
		<a href={$urlFor("holdings/by_cost_currency/")} class:selected={aggregation_key === "by_cost_currency"}>
			{_("Holdings by")} {_("Cost currency")}
		</a>
	</div>
</div>

<div class="actions-line">
	<a class="button" href={$urlFor("query", { query_string })}>
		<Search size={16} />
		<span>{_("Query")}</span>
	</a>
	<QueryLinks query={query_string} />
</div>

<QueryTable table={query_result_table} filter_empty="units" />

<style>
	.headerline {
		margin-bottom: 1em;
	}

	.segmented-control {
		display: inline-flex;
		gap: 2px;
		padding: 4px;
		background-color: var(--glass-bg);
		backdrop-filter: var(--glass-blur);
		border: 1px solid var(--glass-border);
		border-radius: 12px;
		box-shadow: var(--glass-shadow);
		overflow-x: auto;
		max-width: 100%;
	}

	.segmented-control a {
		padding: 0.4em 1em;
		color: var(--text-color-lighter);
		text-decoration: none;
		border-radius: 8px;
		transition: background-color 0.2s, color 0.2s;
		flex-shrink: 0;
		white-space: nowrap;
	}

	.segmented-control a:hover {
		color: var(--text-color);
		background-color: rgba(255, 255, 255, 0.1);
	}

	.segmented-control a.selected {
		color: var(--background);
		background-color: var(--link-color);
	}

	.actions-line {
		display: flex;
		gap: 10px;
		align-items: center;
		margin-bottom: 1em;
	}

	.actions-line a.button {
		display: inline-flex;
		align-items: center;
		gap: 0.5em;
		padding: 0.4em 1em;
		color: var(--text-color-lighter);
		background-color: var(--glass-bg);
		border: 1px solid var(--glass-border);
		border-radius: 8px;
		text-decoration: none;
	}

	.actions-line a.button:hover {
		color: var(--text-color);
		background-color: rgba(255, 255, 255, 0.1);
	}
</style>
