<script lang="ts">
	import type { VideoInfo } from '$lib/types';
	import videoInfoString from '../data/videoUrls.json';

	const videoInfos: VideoInfo[] = videoInfoString;
</script>

<div class="fixed top-0 left-0 overflow-x-hidden overflow-y-scroll bg-[#000000b3] z-30 w-dvw h-dvh p-10">
	<div class="grid stationSelectorGrid">
		{#each videoInfos as videoInfo}
			<div class="block w-full p-4 cursor-pointer">
				<img src="https://img.youtube.com/vi/{videoInfo.id}/maxresdefault.jpg" alt="" />
			</div>
		{/each}
	</div>
</div>
