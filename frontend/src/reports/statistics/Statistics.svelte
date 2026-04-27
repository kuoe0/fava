<script lang="ts">
	import { Copy,Search } from "@lucide/svelte";

	import { urlFor } from "../../helpers.ts";
	import { _ } from "../../i18n.ts";
	import QueryTable from "../query/QueryTable.svelte";
	import EntriesByType from "./EntriesByType.svelte";
	import {
		postings_per_account_query,
		type StatisticsReportProps,
	} from "./index.ts";
	import UpdateActivity from "./UpdateActivity.svelte";

	let {
		all_balance_directives,
		balances,
		entries_by_type,
		postings_per_account,
	}: StatisticsReportProps = $props();
</script>

<div class="grid-container">
	<div class="glass-card">
		<h3>
			<span>{_("Postings per Account")}</span>
			<a class="query-link" href={$urlFor("query", { query_string: postings_per_account_query })}>
				<Search size={16} />
				<span>{_("Query")}</span>
			</a>
		</h3>
		<QueryTable table={postings_per_account} />
	</div>

	<div class="glass-card">
		<h3>
			<span>{_("Update Activity")}</span>
			<copyable-text
				class="button"
				title={_(
					"Click to copy balance directives for accounts (except green ones) to the clipboard.",
				)}
				data-clipboard-text={all_balance_directives}
			>
				<Copy size={16} />
				<span>{_("Copy balance directives")}</span>
			</copyable-text>
		</h3>
		<UpdateActivity {balances} />
	</div>

	<div class="glass-card">
		<h3>{_("Entries per Type")}</h3>
		<EntriesByType {entries_by_type} />
	</div>
</div>

<style>
	.grid-container {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(max-content, 1fr));
		gap: 1rem;
		margin-top: 1rem;
	}

	.glass-card {
		padding: 1rem;
		background-color: var(--glass-bg);
		backdrop-filter: var(--glass-blur);
		border: 1px solid var(--glass-border);
		border-radius: 16px;
		box-shadow: var(--glass-shadow);
		display: flex;
		flex-direction: column;
	}

	.glass-card :global(table) {
		border-collapse: separate !important;
		border-spacing: 0 !important;
		border-radius: 12px;
		overflow: hidden;
		border: 1px solid var(--glass-border) !important;
	}

	.glass-card :global(th),
	.glass-card :global(td) {
		border-width: 0.5px !important;
	}

	h3 {
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin-top: 0;
		font-size: 1.2rem;
		color: var(--text-color);
	}

	a.query-link {
		display: inline-flex;
		align-items: center;
		gap: 0.25em;
		font-size: 0.9rem;
		color: var(--link-color);
		text-decoration: none;
	}

	a.query-link:hover {
		text-decoration: underline;
	}

	copyable-text.button {
		display: inline-flex;
		align-items: center;
		gap: 0.5em;
		padding: 0.4em 1em;
		font-size: 0.85rem;
		color: var(--background);
		background-color: var(--link-color);
		border-radius: 8px;
		cursor: pointer;
		transition: opacity 0.2s;
	}

	copyable-text.button:hover {
		opacity: 0.9;
	}
</style>
