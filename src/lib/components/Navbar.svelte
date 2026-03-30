<script>
	import { ShoppingBag, Send, PanelsLeftBottom } from '@lucide/svelte';
</script>

<nav class="fixed bottom-4 left-1/2 -translate-x-1/2 z-50
            flex gap-5 items-center px-6 py-2
            rounded-full pointer-events-auto
            bg-black/50 backdrop-blur-sm
            border border-gray-500
            shadow-xl shadow-black/30
            text-white/70 text-sm">

	<h1 class="font-bold">moamine&bull;gallery</h1>|
	<div class="flex gap-2 items-center bg-black/60 transition-colors hover:text-black rounded-full py-1 px-2">
		<PanelsLeftBottom class="w-5 h-5 cursor-pointer" />
		<p class="hidden sm:block">Home</p>
	</div>
	<div class="flex gap-2 items-center transition-colors hover:text-black">
		<Send class="w-5 h-5 cursor-pointer" />
		<p class="hidden sm:block">Contact</p>
	</div>
	<div class="flex gap-2 items-center transition-colors hover:text-black">
		<ShoppingBag class="w-5 h-5 cursor-pointer" />
		<p class="hidden sm:block">Shop</p>
	</div>
</nav>