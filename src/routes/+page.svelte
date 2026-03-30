<script>
	import { onMount } from 'svelte';

	const images = import.meta.glob('$lib/images/*.{png,jpg,jpeg,webp}', {
		eager: true,
		query: { url: true }
	});
	const rawPaths = Object.values(images).map(img => img.default);

	const TILE_VW  = 1.5;
	const MIN_IMGS = 80;

	// ─── Responsive config based on device width ──────────────────────────────
	function getResponsiveConfig(vw) {
		if (vw >= 1280) return { cols: 7, gap: 16, pad: 16 };
		if (vw >= 1024) return { cols: 6, gap: 14, pad: 14 };
		if (vw >= 768)  return { cols: 5, gap: 12, pad: 12 };
		if (vw >= 480)  return { cols: 4, gap: 10, pad: 10 };
		return { cols: 3, gap:  8, pad:  8 };
	}

	// ─── Reactive state ───────────────────────────────────────────────────────
	let x          = $state(0);
	let y          = $state(0);
	let tileW      = $state(0);
	let tileH      = $state(0);
	let colWidths  = $state([]);
	let ready      = $state(false);
	let isDragging = $state(false);
	let columns    = $state([]);
	let gap        = $state(12);
	let pad        = $state(12);

	let visibleTiles = $derived(computeVisible(x, y, tileW, tileH));

	function computeVisible(cx, cy, tw, th) {
		if (!tw || !th) return new Set([4]);
		const vw  = window?.innerWidth  ?? 0;
		const vh  = window?.innerHeight ?? 0;
		const buf = Math.max(vw, vh);
		const set = new Set();
		for (let i = 0; i < 9; i++) {
			const tx = cx + (i % 3) * tw;
			const ty = cy + Math.floor(i / 3) * th;
			if (tx + tw > -buf && tx < vw + buf &&
				ty + th > -buf && ty < vh + buf) set.add(i);
		}
		return set;
	}

	let lastPX = 0, lastPY = 0;
	let velX = 0, velY = 0;
	let animId = null;

	// Cache image ratios so resize doesn't re-fetch
	let ratioCache = new Map();

	async function loadRatios() {
		const uncached = rawPaths.filter(src => !ratioCache.has(src));
		await Promise.all(uncached.map(src => new Promise(resolve => {
			const img = new Image();
			img.onload  = () => { ratioCache.set(src, img.naturalHeight / img.naturalWidth); resolve(); };
			img.onerror = () => { ratioCache.set(src, 4 / 3); resolve(); };
			img.src = src;
		})));
	}

	async function init() {
		await loadRatios();

		const vw = window.innerWidth;
		const vh = window.innerHeight;
		const { cols, gap: g, pad: p } = getResponsiveConfig(vw);

		const newTileW = Math.round(vw * TILE_VW);
		const availW   = newTileW - p * 2 - g * (cols - 1);
		const colW     = availW / cols;
		const newColWidths = Array(cols).fill(colW);

		// Repeat images to fill columns densely
		const allPaths = Array.from(
			{ length: Math.max(MIN_IMGS, rawPaths.length) },
			(_, i) => rawPaths[i % rawPaths.length]
		);

		// Round-robin distribution
		const cols_ = Array.from({ length: cols }, (_, col) =>
			allPaths.filter((_, i) => i % cols === col)
		);

		// Exact column height using real ratios
		const colHeight = (col) =>
			col.reduce((h, src) => h + colW * (ratioCache.get(src) ?? 4 / 3) + g, -g) + p * 2;

		const newTileH = Math.ceil(Math.max(...cols_.map(colHeight)));

		// Apply all state at once to avoid intermediate renders
		gap      = g;
		pad      = p;
		tileW    = newTileW;
		tileH    = newTileH;
		colWidths = newColWidths;
		columns  = cols_;

		x = -tileW + (tileW - vw) / 2;
		y = -tileH + (tileH - vh) / 2;
		ready = true;
	}

	onMount(() => {
		init();
		window.addEventListener('resize', init);
		return () => {
			window.removeEventListener('resize', init);
			cancelAnimationFrame(animId);
		};
	});

	function wrap() {
		if (!tileW || !tileH) return;
		x = ((x % tileW) + tileW) % tileW - tileW;
		y = ((y % tileH) + tileH) % tileH - tileH;
	}

	function onPointerDown(e) {
		isDragging = true;
		lastPX = e.clientX; lastPY = e.clientY;
		velX = 0; velY = 0;
		cancelAnimationFrame(animId);
		e.currentTarget.setPointerCapture(e.pointerId);
	}

	function onPointerMove(e) {
		if (!isDragging || !ready) return;
		const dx = e.clientX - lastPX;
		const dy = e.clientY - lastPY;
		velX = velX * 0.4 + dx * 0.6;
		velY = velY * 0.4 + dy * 0.6;
		x += dx; y += dy;
		wrap();
		lastPX = e.clientX; lastPY = e.clientY;
	}

	function onPointerUp() {
		if (!isDragging) return;
		isDragging = false;
		(function inertia() {
			if (Math.abs(velX) < 0.4 && Math.abs(velY) < 0.4) return;
			velX *= 0.92; velY *= 0.92;
			x += velX; y += velY;
			wrap();
			animId = requestAnimationFrame(inertia);
		})();
	}
</script>

<svelte:head>
	<style>body { overflow: hidden; margin: 0; }</style>
</svelte:head>

<div
	role="presentation"
	class="fixed bg-off-black inset-0 overflow-hidden touch-none select-none transition-opacity duration-300
	       {isDragging ? 'cursor-grabbing' : 'cursor-grab'}
	       {ready ? 'opacity-100' : 'opacity-0'}"
	onpointerdown={onPointerDown}
	onpointermove={onPointerMove}
	onpointerup={onPointerUp}
	onpointercancel={onPointerUp}
	oncontextmenu={(e) => e.preventDefault()}
>
	<div
		class="absolute top-0 left-0 will-change-transform"
		style="
			display: grid;
			grid-template-columns: repeat(3, {tileW}px);
			grid-template-rows: repeat(3, {tileH}px);
			transform: translate3d({x}px, {y}px, 0);
		"
	>
		{#each Array(9) as _, tileIdx}
			<div
				style="
					width: {tileW}px;
					height: {tileH}px;
					overflow: hidden;
					box-sizing: border-box;
					padding: {pad}px;
					display: {visibleTiles.has(tileIdx) ? 'grid' : 'none'};
					grid-template-columns: {colWidths.map(w => w + 'px').join(' ')};
					gap: {gap}px;
				"
			>
				{#each columns as col, colIdx}
					<div style="
						display: flex;
						flex-direction: column;
						height: {tileH - pad * 2}px;
						justify-content: space-between;
					">
						{#each col as path, imgIdx}
							<img
								src={path}
								alt=""
								loading={tileIdx === 4 && imgIdx < columns.length * 2 ? 'eager' : 'lazy'}
								decoding="async"
								fetchpriority={tileIdx === 4 && imgIdx < columns.length ? 'high' : 'auto'}
								draggable="false"
								class="w-full rounded-lg pointer-events-none block"
							/>
						{/each}
					</div>
				{/each}
			</div>
		{/each}
	</div>
</div>