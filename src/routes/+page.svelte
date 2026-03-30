<script>
	import { onMount } from 'svelte';

	const images = import.meta.glob('$lib/images/*.{png,jpg,jpeg,webp}', {
		eager: true,
		query: { url: true }
	});
	const imagePaths = Object.values(images).map(img => img.default);

	let canvasEl;
	let tileRefs = $state(new Array(9).fill(null));
	let x = $state(0);
	let y = $state(0);
	let tileW = 0, tileH = 0;
	let ready = $state(false);
	let isDragging = $state(false);
	let lastPX = 0, lastPY = 0;
	let velX = 0, velY = 0;
	let animId = null;

	function initDimensions() {
		if (!tileRefs[0]) return;
		tileW = tileRefs[0].offsetWidth;
		tileH = tileRefs[0].offsetHeight;
		if (tileW > 0 && tileH > 0) {
			x = -tileW + (tileW - window.innerWidth) / 2;
			y = -tileH + (tileH - window.innerHeight) / 2;
			ready = true;
		}
	}

	onMount(() => {
		initDimensions();
		const imgs = tileRefs[0]?.querySelectorAll('img') ?? [];
		let pending = [...imgs].filter(img => !img.complete).length;
		if (pending === 0) { initDimensions(); return; }
		imgs.forEach(img => {
			if (!img.complete) img.addEventListener('load', () => {
				if (--pending === 0) initDimensions();
			});
		});
		return () => cancelAnimationFrame(animId);
	});

	function wrap() {
		if (!tileW || !tileH) return;
		x = ((x % tileW) + tileW) % tileW - tileW;
		y = ((y % tileH) + tileH) % tileH - tileH;
	}

	function onPointerDown(e) {
		isDragging = true;
		lastPX = e.clientX;
		lastPY = e.clientY;
		velX = 0; velY = 0;
		cancelAnimationFrame(animId);
		e.currentTarget.setPointerCapture(e.pointerId);
	}

	function onPointerMove(e) {
		if (!isDragging || !ready) return;
		const dx = e.clientX - lastPX;
		const dy = e.clientY - lastPY;
		velX = velX * 0.2 + dx * 0.2;
		velY = velY * 0.2 + dy * 0.2;
		x += dx; y += dy;
		wrap();
		lastPX = e.clientX;
		lastPY = e.clientY;
	}

	function onPointerUp() {
		if (!isDragging) return;
		isDragging = false;
		(function inertia() {
			if (Math.abs(velX) < 0.3 && Math.abs(velY) < 0.3) return;
			velX *= 0.95; velY *= 0.95;
			x += velX; y += velY;
			wrap();
			animId = requestAnimationFrame(inertia);
		})();
	}
</script>

<svelte:head>
	<style> body { overflow: hidden; } </style>
</svelte:head>

<!-- Infinite canvas -->
<div
	role="presentation"
	class="fixed inset-0 overflow-hidden touch-none select-none
	       {isDragging ? 'cursor-grabbing' : 'cursor-grab'}
	       {ready ? 'opacity-100' : 'opacity-0'}"
	bind:this={canvasEl}
	onpointerdown={onPointerDown}
	onpointermove={onPointerMove}
	onpointerup={onPointerUp}
	onpointercancel={onPointerUp}
	oncontextmenu={(e) => e.preventDefault()}
>
	<div
		class="absolute top-0 left-0 grid will-change-transform"
		style="grid-template-columns: repeat(3, 150vw); transform: translate3d({x}px, {y}px, 0);"
	>
		{#each Array(9) as _, i}
			<div
				bind:this={tileRefs[i]}
				class="w-[150vw] columns-3 md:columns-4 lg:columns-7 gap-6 p-3"
			>
				{#each imagePaths as path}
					<div class="mb-6 break-inside-avoid">
						<img
							src={path}
							alt="Gallery piece"
							class="w-full rounded-lg shadow-md pointer-events-none"
							draggable="false"
						/>
					</div>
				{/each}
			</div>
		{/each}
	</div>
</div>