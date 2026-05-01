<script>
	import { onMount } from 'svelte';

	const images = import.meta.glob('$lib/images/*.{png,jpg,jpeg,webp}', {
		eager: true,
		query: {
			w: '400;800;1200',
			format: 'webp',
			as: 'picture'
		}
	});

	const rawImages = Object.values(images).map(img => img.default);

	const TILE_VW = 1.5;
	const MIN_IMGS = 80;

	let x = $state(0);
	let y = $state(0);
	let tileW = $state(0);
	let tileH = $state(0);
	let colWidths = $state([]);
	let ready = $state(false);
	let isDragging = $state(false);
	let columns = $state([]);
	let gap = $state(12);
	let pad = $state(6);

	// ─── Curve state ──────────────────────────────────────────────────────────
	// curveX: horizontal drag velocity → rotateY on each row
	// curveY: vertical drag velocity → rotateX on each column
	let curveX = $state(0); // driven by velX
	let curveY = $state(0); // driven by velY

	const CURVE_SCALE = 0.04;    // how much velocity maps to degrees
	const CURVE_MAX   = 18;      // max degrees of bend
	const CURVE_DECAY = 0.88;    // how fast it springs back to flat

	let visibleTiles = $derived(computeVisible(x, y, tileW, tileH));

	function computeVisible(cx, cy, tw, th) {
		if (!tw || !th) return new Set([4]);
		const vw = window?.innerWidth ?? 0;
		const vh = window?.innerHeight ?? 0;
		const buf = Math.max(vw, vh) * 0.25;
		const set = new Set();
		for (let i = 0; i < 9; i++) {
			const tx = cx + (i % 3) * tw;
			const ty = cy + Math.floor(i / 3) * th;
			if (tx + tw > -buf && tx < vw + buf && ty + th > -buf && ty < vh + buf) set.add(i);
		}
		return set;
	}

	let lastPX = 0, lastPY = 0;
	let velX = 0, velY = 0;
	let animId = null;

	function getResponsiveConfig(vw) {
		if (vw >= 1280) return { cols: 7, gap: 16, pad: 8 };
		if (vw >= 1024) return { cols: 6, gap: 14, pad: 7 };
		if (vw >= 768)  return { cols: 5, gap: 12, pad: 6 };
		if (vw >= 480)  return { cols: 4, gap: 10, pad: 5 };
		return { cols: 3, gap:  8, pad:  4 };
	}

	function init() {
		const vw = window.innerWidth;
		const vh = window.innerHeight;
		const { cols, gap: g, pad: p } = getResponsiveConfig(vw);

		const newTileW = Math.round(vw * TILE_VW);
		const availW = newTileW - p * 2 - g * (cols - 1);
		const colW = availW / cols;

		const allImgs = Array.from(
			{ length: Math.max(MIN_IMGS, rawImages.length) },
			(_, i) => rawImages[i % rawImages.length]
		);

		const cols_ = Array.from({ length: cols }, () => []);
		const colHeights = Array(cols).fill(0);

		for (const imgData of allImgs) {
			const shortestIdx = colHeights.indexOf(Math.min(...colHeights));
			const meta = imgData.img;
			const ratio = meta.h / meta.w;
			const imgHeight = colW * ratio;
			cols_[shortestIdx].push(imgData);
			colHeights[shortestIdx] += imgHeight + g;
		}

		gap = g;
		pad = p;
		tileW = newTileW;
		tileH = Math.ceil(Math.max(...colHeights)) + p;
		colWidths = Array(cols).fill(colW);
		columns = cols_;

		if (!ready) {
			x = -tileW + (tileW - vw) / 2;
			y = -tileH + (tileH - vh) / 2;
			ready = true;
		}
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

	function clamp(v, min, max) {
		return Math.max(min, Math.min(max, v));
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

		// Update live curve while dragging
		curveX = clamp(-velX * CURVE_SCALE, -CURVE_MAX, CURVE_MAX);
		curveY = clamp(-velY * CURVE_SCALE, -CURVE_MAX, CURVE_MAX);

		x += dx; y += dy;
		wrap();
		lastPX = e.clientX; lastPY = e.clientY;
	}

	function onPointerUp() {
		if (!isDragging) return;
		isDragging = false;
		(function inertia() {
			velX *= 0.92; velY *= 0.92;

			// Decay curve back to flat
			curveX *= CURVE_DECAY;
			curveY *= CURVE_DECAY;

			if (Math.abs(curveX) < 0.01) curveX = 0;
			if (Math.abs(curveY) < 0.01) curveY = 0;

			if (Math.abs(velX) < 0.4 && Math.abs(velY) < 0.4 && curveX === 0 && curveY === 0) return;

			x += velX; y += velY;
			wrap();
			animId = requestAnimationFrame(inertia);
		})();
	}

	// ─── Per-item curve transform ─────────────────────────────────────────────
	// Each image gets a rotateY proportional to its column position (for X drag)
	// and a rotateX proportional to its row position (for Y drag).
	// The result: the whole grid bends like a flexible sheet in the drag direction.
	function getItemTransform(
		colIdx,
		totalCols,
	) {
		// Normalized position: -1 (left edge) to +1 (right edge)
		const normCol = totalCols > 1 ? (colIdx / (totalCols - 1)) * 2 - 1 : 0;

		// rotateY bends the grid horizontally (driven by horizontal drag)
		const rotY = normCol * curveX;

		// Z pushback makes edges recede, reinforcing the cylinder feel
		const tz = -(normCol * normCol) * Math.abs(curveX) * 2.5;

		return `perspective(900px) rotateY(${rotY}deg) translateZ(${tz}px)`;
	}

	function getRowTransform(
		rowPct, // 0 (top) to 1 (bottom) within the column
	) {
		// Normalized: -1 (top) to +1 (bottom)
		const normRow = rowPct * 2 - 1;
		const rotX = normRow * curveY;
		const tz = -(normRow * normRow) * Math.abs(curveY) * 2.5;
		return `perspective(900px) rotateX(${rotX}deg) translateZ(${tz}px)`;
	}
</script>

<div
	role="presentation"
	class="fixed inset-0 bg-background overflow-hidden touch-none select-none transition-opacity duration-500
	       {isDragging ? 'cursor-grabbing' : 'cursor-grab'}
	       {ready ? 'opacity-100' : 'opacity-0'}"
	onpointerdown={onPointerDown}
	onpointermove={onPointerMove}
	onpointerup={onPointerUp}
	onpointercancel={onPointerUp}
	style="perspective: 1400px; transform-style: preserve-3d;"
>
	<div
		class="absolute top-0 left-0"
		style="
			display: grid;
			grid-template-columns: repeat(3, {tileW}px);
			grid-template-rows: repeat(3, {tileH}px);
			transform: translate3d({x}px, {y}px, 0);
			will-change: transform;
			transform-style: preserve-3d;
		"
	>
		{#each Array(9) as _, tileIdx}
			{#if visibleTiles.has(tileIdx)}
				<div
					style="
						width: {tileW}px; height: {tileH}px; padding: {pad}px;
						display: grid;
						grid-template-columns: {colWidths.map(w => w + 'px').join(' ')};
						gap: {gap}px;
						contain: layout paint;
						overflow: hidden;
						transform-style: preserve-3d;
					"
				>
					{#each columns as col, colIdx}
						<div
							style="
								display: flex;
								flex-direction: column;
								gap: {gap}px;
								transform: {getItemTransform(colIdx, columns.length)};
								transform-origin: center center;
							"
						>
							{#each col as imgData, imgIdx}
								<picture
									style="
										transform: {getRowTransform((imgIdx + 0.5) / col.length)};
										transform-origin: center center;
										display: block;
									"
								>
									{#each Object.entries(imgData.sources) as [format, srcset]}
										<source {srcset} type="image/{format}" />
									{/each}
									<img
										src={imgData.img.src}
										alt=""
										loading="lazy"
										decoding="async"
										draggable="false"
										class="w-full rounded-lg pointer-events-none block"
										style="aspect-ratio: {imgData.img.w} / {imgData.img.h}; height: auto;"
									/>
								</picture>
							{/each}
						</div>
					{/each}
				</div>
			{:else}
				<div style="width: {tileW}px; height: {tileH}px;"></div>
			{/if}
		{/each}
	</div>
</div>