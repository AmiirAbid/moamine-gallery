<script lang="ts">
	import './layout.css';
	import favicon from '$lib/assets/favicon.ico';
	import Navbar from '$lib/components/Navbar.svelte';
	import ThemeToggle from '$lib/components/ThemeToggle.svelte';
	let { children } = $props();

	let contactOpen = $state(false);
</script>

<svelte:head>
	<title>Moamine Gallery</title>
	<link rel="icon" href={favicon} />
	<link rel="stylesheet" href="/fonts/satoshi.css" />
</svelte:head>

<Navbar bind:contactOpen />
<ThemeToggle {contactOpen} />
{@render children()}
