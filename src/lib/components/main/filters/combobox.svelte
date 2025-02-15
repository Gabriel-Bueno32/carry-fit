<script lang="ts">
	import { VirtualList } from 'svelte-virtuallists';
	import { Search } from 'lucide-svelte';
	import { computeMatchScore } from '$lib/utils/matching';
	import { Input } from '$lib/components/ui/input';
	import { onDestroy, type Snippet } from 'svelte';
	import type { AirlineInfo } from '$lib/types';

	interface Props {
		items: AirlineInfo[];
		onSelect: (item: AirlineInfo) => void;
		placeholder: string;
		element: Snippet<[{ item: AirlineInfo }]>;
	}

	let { items = [], onSelect, placeholder = 'Select items...', element }: Props = $props();

	let searchTerm = $state('');

	const filteredItems = $derived(
		items
			.map((item) => ({
				airline: item,
				score: computeMatchScore(searchTerm.toLowerCase(), item.airline)
			}))
			.filter((item) => {
				if (!searchTerm) return true;
				return item.score > 0.5;
			})
			.toSorted((a, b) => {
				return b.score - a.score;
			})
	);

	onDestroy(() => {
		searchTerm = '';
	});
</script>

<div
	data-testid="combobox-content"
	class="w-full rounded-md border bg-popover text-popover-foreground shadow-md"
>
	<div class="flex items-center border-b px-3 py-1">
		<Search class="mr-2 h-4 w-4 shrink-0 opacity-50" />
		<Input
			class="h-9 w-full border-0 bg-transparent py-3 text-sm outline-none placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-0 focus-visible:ring-offset-0 disabled:cursor-not-allowed disabled:opacity-50"
			bind:value={searchTerm}
			{placeholder}
		/>
	</div>

	<div class="relative h-[200px] w-full">
		{#if filteredItems.length > 0}
			<VirtualList class="virtual-list-viewport h-full w-full" items={filteredItems}>
				{#snippet vl_slot({ index, item })}
					<button
						class="relative flex w-full min-w-0 cursor-pointer select-none items-start rounded-sm px-2 py-1.5 text-left text-sm outline-none hover:bg-accent hover:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
						role="option"
						onclick={() => onSelect(item.airline)}
					>
						<div class="flex min-w-0 flex-1">
							{@render element({ item: item.airline })}
						</div>
					</button>
				{/snippet}
			</VirtualList>
		{:else}
			<div class="absolute inset-0 flex items-center justify-center text-sm text-muted-foreground">
				No airlines found.
			</div>
		{/if}
	</div>
</div>

<style>
	:global(.virtual-list-viewport) {
		scrollbar-width: thin;
		scrollbar-color: hsl(var(--muted-foreground)) transparent;
	}

	:global(.virtual-list-viewport::-webkit-scrollbar) {
		width: 6px;
	}

	:global(.virtual-list-viewport::-webkit-scrollbar-track) {
		background-color: transparent;
	}

	:global(.virtual-list-viewport::-webkit-scrollbar-thumb) {
		background-color: hsl(var(--muted-foreground) / 0.3);
		border-radius: 3px;
	}

	:global(.virtual-list-viewport::-webkit-scrollbar-thumb:hover) {
		background-color: hsl(var(--muted-foreground) / 0.5);
	}
</style>
