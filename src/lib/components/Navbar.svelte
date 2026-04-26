<script>
	import { ShoppingBag, Send, PanelsLeftBottom, X } from '@lucide/svelte';
	import { resolve } from '$app/paths';
	import { page } from '$app/stores';

	let { contactOpen = $bindable(false), navVisible = $bindable(true) } = $props();

	function openContact() {
		navVisible = false;
		setTimeout(() => { contactOpen = true; }, 200);
	}

	function closeContact() {
		contactOpen = false;
		setTimeout(() => { navVisible = true; }, 200);
	}

	function handleBackdropClick(e) {
		if (e.target === e.currentTarget) closeContact();
	}

	function handleKeydown(e) {
		if (e.key === 'Escape') closeContact();
	}

	// --- Sliding pill ---
	let navEl = $state(null);
	let pillStyle = $state('');

	function movePill(el) {
		if (!el || !navEl) return;
		const navRect = navEl.getBoundingClientRect();
		const elRect = el.getBoundingClientRect();
		const left = elRect.left - navRect.left;
		const width = elRect.width;
		const height = elRect.height;
		const top = elRect.top - navRect.top;
		pillStyle = `left:${left}px; width:${width}px; height:${height}px; top:${top}px; opacity:1;`;
	}

	function clearPill() {
		pillStyle = pillStyle.replace('opacity:1', 'opacity:0');
	}

	// Move pill to correct element on route change or modal open/close
	let homeEl = $state(null);
	let shopEl = $state(null);
	let contactEl = $state(null);

	$effect(() => {
		if (contactOpen && contactEl) {
			movePill(contactEl);
		} else if ($page.url.pathname === resolve('/shop') && shopEl) {
			movePill(shopEl);
		} else if ($page.url.pathname === resolve('/') && homeEl) {
			movePill(homeEl);
		} else {
			clearPill();
		}
	});
</script>

<svelte:window onkeydown={handleKeydown} />

