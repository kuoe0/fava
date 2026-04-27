<script lang="ts" module>
	import { 
Check, Coins,
FileText, FolderClosed, 		FolderOpen, Link, List,
Lock, 		Scale, Search, Settings, 		StickyNote, Tag, Terminal, TriangleAlert, X	} from "@lucide/svelte";

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
			case "cleared": return Check;
			case "pending": return TriangleAlert;
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

<form class="flex-row tags-container">
	{#each buttons as [type, button_text, title, shortcut, supertype] (type)}
		{@const filterIcon = getIcon(type)}
		<button
			type="button"
			title={title ?? format(toggleText, { type: button_text })}
			{@attach keyboardShortcut(shortcut)}
			class:active={active(type, supertype)}
			class="tag"
			onclick={() => {
				toggle_type(type);
			}}
		>
			{#if filterIcon}
				<svelte:component this={filterIcon} size={14} />
			{/if}
			<span>{button_text}</span>
		</button>
	{/each}
</form>

<style>
	.tags-container {
		display: flex;
		flex-wrap: wrap;
		gap: 6px;
		justify-content: flex-start;
		padding: 8px;
		background-color: var(--glass-bg);
		backdrop-filter: var(--glass-blur);
		border: 1px solid var(--glass-border);
		border-radius: 16px;
	}

	.tag {
		display: inline-flex;
		align-items: center;
		gap: 4px;
		padding: 4px 10px;
		font-size: 0.85rem;
		color: var(--text-color-lighter);
		background-color: rgba(255, 255, 255, 0.05);
		border: 1px solid rgba(255, 255, 255, 0.1);
		border-radius: 20px;
		cursor: pointer;
		box-shadow: none;
		transition: background-color 0.2s, color 0.2s, transform 0.1s;
	}

	.tag:hover {
		color: var(--text-color);
		background-color: rgba(255, 255, 255, 0.15);
		transform: translateY(-1px);
	}

	.tag.active {
		color: var(--background);
		background-color: var(--link-color);
		border-color: var(--link-color);
	}

	.tag:active {
		transform: translateY(0);
	}
</style>
