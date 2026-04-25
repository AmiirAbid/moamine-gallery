<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/moafavicon.ico';
	import Navbar from '$lib/components/Navbar.svelte';
	import ThemeToggle from '$lib/components/ThemeToggle.svelte';
	let { children } = $props();

</script>

<svelte:head>
	<title>Moamine Gallery</title>
	<link rel="icon" href={favicon} />
	<link rel="stylesheet" href="/fonts/satoshi.css" />
</svelte:head>

<Navbar />
<ThemeToggle />
{@render children()}
