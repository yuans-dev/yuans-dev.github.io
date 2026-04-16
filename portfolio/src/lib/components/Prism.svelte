<script lang="ts">
	import { onMount } from 'svelte';

	let canvas: HTMLCanvasElement;
	let ctx: CanvasRenderingContext2D;

	let width = 0;
	let height = 0;
	let dpr = 1;

	let animationFrame: number;

	let mouse = { x: 0, y: 0 };
	let smoothedMouse = { x: 0, y: 0 };

	type Pane = {
		x: number;
		w: number;
		speed: number;
		hue: number;
		alpha: number;
		targetAlpha: number;
	};

	let panes: Pane[] = [];

	const MAX_PANES = 12;

	// -----------------------------
	// Resize (NO reset)
	// -----------------------------
	function resize() {
		dpr = window.devicePixelRatio || 1;

		width = canvas.clientWidth;
		height = canvas.clientHeight;

		canvas.width = width * dpr;
		canvas.height = height * dpr;

		ctx.setTransform(dpr, 0, 0, dpr, 0, 0);

		reconcilePaneCount();
	}

	// -----------------------------
	// Target density based on width
	// -----------------------------
	function getTargetPaneCount() {
		const base = Math.floor(width / 220);
		return Math.min(MAX_PANES, Math.max(3, base));
	}

	// -----------------------------
	// Create pane (spawn from edges)
	// -----------------------------
	function createPaneFromEdge(): Pane {
		const fromLeft = Math.random() < 0.1;

		return {
			x: fromLeft ? -300 : width + 300,
			w: 240 + Math.random() * 400,
			speed: (0.1 + Math.random() * 0.4) * (Math.random() < 0.6 ? -1 : 1),
			hue: 220 + Math.random() * 80,
			alpha: 0,
			targetAlpha: 0.08 + Math.random() * 0.12
		};
	}

	// -----------------------------
	// Gradual add/remove logic
	// -----------------------------
	function reconcilePaneCount() {
		const target = getTargetPaneCount();

		// ADD
		while (panes.length < target) {
			panes.push(createPaneFromEdge());
		}

		// REMOVE (fade out instead of instant delete)
		while (panes.length > target) {
			const p = panes.pop();
			if (p) p.targetAlpha = 0;
		}
	}

	// -----------------------------
	// Mouse input
	// -----------------------------
	function handleMouseMove(e: MouseEvent) {
		const rect = canvas.getBoundingClientRect();
		mouse.x = e.clientX - rect.left;
		mouse.y = e.clientY - rect.top;
	}

	// -----------------------------
	// Background
	// -----------------------------
	function drawBackground() {
		const g = ctx.createLinearGradient(0, 0, width, height);

		g.addColorStop(0, '#050816');
		g.addColorStop(0.5, '#0b1026');
		g.addColorStop(1, '#070415');

		ctx.fillStyle = g;
		ctx.fillRect(0, 0, width, height);
	}

	// -----------------------------
	// Light influence (cursor spotlight effect on panes)
	// -----------------------------
	function getLightBoost(x: number) {
		const dx = x - smoothedMouse.x;
		const dist = Math.abs(dx);

		const radius = 280;

		if (dist > radius) return 0;

		return Math.pow(1 - dist / radius, 2);
	}

	// -----------------------------
	// Pane rendering
	// -----------------------------
	function drawPane(p: Pane) {
		const gradient = ctx.createLinearGradient(p.x, 0, p.x + p.w, 0);

		gradient.addColorStop(0, `hsla(${p.hue}, 85%, 60%, ${p.alpha})`);
		gradient.addColorStop(0.5, `hsla(${p.hue + 15}, 90%, 65%, ${p.alpha})`);
		gradient.addColorStop(1, `hsla(${p.hue + 35}, 80%, 55%, ${p.alpha})`);

		ctx.fillStyle = gradient;
		ctx.fillRect(p.x, 0, p.w, height);
	}

	// -----------------------------
	// Pane update
	// -----------------------------
	function updatePane(p: Pane) {
		p.x += p.speed;

		// fade toward target alpha
		p.alpha += (p.targetAlpha - p.alpha) * 0.05;

		// wrap behavior
		if (p.speed > 0 && p.x > width + 300) {
			p.x = -p.w;
		}

		if (p.speed < 0 && p.x < -p.w - 300) {
			p.x = width + 300;
		}
	}

	// -----------------------------
	// Spotlight overlay
	// -----------------------------
	function drawSpotlight() {
		const radius = 260;

		const g = ctx.createRadialGradient(
			smoothedMouse.x,
			smoothedMouse.y,
			0,
			smoothedMouse.x,
			smoothedMouse.y,
			radius
		);

		g.addColorStop(0, 'rgba(120,180,255,0.12)');
		g.addColorStop(0.4, 'rgba(170,120,255,0.10)');
		g.addColorStop(1, 'rgba(0,0,0,0)');

		ctx.globalCompositeOperation = 'lighter';
		ctx.fillStyle = g;
		ctx.fillRect(0, 0, width, height);
		ctx.globalCompositeOperation = 'source-over';
	}

	// -----------------------------
	// Cleanup faded panes
	// -----------------------------
	function cleanupPanes() {
		panes = panes.filter((p) => !(p.targetAlpha === 0 && p.alpha < 0.01));
	}
	function drawPaneWithAlpha(p: Pane, alpha: number) {
		const gradient = ctx.createLinearGradient(p.x, 0, p.x + p.w, 0);

		gradient.addColorStop(0, `hsla(${p.hue}, 85%, 60%, ${alpha})`);
		gradient.addColorStop(0.5, `hsla(${p.hue + 15}, 90%, 65%, ${alpha})`);
		gradient.addColorStop(1, `hsla(${p.hue + 35}, 80%, 55%, ${alpha})`);

		ctx.fillStyle = gradient;
		ctx.fillRect(p.x, 0, p.w, height);
	}
	// -----------------------------
	// Draw loop
	// -----------------------------
	function draw() {
		ctx.clearRect(0, 0, width, height);

		for (const p of panes) {
			updatePane(p);

			const boost = getLightBoost(p.x);
			drawPaneWithAlpha(p, p.alpha + boost * 0.18);
		}

		drawSpotlight();
		cleanupPanes();
	}

	function animate() {
		smoothedMouse.x += (mouse.x - smoothedMouse.x) * 0.08;
		smoothedMouse.y += (mouse.y - smoothedMouse.y) * 0.08;

		draw();

		animationFrame = requestAnimationFrame(animate);
	}

	// -----------------------------
	// Lifecycle
	// -----------------------------
	onMount(() => {
		ctx = canvas.getContext('2d')!;

		resize();

		window.addEventListener('resize', resize);
		canvas.addEventListener('mousemove', handleMouseMove);

		animate();

		return () => {
			cancelAnimationFrame(animationFrame);
			window.removeEventListener('resize', resize);
			canvas.removeEventListener('mousemove', handleMouseMove);
		};
	});
</script>

<canvas bind:this={canvas}></canvas>

<style>
	canvas {
		width: 100%;
		height: 100%;
		display: block;
	}
</style>
