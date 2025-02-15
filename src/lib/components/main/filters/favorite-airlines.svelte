<script lang="ts">
	import { Check, ChevronsUpDown, X } from 'lucide-svelte';
	import * as Dialog from '$lib/components/ui/dialog';
	import * as Popover from '$lib/components/ui/popover';
	import { ScrollArea } from '$lib/components/ui/scroll-area';
	import { Button, buttonVariants } from '$lib/components/ui/button';

	import { Badge } from '$lib/components/ui/badge';
	import { cn } from '$lib/utils/styling';
	import type { AirlineInfo } from '$lib/types';
	import Combobox from './combobox.svelte';
	import { metrics } from '$lib/analytics';

	interface Props {
		open?: boolean;
		airlines: AirlineInfo[];
		favoriteAirlines: string[];
	}

	let { open = $bindable(false), airlines, favoriteAirlines = $bindable() }: Props = $props();

	$effect(() => {
		if (open) {
			metrics.favoriteAirlinesDialogOpened();
		}
	});

	let popoverOpen = $state(false);
	let triggerRef = $state<HTMLButtonElement>(null!);

	const favoriteAirlinesSet = $derived(new Set(favoriteAirlines));

	function toggleFavorite(airline: AirlineInfo) {
		if (favoriteAirlinesSet.has(airline.airline)) {
			favoriteAirlines = favoriteAirlines.filter((code) => code !== airline.airline);
		} else {
			favoriteAirlines = [...favoriteAirlines, airline.airline];
		}
	}

	function removeFavorite(airlineCode: string) {
		favoriteAirlines = favoriteAirlines.filter((code) => code !== airlineCode);
	}
</script>

<Dialog.Root bind:open>
	<Dialog.Content data-testid="favorite-airlines-dialog" class="max-w-lg">
		<Dialog.Header>
			<Dialog.Title>Manage Favorite Airlines</Dialog.Title>
			<Dialog.Description>
				Add or remove airlines from your favorites list. These airlines will be saved in your
				browser and can be used for quick bag scoring.
			</Dialog.Description>
		</Dialog.Header>

		<div class="space-y-4 py-4">
			<Popover.Root bind:open={popoverOpen}>
				<Popover.Trigger bind:ref={triggerRef}>
					{#snippet child({ props })}
						<Button
							data-testid="favorite-airlines-search-button"
							variant="outline"
							role="combobox"
							aria-expanded={popoverOpen}
							class="w-[300px] justify-between"
							{...props}
						>
							Select an airline
							<ChevronsUpDown class="ml-2 h-4 w-4 shrink-0 opacity-50" />
						</Button>
					{/snippet}
				</Popover.Trigger>
				<Popover.Content
					data-testid="favorite-airlines-search-popover"
					avoidCollisions={false}
					class="w-[300px] p-0"
				>
					<Combobox items={airlines} onSelect={toggleFavorite} placeholder="Search airlines...">
						{#snippet element({ item })}
							{@const isFavorite = favoriteAirlinesSet.has(item.airline)}

							<div class="flex min-w-0 items-center gap-2">
								<Check
									data-testid="favorite-airline-check-icon"
									class={cn('h-4 w-4 flex-shrink-0', isFavorite ? 'opacity-100' : 'opacity-0')}
								/>
								<span class="truncate">{item.airline}</span>
								<span class="flex-shrink-0 text-muted-foreground">({item.region})</span>
							</div>
						{/snippet}
					</Combobox>
				</Popover.Content>
			</Popover.Root>

			<div class="space-y-4">
				<h4 class="text-sm font-medium">
					Selected Airlines ({favoriteAirlines.length})
				</h4>
				<div class="h-[200px] rounded-md border border-muted p-2">
					{#if favoriteAirlines.length > 0}
						<ScrollArea class="h-full">
							<div class="space-y-2">
								{#each favoriteAirlines as airlineName}
									{@const airline = airlines.find((a) => a.airline === airlineName)}

									{#if airline}
										<div class="flex items-center justify-between gap-2">
											<div class="flex items-center gap-2">
												<Badge variant="secondary">
													{airline.region}
												</Badge>
												<span class="text-sm">{airline.airline}</span>
											</div>
											<Button
												variant="ghost"
												size="sm"
												class="mr-1.5 h-8 w-8 p-0"
												onclick={() => removeFavorite(airlineName)}
											>
												<X class="h-4 w-4" />
												<span class="sr-only">Remove {airline.airline}</span>
											</Button>
										</div>
									{/if}
								{/each}
							</div>
						</ScrollArea>
					{:else}
						<p class="text-sm text-muted-foreground">
							No favorite airlines selected. Use the search box above to add airlines to your
							favorites.
						</p>
					{/if}
				</div>
			</div>
		</div>

		<Dialog.Footer>
			<Dialog.Close
				data-testid="favorite-airlines-dialog-close-button"
				class={buttonVariants({ variant: 'outline' })}
			>
				Close
			</Dialog.Close>
		</Dialog.Footer>
	</Dialog.Content>
</Dialog.Root>
