<script lang="ts">
	import { Tag } from "@lucide/svelte";

	import type { EntryMetadata } from "../entries/index.ts";
	import { _ } from "../i18n.ts";

	interface Props {
		meta: EntryMetadata;
	}

	let { meta = $bindable() }: Props = $props();
</script>

<button
	type="button"
	class="add-metadata-btn"
	onclick={() => {
		meta = meta.add();
	}}
	tabindex={-1}
	title={_("Add metadata")}
	aria-label={_("Add metadata")}
>
	<Tag size={14} />
</button>

<style>
	.add-metadata-btn {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		width: 24px;
		height: 24px;
		padding: 0;
		color: var(--text-color-lighter);
		background-color: transparent;
		border: 1px solid var(--glass-border);
		border-radius: 50%;
		cursor: pointer;
		transition: background-color 0.2s, color 0.2s, border-color 0.2s;
	}

	.add-metadata-btn:hover {
		color: var(--text-color);
		background-color: rgba(255, 255, 255, 0.1);
		border-color: rgba(255, 255, 255, 0.2);
	}
</style>