<!-- Contact Modal -->
{#if contactOpen}
	<div
		class="fixed inset-0 z-40 dark:bg-black/20 bg-black/40 backdrop-blur-[2px] flex items-center"
		role="button"
		tabindex="-1"
		aria-label="Close contact"
		onclick={handleBackdropClick}
		onkeydown={handleKeydown}
	>
		<div
			class="absolute left-1/2 text-center
                   flex flex-col gap-4 p-12
                   rounded-3xl
                   dark:bg-white/50 bg-black/50 backdrop-blur-sm
                   border border-gray-500
                   shadow-xl shadow-black/30
                   dark:text-black/80 text-white/80
                   w-[min(360px,90vw)]
                   modal-enter"
			role="dialog"
			aria-modal="true"
			aria-label="Contact"
		>
			<button
				onclick={closeContact}
				class="absolute top-3 right-3 p-1 rounded-full
                       dark:text-black/40 text-white/40
                       dark:hover:text-black hover:text-white
                       dark:hover:bg-black/10 hover:bg-white/10
                       active:scale-95 transition-all duration-150"
				aria-label="Close"
			>
				<X class="w-4 h-4" />
			</button>

			<div>
				<p class="font-bold text-lg leading-tight">Mohamed Amine Testouri</p>
				<p class="dark:text-black/50 text-white/50 mt-0.5">A rising North African designer</p>
			</div>

			<div class="w-full h-px bg-white/70 dark:bg-black/70"></div>

			<div class="flex flex-col gap-2">
				<a href="https://www.instagram.com/mohamedaminetestouriii/" target="_blank" rel="noopener noreferrer"
				   class="flex items-center gap-3 rounded-full px-3 py-2 -mx-1 transition-all duration-150
					dark:hover:bg-black/10 hover:bg-white/10 dark:hover:text-black hover:text-white
					active:scale-95 dark:active:bg-black/20 active:bg-white/20 justify-center">
					<span class="text-sm font-medium">Instagram</span>
				</a>
				<a href="https://www.linkedin.com/in/testouri-mohamed-amine-320222345/" target="_blank" rel="noopener noreferrer"
				   class="flex items-center gap-3 rounded-full px-3 py-2 -mx-1 transition-all duration-150
					dark:hover:bg-black/10 hover:bg-white/10 dark:hover:text-black hover:text-white
					active:scale-95 dark:active:bg-black/20 active:bg-white/20 justify-center">
					<span class="text-sm font-medium">LinkedIn</span>
				</a>
				<a href="https://www.behance.net/tisstisgraphic" target="_blank" rel="noopener noreferrer"
				   class="flex items-center gap-3 rounded-full px-3 py-2 -mx-1 transition-all duration-150
					dark:hover:bg-black/10 hover:bg-white/10 dark:hover:text-black hover:text-white
					active:scale-95 dark:active:bg-black/20 active:bg-white/20 justify-center">
					<span class="text-sm font-medium">Behance</span>
				</a>
				<a href="https://wa.me/+21697494493" target="_blank" rel="noopener noreferrer"
				   class="flex items-center gap-3 rounded-full px-3 py-2 -mx-1 transition-all duration-150
					dark:hover:bg-black/10 hover:bg-white/10 dark:hover:text-black hover:text-white
					active:scale-95 dark:active:bg-black/20 active:bg-white/20 justify-center">
					<span class="text-sm font-medium">WhatsApp</span>
				</a>
			</div>
		</div>
	</div>
{/if}

<!-- Navbar -->
<nav
	bind:this={navEl}
	class="fixed bottom-4 left-1/2 z-50
            flex gap-5 items-center px-6 py-2
            rounded-full pointer-events-auto
            dark:bg-white/50 bg-black/50 backdrop-blur-sm
            border border-gray-500
            shadow-lg shadow-black/30 dark:shadow-white/10
            dark:text-black/70 text-white/70 text-sm
            transition-all duration-300 ease-in-out"
	style="transform: translateX(-50%) translateY({navVisible ? '0' : '80px'}); opacity: {navVisible ? '1' : '0'};"
>
	<!-- Sliding pill background -->
	<div
		class="absolute rounded-full pointer-events-none
               dark:bg-black/60 bg-white/20
               transition-all duration-300 ease-[cubic-bezier(0.16,1,0.3,1)]"
		style={pillStyle}
	></div>

	<h1 class="font-bold relative z-10">moamine&bull;gallery</h1>
	<div class="w-px h-5 bg-white/70 dark:bg-black/70 relative z-10"></div>

	<a
	bind:this={homeEl}
	href={resolve('/')}
	class="flex gap-2 items-center rounded-full py-1 px-2
	relative z-10 transition-colors duration-150
	dark:hover:text-black hover:text-white
	{$page.url.pathname === resolve('/') ? 'bg-black/60 text-white dark:text-white' : ''}"
	>
	<PanelsLeftBottom class="w-5 h-5" />
	<p class="hidden sm:block">Home</p>
	</a>

<a
	bind:this={shopEl}
	href={resolve('/shop')}
	class="flex gap-2 items-center rounded-full py-1 px-2
	relative z-10 transition-colors duration-150
	dark:hover:text-black hover:text-white
	{$page.url.pathname === resolve('/shop') ? 'bg-black/60 text-white dark:text-white' : ''}"
	>
	<ShoppingBag class="w-5 h-5" />
	<p class="hidden sm:block">Shop</p>
	</a>

	<button
		bind:this={contactEl}
		onclick={openContact}
		class="flex gap-2 items-center rounded-full py-1 px-2
               relative z-10 transition-colors duration-150
               dark:hover:text-black hover:text-white"
	>
		<Send class="w-5 h-5" />
		<p class="hidden sm:block">Contact</p>
	</button>
</nav>

<style>
    @keyframes modal-enter {
        from { opacity: 0; transform: translateX(-50%) translateY(40px); }
        to   { opacity: 1; transform: translateX(-50%) translateY(0); }
    }
    .modal-enter {
        animation: modal-enter 0.25s cubic-bezier(0.16, 1, 0.3, 1) both;
    }
</style>