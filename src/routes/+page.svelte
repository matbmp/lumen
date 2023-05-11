<script lang="ts">
	import ColorPicker from './ColorPicker.svelte';
	import { Tabs, TabItem, Button, Toast } from 'flowbite-svelte';

	let maxColors = 8;
	let editorHidden = true;
	let toastOpen = false;
	let colors: string[] = ['#ff0000', '#0000ff'];
	let gradient: string;

	$: gradient = `linear-gradient(to right ${colors.reduce(
		(partial, value) => partial + `, ${value}`,
		''
	)})`;
</script>

<div class="w-full relative" style:background-image={gradient} style:height="100svh">
	<div class="w-full absolute bottom-0">
		<div class="mx-auto flex flex-col w-min">
			<div class="bg-slate-900" style:display={editorHidden ? 'none' : ''}>
				<div class="p-2">
					<Button
						color="alternative"
						on:click={() => {
							if (colors.length < maxColors) colors = [...colors, '#ff0000'];
						}}>Add color</Button
					>
				</div>

				<Tabs defaultClass="flex bg-slate-900" contentClass="bg-slate-900">
					{#each colors as color, i}
						<TabItem title={(i + 1).toLocaleString()} open={i === 0}>
							<div class="p-2">
								<Button
									disabled={colors.length <= 2}
									color="red"
									on:click={() => {
										colors = [...colors.slice(0, i), ...colors.slice(i + 1)];
									}}>Remove</Button
								>
							</div>
							<ColorPicker bind:hex={color} />
						</TabItem>
					{/each}
				</Tabs>
				<button
					class="w-full flex items-center bg-white border-2 border-rose-500 h-20 p-2"
					on:click={() => {
						navigator.clipboard.writeText(gradient);
						toastOpen = true;
					}}
				>
					{gradient}
				</button>
			</div>

			<Button
				color="alternative"
				on:click={() => {
					editorHidden = !editorHidden;
				}}>{editorHidden ? 'Open' : 'Close'}</Button
			>
		</div>
	</div>
</div>
<div class="relative h-100">
	<Toast position="bottom-right" bind:open={toastOpen}>
		<div class="text-slate-900">
			Copied: <span class="text-amber-800">{gradient}</span> to clipboard
		</div>
	</Toast>
</div>
