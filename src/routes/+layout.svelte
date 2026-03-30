<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.svg';
	import Navbar from '$lib/components/Navbar.svelte';
	let { children } = $props();
</script>

<svelte:head>
	<title>Moamine Gallery</title>
	<link rel="icon" href={favicon} />
	<link rel="stylesheet" href="/fonts/satoshi.css" />
</svelte:head>

<Navbar />
{@render children()}
