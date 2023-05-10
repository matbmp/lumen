<script lang="ts">
	import { onMount } from 'svelte';
	import { converter, formatHex } from 'culori';
	let rgbConverter = converter('rgb');
	let hslConverter = converter('hsl');

	export let hex: string = '#ff0000';
	const leftButton = 1;
	let bigWdith: number;
	let bigHeight: number;
	let bigFocus: boolean = false;
	let hueFocus: boolean = false;
	let bigElement: HTMLElement;
	let hueElement: HTMLElement;

	let hsl = hslConverter(hex) ?? { mode: 'hsl', h: 1, s: 1, l: 0.5 };
	let h: number;
	let rgb = rgbConverter(hsl);

	let sampleStyle: string;
	let bigStyle: string;
	let bigSliderStyle: string;
	let hueSliderStyle: string;

	$: rgb = rgbConverter(hsl);
	$: hex = formatHex(rgb);

	function clamp(min: number, value: number, max: number) {
		return Math.max(Math.min(value, max), min);
	}

	function mouseMoved(event: MouseEvent) {
		if (event.buttons === leftButton) {
			if (bigFocus) {
				let rect = bigElement.getBoundingClientRect();
				let dx = event.clientX - rect.left;
				let dy = event.clientY - rect.top;
				const px = clamp(0, dx / bigWdith, 1);
				const py = clamp(0, dy / bigHeight, 1);

				hsl.s = px;
				hsl.l = 1 - py / (1 + px) - px / 2;
			}
			if (hueFocus) {
				let rect = hueElement.getBoundingClientRect();
				let dx = event.clientX - rect.left;
				const px = clamp(0, dx / bigWdith, 1);
				hsl.h = px * 360;
			}
		} else {
			bigFocus = hueFocus = false;
		}
	}

	onMount(() => {
		document.addEventListener('mousemove', mouseMoved);
	});
</script>

<div class="w-96 bg-slate-900 p-4 rounded-lg relative">
	<div class="h-20 mb-2" style:background={hex} />

	<div
		class="relative h-40 window-saturation border border-gray rounded shadow window-big"
		style={`
            background: hsl(${hsl.h} 100% 50%);
            background-image: linear-gradient(transparent, black),
			    linear-gradient(to right, white, transparent);
        `}
		on:mousedown={(e) => {
			if (!hueFocus && e.buttons === leftButton) bigFocus = true;
		}}
		bind:clientWidth={bigWdith}
		bind:clientHeight={bigHeight}
		bind:this={bigElement}
	>
		<div
			class="absolute h-6 w-6 rounded-2xl border-4 border-white pointer-events-none"
			style:top={`calc(${(1 + hsl.s) * (1 - hsl.l - hsl.s / 2) * bigHeight}px - 0.75rem)`}
			style:left={`calc(${hsl.s * bigWdith}px - 0.75rem)`}
		/>
	</div>

	<div
		class="relative w-full h-4 rounded-xl window-hue my-2 border border-gray"
		on:mousedown={(e) => {
			if (!bigFocus && e.buttons === leftButton) hueFocus = true;
		}}
		bind:this={hueElement}
	>
		<div
			class="absolute h-6 w-6 -top-1 rounded-2xl border-4 border-white pointer-events-none"
			style:left={`calc(${((hsl.h || 0) / 360) * bigWdith}px - 0.75rem)`}
			style:background={`hsl(${hsl.h} 100% 50%)`}
		/>
	</div>

	<div class="grid grid-cols-3 h-10 gap-2 text-white mb-2">
		<div class="border-2 border-slate-300 rounded flex items-center justify-center">
			{hex}
		</div>
		<div class="border-2 border-slate-300 rounded flex items-center justify-center">
			{(rgb.r * 255).toFixed(0)}
			{(rgb.g * 255).toFixed(0)}
			{(rgb.b * 255).toFixed(0)}
		</div>
		<div class="border-2 border-slate-300 rounded flex items-center justify-center">
			{(hsl.h || 0).toFixed(0)}
			{(hsl.s * 100).toFixed(0)}
			{(hsl.l * 100).toFixed(0)}
		</div>
		<div class="flex justify-center">HEX</div>
		<div class="flex justify-center">RGB</div>
		<div class="flex justify-center">HSL</div>
	</div>
</div>

<style>
	.window-hue {
		background-image: linear-gradient(
			to right,
			#ff0000,
			#ffff00,
			#00ff00,
			#00ffff,
			#0000ff,
			#ff00ff,
			#ff0000
		);
	}
</style>
