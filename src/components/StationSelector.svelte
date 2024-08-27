<script lang="ts">
	import type { VideoInfo } from '$lib/types';
	import videoInfoString from '../data/videoUrls.json';
	import { createEventDispatcher, onMount } from 'svelte';

	const dispatch = createEventDispatcher();
	const videoInfos: VideoInfo[] = videoInfoString;

	function handleStationClick(videoInfo: VideoInfo) {
		dispatch('stationClick', { videoInfo });
	}

	onMount(() => {
		window.addEventListener('keydown', handleKeyDown);
	});

	function handleKeyDown(event: KeyboardEvent) {
		if (event.key == 'Escape') {
			dispatch('stationEsc');
		}
	}
</script>

<div
	class="fixed top-0 left-0 overflow-x-hidden overflow-y-scroll bg-black bg-opacity-20 z-40 w-dvw h-dvh p-10 backdrop-blur-sm"
>
	<div class="grid stationSelectorGrid">
		{#each videoInfos as videoInfo}
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<div
				class="block w-full p-4 cursor-default"
				on:click={() => handleStationClick(videoInfo)}
				role="button"
				tabindex="0"
			>
				<img
					class="cursor-pointer"
					src="https://img.youtube.com/vi/{videoInfo.id}/maxresdefault.jpg"
					alt=""
				/>
			</div>
		{/each}
	</div>
	<div class="absolute text-white top-0 right-0 m-3">
		<button
			class="cursor-pointer"
			on:click={() => {
				dispatch('stationEsc');
			}}><i class="fa-solid fa-xmark fa-lg shadow-icon"></i></button
		>
	</div>
</div>

<style>
	.shadow-icon {
		filter: drop-shadow(0px 0px 2px var(--main-color)) drop-shadow(0px 0px 8px var(--main-color));
	}

	.stationSelectorGrid {
		grid-template-columns: repeat(auto-fit, minmax(480px, 1fr));
		grid-template-rows: min-content;
	}

	img:hover {
		filter: drop-shadow(0px 0px 2px var(--main-color)) drop-shadow(0px 0px 8px var(--main-color));
		scale: 1.01;
	}
</style>
