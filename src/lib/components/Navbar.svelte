<script>
	import { ShoppingBag, Send, PanelsLeftBottom } from '@lucide/svelte';
	import { resolve } from '$app/paths';
	import { page } from '$app/stores';
</script>

<nav class="fixed bottom-4 left-1/2 -translate-x-1/2 z-50
            flex gap-5 items-center px-6 py-2
            rounded-full pointer-events-auto
            dark:bg-white/50 bg-black/50 backdrop-blur-sm
            border border-gray-500
            shadow-lg shadow-black/30 dark:shadow-white/10
            dark:text-black/70 text-white/70 text-sm">

	<h1 class="font-bold">moamine&bull;gallery</h1>
	<div class="w-px h-5 bg-white/70 dark:bg-black/70"></div>

<a
	href={resolve('/')}
	class="flex gap-2 items-center rounded-full py-1 px-2 transition-all duration-150
	dark:hover:text-black hover:text-white dark:hover:bg-black/10 hover:bg-white/10
	active:scale-95 dark:active:bg-black/20 active:bg-white/20
	{$page.url.pathname === resolve('/') ? 'bg-black/60 text-white dark:text-white' : ''}"
	>
	<PanelsLeftBottom class="w-5 h-5" />
	<p class="hidden sm:block">Home</p>
	</a>

<a
	href={resolve('/contact')}
	class="flex gap-2 items-center rounded-full py-1 px-2 transition-all duration-150
	dark:hover:text-black hover:text-white dark:hover:bg-black/10 hover:bg-white/10
	active:scale-95 dark:active:bg-black/20 active:bg-white/20
	{$page.url.pathname === resolve('/contact') ? 'bg-black/60 text-white dark:text-white' : ''}"
	>
	<Send class="w-5 h-5" />
	<p class="hidden sm:block">Contact</p>
	</a>

<a
	href={resolve('/shop')}
	class="flex gap-2 items-center rounded-full py-1 px-2 transition-all duration-150
	dark:hover:text-black hover:text-white dark:hover:bg-black/10 hover:bg-white/10
	active:scale-95 dark:active:bg-black/20 active:bg-white/20
	{$page.url.pathname === resolve('/shop') ? 'bg-black/60 text-white dark:text-white' : ''}"
	>
	<ShoppingBag class="w-5 h-5" />
	<p class="hidden sm:block">Shop</p>
	</a>
</nav>