<script>
	import { Sun, Moon } from '@lucide/svelte';
	import { onMount } from 'svelte';

	let isDark = $state(false);

	onMount(() => {
		// Check localStorage or system preference on load
		const savedTheme = localStorage.getItem('theme');
		const systemPrefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;

		isDark = savedTheme === 'dark' || (!savedTheme && systemPrefersDark);

		// Apply the correct class immediately
		if (isDark) {
			document.documentElement.classList.add('dark');
		} else {
			document.documentElement.classList.remove('dark');
		}
	});

	function toggleTheme() {
		isDark = !isDark;
		if (isDark) {
			document.documentElement.classList.add('dark');
			document.documentElement.classList.remove('light');
			localStorage.setItem('theme', 'dark');
		} else {
			document.documentElement.classList.remove('dark');
			document.documentElement.classList.add('light');
			localStorage.setItem('theme', 'light');
		}
	}
</script>

<button
	onclick={toggleTheme}
	class="fixed top-4 right-4 z-50
           flex items-center justify-center p-3
           rounded-full cursor-pointer transition-all duration-300
           bg-white/50 dark:bg-black/50 backdrop-blur-sm
           border border-gray-500
           shadow-xl shadow-black/30
           hover:border-white/50 active:scale-95
           text-black/70 dark:text-white/70"
	aria-label="Toggle Theme"
>
	{#if isDark}
		<div class="flex items-center gap-2">
			<Moon class="w-5 h-5" />
			<span class="text-xs font-medium hidden md:block">Dark</span>
		</div>
	{:else}
		<div class="flex items-center gap-2">
			<Sun class="w-5 h-5" />
			<span class="text-xs font-medium hidden md:block">Light</span>
		</div>
	{/if}
</button>