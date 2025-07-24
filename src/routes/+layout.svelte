<script lang="ts">
	import Analytics from '$lib/Analytics.svelte';
	import '../app.css';

	let { children } = $props();
</script>

<svelte:head>
	<title>SVWB Group Counter</title>
	<meta
		name="description"
		content="『SVWB』の勝敗直近20戦をカウントするアプリです。グループ昇降格までの回数を可視化します。"
	/>
</svelte:head>

<Analytics />

{@render children()}
