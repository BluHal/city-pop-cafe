<script lang="ts">
	import type { VideoInfo } from '$lib/types';
	import videoInfoString from '../data/videoUrls.json';
	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();
	const videoInfos: VideoInfo[] = videoInfoString;

	function handleStationClick(videoInfo: VideoInfo) {
		dispatch('stationClick', { videoInfo });
	}
</script>

<div
	class="fixed top-0 left-0 overflow-x-hidden overflow-y-scroll bg-[#000000b3] z-40 w-dvw h-dvh p-10"
>
	<div class="grid stationSelectorGrid">
		{#each videoInfos as videoInfo}
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<div
				class="block w-full p-4 cursor-pointer"
				on:click={() => handleStationClick(videoInfo)}
				role="button"
				tabindex="0"
			>
				<img src="https://img.youtube.com/vi/{videoInfo.id}/maxresdefault.jpg" alt="" />
			</div>
		{/each}
	</div>
</div>
