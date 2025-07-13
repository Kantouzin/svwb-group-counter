<script lang="ts">
	import groupInfos from '$lib/data/groupInfos.json';
	import { onMount } from 'svelte';

	interface HistoryEntry {
		type: 'win' | 'loss';
		timestamp: Date;
	}

	type Group = 'emerald' | 'topaz' | 'ruby' | 'sapphire' | 'diamond';
	let selectedGroup: Group = $state('sapphire');
	let history: HistoryEntry[] = $state([]);

	onMount(() => {
		const savedHistory = localStorage.getItem('svwb-group-counter.history');
		if (savedHistory) {
			try {
				const parsed = JSON.parse(savedHistory);
				history = parsed.map((entry: any) => ({
					...entry,
					timestamp: new Date(entry.timestamp)
				}));
			} catch (e) {
				console.error('Failed to load history', e);
			}
		}
	});

	function saveHistory() {
		localStorage.setItem('svwb-group-counter.history', JSON.stringify(history));
	}

	let wins = $derived(history.filter((entry) => entry.type === 'win').length);
	let losses = $derived(history.filter((entry) => entry.type === 'loss').length);

	$effect(() => {
		const originalSelectedGroup = selectedGroup;

		const index = groupInfos.findIndex((group) => group.value === selectedGroup);
		if (index !== -1) {
			const groupInfo = groupInfos[index];
			if (groupInfo.promotion !== null && wins >= groupInfo.promotion) {
				selectedGroup = groupInfos[index - 1].value as Group;
			}
			if (groupInfo.demotion !== null && losses >= groupInfo.demotion) {
				selectedGroup = groupInfos[index + 1].value as Group;
			}
		}

		if (originalSelectedGroup !== selectedGroup) {
			resetHistory();
		}
	});

	function addWin() {
		const newEntry: HistoryEntry = { type: 'win', timestamp: new Date() };
		history = [newEntry, ...history].slice(0, 20);
		saveHistory();
	}

	function addLoss() {
		const newEntry: HistoryEntry = { type: 'loss', timestamp: new Date() };
		history = [newEntry, ...history].slice(0, 20);
		saveHistory();
	}

	function resetHistory() {
		history = [];
		saveHistory();
	}

	function getGroup() {
		return groupInfos.find((group) => group.value === selectedGroup);
	}

	let title = $state('SVWB');
	function setTitle() {
		const messages = ['ShadoBeyo', 'SeWolYon', 'OwWorN'];
		title = messages[Math.floor(Math.random() * messages.length)];
	}
</script>

<svelte:head>
	<title>SVWB Group Counter</title>
</svelte:head>

<div class="flex min-h-screen flex-col">
	<header class="bg-black p-4 text-white">
		<h1 class="cursor-auto text-xl font-bold" ondblclick={setTitle}>
			{title} Group Counter
		</h1>
	</header>

	<main class="flex-grow">
		<div class="mx-auto my-8 max-w-md p-4 text-center">
			<div class="mb-4 flex justify-center">
				<select
					bind:value={selectedGroup}
					onchange={resetHistory}
					class="w-auto max-w-fit rounded border-2 border-gray-200 p-2"
				>
					{#each groupInfos as group}
						<option value={group.value}>{group.text}</option>
					{/each}
				</select>
			</div>

			<div class="mx-auto mb-5 w-full max-w-md">
				<div class="relative h-6 overflow-hidden rounded-full bg-red-500">
					{#if history.length === 0}
						<div class="h-full bg-gray-300"></div>
					{:else}
						<div
							class="h-full bg-green-500"
							style="width: {(wins / (wins + losses)) * 100}%;"
						></div>
					{/if}
				</div>
			</div>

			<div class="mb-8 flex justify-around">
				<div class="min-w-[120px] rounded-lg border-2 border-gray-200 p-4">
					<span class="mb-2 block text-lg text-gray-600">勝利</span>
					<span class="mb-2 block text-2xl font-bold">{wins}</span>
					<span class="text-green-600">
						{#if getGroup()?.promotion !== null}
							{getGroup()?.promotion}
						{:else}
							-
						{/if}
					</span>
				</div>
				<div class="min-w-[120px] rounded-lg border-2 border-gray-200 p-4">
					<span class="mb-2 block text-lg text-gray-600">敗北</span>
					<span class="mb-2 block text-2xl font-bold">{losses}</span>
					<span class="text-red-600">
						{#if getGroup()?.demotion !== null}
							{getGroup()?.demotion}
						{:else}
							-
						{/if}
					</span>
				</div>
			</div>

			<div class="mb-8 flex justify-center gap-4">
				<button
					onclick={addWin}
					class="rounded bg-green-500 px-6 py-3 text-white transition-colors hover:bg-green-600 active:scale-110"
				>
					勝利
				</button>
				<button
					onclick={addLoss}
					class="rounded bg-red-500 px-6 py-3 text-white transition-colors hover:bg-red-600 active:scale-110"
				>
					敗北
				</button>
			</div>

			<div class="history-section">
				<h3 class="mb-4 text-lg font-semibold">勝敗履歴</h3>
				<div class="rounded-lg border border-gray-200 p-4">
					{#if history.length === 0}
						<p class="text-gray-500">履歴がありません</p>
					{:else}
						<div class="space-y-2">
							{#each history as entry (entry.timestamp)}
								<div class="flex justify-between rounded bg-gray-50 p-2 text-sm">
									<span
										class="font-medium {entry.type === 'win' ? 'text-green-600' : 'text-red-600'}"
									>
										{entry.type === 'win' ? '勝利' : '敗北'}
									</span>
									<span class="text-gray-600">
										{new Date(entry.timestamp).toLocaleString('ja-JP')}
									</span>
								</div>
							{/each}
						</div>

						<button
							onclick={resetHistory}
							class="mt-4 rounded bg-gray-500 px-4 py-2 text-white transition-colors hover:bg-gray-600 active:scale-110"
						>
							履歴消去
						</button>
					{/if}
				</div>
			</div>
		</div>
	</main>

	<footer class="bg-black p-4 text-center text-white">© Kantouzin</footer>
</div>
