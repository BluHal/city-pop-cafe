<script lang="ts">
	import type { VideoInfo } from '$lib/types';
	import StationSelector from '../components/StationSelector.svelte';
	import YoutubeEmbed from '../components/YoutubeEmbed.svelte';
	import videoInfoString from '../data/videoUrls.json';

	const videoInfos: VideoInfo[] = videoInfoString;
	let player: any;
	let selectedVideo: VideoInfo = videoInfos[0];
	let videoLoaded = false;
	let videoPlaying = true;
	let gifIndex = 1;
	let showStationSelector = false;

	const toggleVideo = () => {
		if (videoLoaded) {
			if (videoPlaying) player.pauseVideo();
			else player.playVideo();

			videoPlaying = !videoPlaying;
		} else {
			loadVideoById();
		}
	};

	const changeGif = () => {
		const availableGifIndex = [1, 2];
		let newGifIndex: number | undefined = gifIndex;

		while (newGifIndex == gifIndex) {
			newGifIndex = getRandomElement(availableGifIndex);
		}

		if (!newGifIndex) return;

		gifIndex = newGifIndex;
	};

	const shuffleVideo = () => {
		let newVideoToPlay: VideoInfo | undefined = selectedVideo;

		while (newVideoToPlay == selectedVideo) {
			newVideoToPlay = getRandomElement(videoInfos);
		}

		if (!newVideoToPlay) return;
		selectedVideo = newVideoToPlay;
		loadVideoById();
	};

	const loadVideoById = () => {
		player.loadVideoById(selectedVideo.id);
		videoLoaded = true;
	};

	function getRandomElement<T>(array: T[]): T | undefined {
		if (array.length === 0) return undefined;
		const randomIndex = Math.floor(Math.random() * array.length);
		return array[randomIndex];
	}
</script>

<div class="w-dvw h-dvh flex flex-col p-10 items-stretch justify-between">
	<div class="w-dvw h-dvh absolute inset-0 z-0">
		<YoutubeEmbed bind:player />
	</div>

	<div
		class="w-dvw h-dvh absolute inset-0 z-10 bg-cover"
		style="background-image: url('/gifs/gif-{gifIndex}.gif');"
	></div>

	<div class="vignette"></div>

	<div class="z-20 text-[#00CED1]">
		<button><i class="fa-solid fa-stopwatch fa-lg shadow-icon"></i></button>
	</div>

	<div class="flex flex-col gap-3">
		<div class="z-20 text-[#00CED1] flex gap-5">
			<button class="cursor-pointer" on:click={() => toggleVideo()}>
				<i class="fa-solid {videoLoaded && videoPlaying ? 'fa-pause' : 'fa-play'} fa-lg shadow-icon"
				></i>
			</button>
			<button class="cursor-pointer" on:click={() => shuffleVideo()}>
				<i class="fa-solid fa-shuffle fa-lg shadow-icon"></i>
			</button>
			<button class="cursor-pointer">
				<i class="fa-solid fa-backward fa-lg shadow-icon"></i>
			</button>
			<button class="cursor-pointer">
				<i class="fa-solid fa-forward fa-lg shadow-icon"></i>
			</button>
			<button class="cursor-pointer" on:click={() => changeGif()}>
				<i class="fa-solid fa-image fa-lg shadow-icon"></i>
			</button>
		</div>

		<button
			class="z-20 text-[#00CED1] flex gap-5 cursor-pointer"
			on:click={() => (showStationSelector = true)}
		>
			<h1 class="shadow-text">Now Playing:</h1>
			<span class="shadow-text">{selectedVideo.title}</span>
		</button>
	</div>

	{#if showStationSelector}
		<StationSelector></StationSelector>
	{/if}
</div>
