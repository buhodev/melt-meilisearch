<script>
	import { PUBLIC_MEILISEARCH_PATH, PUBLIC_MEILISEARCH_API_KEY } from '$env/static/public';
	import { createCombobox, melt } from '@melt-ui/svelte';
	import { onMount } from 'svelte';
	import { fly } from 'svelte/transition';
	import MeiliSearch from 'meilisearch';

	let client;
	let isLoading = false;
	let results = [];

	onMount(() => {
		client = new MeiliSearch({
			host: PUBLIC_MEILISEARCH_PATH,
			apiKey: PUBLIC_MEILISEARCH_API_KEY
		});
	});

	const {
		elements: { menu, input, option, label },
		states: { open, isEmpty, inputValue },
		helpers: { isSelected }
	} = createCombobox({
		forceVisible: true
	});

	inputValue.subscribe(({ value, debounced }) => {
		isLoading = debounced !== '' && value !== '' && value.split('')[0] !== '?';

		if (debounced !== '' && value !== '' && debounced === value && value.split('')[0] !== '?') {
			client
				.multiSearch({
					queries: [
						{
							indexUid: 'movies',
							q: value,
							limit: 3
						},
						{
							indexUid: 'books',
							q: value,
							limit: 3
						}
					]
				})
				.then((res) => {
					isLoading = false;

					if (res.results[0].hits.length > 0) {
						// open.set(true);
						results[0] = { id: res.results[0].indexUid, hits: res.results[0].hits };
					}
					if (res.results[1].hits.length > 0) {
						// open.set(true);
						results[1] = { id: res.results[1].indexUid, hits: res.results[1].hits };
					}
					if (res.results[0].hits.length === 0 && res.results[1].hits.length === 0) {
						// open.set(false);
						results = [];
					}
					console.log(res.results);
				});
		}
	});
</script>

<div class="flex flex-col gap-1">
	<!-- svelte-ignore a11y-label-has-associated-control - $label contains the 'for' attribute -->
	<label use:melt={$label}>
		<span class="text-sm font-medium text-gray-900">Choose your favorite book or movie:</span>
	</label>

	<div class="relative">
		<input
			use:melt={$input}
			class="flex h-10 items-center justify-between rounded-lg bg-white px-3 pr-12 text-black"
			placeholder="Best book ever"
		/>
		{#if isLoading}
			loading...
		{:else}
			results:
		{/if}
	</div>
</div>
{#if $open}
	<ul
		class="z-10 flex max-h-[300px] flex-col overflow-hidden rounded-lg"
		use:melt={$menu}
		transition:fly={{ duration: 150, y: -5 }}
	>
		<!-- svelte-ignore a11y-no-noninteractive-tabindex -->
		<div
			class="flex max-h-full flex-col gap-0 overflow-y-auto bg-white px-2 py-2 text-black"
			tabindex="0"
		>
			{#each results as result, index (index)}
				{#if result.id === 'movies' && result.hits.length > 0}
					<h2 class="mb-2 mt-4 px-3 text-xs font-semibold text-gray-900 dark:text-gray-200">
						Movies
					</h2>

					{#each result.hits as movie, index (index)}
						<li
							use:melt={$option({
								value: movie,
								label: movie.title
								// disabled: book.disabled
							})}
							class="relative cursor-pointer scroll-my-2 rounded-md py-2 pl-4 pr-4 data-[highlighted]:bg-gray-200 data-[highlighted]:text-gray-900 data-[disabled]:opacity-50"
						>
							{#if $isSelected(movie)}
								<div class="check absolute left-2 top-1/2 z-10 text-gray-900">
									✓
									<!-- <Check class="square-4" /> -->
								</div>
							{/if}
							<div class="pl-4">
								<span class="font-medium">{movie.title}</span>
								<span class="block text-sm opacity-75">{movie.overview.slice(0, 100)}...</span>
							</div>
						</li>
					{/each}
				{:else if result.id === 'books' && result.hits.length > 0}
					<h2 class="mb-2 mt-4 px-3 text-xs font-semibold text-gray-900 dark:text-gray-200">
						Books
					</h2>
					{#each result.hits as book, index (index)}
						<li
							use:melt={$option({
								value: book,
								label: book.title
								// disabled: book.disabled
							})}
							class="relative cursor-pointer scroll-my-2 rounded-md py-2 pl-4 pr-4 data-[highlighted]:bg-gray-200 data-[highlighted]:text-gray-900 data-[disabled]:opacity-50"
						>
							{#if $isSelected(book)}
								<div class="check absolute left-2 top-1/2 z-10 text-gray-900">
									✓
									<!-- <Check class="square-4" /> -->
								</div>
							{/if}
							<div class="pl-4">
								<span class="font-medium">{book.title}</span>
								<span class="block text-sm opacity-75">{book.author}</span>
							</div>
						</li>
					{/each}
				{/if}
			{/each}
			{#if $isEmpty}
				<li
					class="relative cursor-pointer rounded-md py-1 pl-8 pr-4 data-[highlighted]:bg-gray-100 data-[highlighted]:text-gray-700"
				>
					No results found
				</li>
			{/if}
		</div>
	</ul>
{/if}

<style lang="postcss">
	.check {
		@apply absolute left-2 top-1/2 text-gray-500;
		translate: 0 calc(-50% + 1px);
	}
</style>
