<script lang="ts">
	import type { VideoInfo } from '$lib/types';
	import { onMount } from 'svelte';
	import StationSelector from '../components/StationSelector.svelte';
	import YoutubeEmbed from '../components/YoutubeEmbed.svelte';
	import videoInfoString from '../data/videoUrls.json';

	let staticSound: any;
	const videoInfos: VideoInfo[] = videoInfoString;

	let firstStart = true;
	let player: any;
	let videoTitle: string;
	let selectedVideo: VideoInfo = getRandomElement(videoInfos) || videoInfos[0];
	let videoLoaded = false;
	let videoPlaying = true;
	let gifIndex = 1;
	let showStationSelector = false;
	let volume = 100;
	let showStaticEffect = false;
	let showOriginalVideo = false;
	let showIndicator = true;

	onMount(() => {
		let interval = setInterval(updateIndicator, 750);

		window.addEventListener('keydown', handleKeyDown);
		window.addEventListener('click', handleMouseClick);
	});

	function handleKeyDown(event: KeyboardEvent) {
		if (firstStart) {
			firstStart = false;
			toggleVideo();
			return;
		}

		switch (event.key) {
			case 'g':
				changeGif();
				break;
			case 's':
				shuffleVideo();
				break;
			case ' ':
				toggleVideo();
			default:
				break;
		}
	}

	function handleMouseClick() {
		if (!firstStart) return;

		firstStart = false;
		toggleVideo();
	}

	function updateIndicator() {
		showIndicator = !showIndicator;
	}

	function onVideoLoaded(event: any) {
		videoLoaded = true;
		videoTitle = event.detail;
		stopStaticSound();
	}

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
		const availableGifIndex = [1, 2, 3, 4, 5, 6, 7];
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
		if (videoLoaded) {
			playStaticSound();
			changeGif();
		}
		setTimeout(() => {
			player.loadVideoById(selectedVideo.id);
		}, 100);
	};

	function stationChanged(event: any) {
		selectedVideo = event.detail.videoInfo;
		loadVideoById();
		showStationSelector = false;
	}

	function playStaticSound() {
		showStaticEffect = true;
		staticSound.play();
	}

	function stopStaticSound() {
		staticSound.pause();
		staticSound.currentTime = 0;
		showStaticEffect = false;
	}

	function getRandomElement<T>(array: T[]): T | undefined {
		if (array.length === 0) return undefined;
		const randomIndex = Math.floor(Math.random() * array.length);
		return array[randomIndex];
	}
</script>

<div class="w-dvw h-dvh flex flex-col p-10 items-stretch justify-between">
	<div class="w-dvw h-dvh absolute inset-0 {showOriginalVideo ? 'z-20' : 'z-0'}">
		<YoutubeEmbed bind:player on:videoLoaded={onVideoLoaded} />
	</div>

	{#if showStaticEffect}
		<div
			class="w-dvw h-dvh absolute inset-0 z-20 bg-cover"
			style="background-image: url('/gifs/static.gif');"
		></div>
	{/if}

	<!-- svelte-ignore a11y-click-events-have-key-events -->
	{#if !firstStart}
		<div
			class="absolute inset-1/4 cursor-pointer z-30 flex items-middle justify-center centerControllerWrapper"
			on:click={() => toggleVideo()}
			role="button"
			tabindex="0"
		>
			<button class="text-neon-pink scale-0">
				<i class="fa-solid {videoLoaded && videoPlaying ? 'fa-pause' : 'fa-play'} fa-lg shadow-icon"
				></i>
			</button>
		</div>
	{/if}

	<div
		class="w-dvw h-dvh absolute inset-0 z-10 bg-cover"
		style="background-image: url('/gifs/gif-{gifIndex}.gif');"
	></div>

	<div class="vignette w-dvw h-dvh z-30"></div>

	<div class="z-20 text-white flex justify-between gap-5">
		{#if !firstStart}
			<!-- TODO -->
			<button><i class="fa-solid fa-stopwatch fa-lg shadow-icon hidden"></i></button>

			{#if videoLoaded}
				<button on:click={() => (showOriginalVideo = !showOriginalVideo)}
					><i class="fa-brands fa-youtube fa-lg shadow-icon"></i></button
				>
			{/if}
		{:else}
			<span></span>
		{/if}
	</div>

	{#if !firstStart}
		<div class="flex flex-col gap-5 w-full">
			<div class="z-20 text-white flex gap-5">
				<button class="cursor-pointer w-4" on:click={() => toggleVideo()}>
					<i
						class="fa-solid {videoLoaded && videoPlaying
							? 'fa-pause'
							: 'fa-play'} fa-lg shadow-icon"
					></i>
				</button>

				<button class="cursor-pointer" on:click={() => shuffleVideo()}>
					<i class="fa-solid fa-shuffle fa-lg shadow-icon"></i>
				</button>

				<button class="cursor-pointer" on:click={() => changeGif()}>
					<i class="fa-solid fa-image fa-lg shadow-icon"></i>
				</button>

				<!-- TODO -->
				<div class="items-center w-full max-w-screen-lg gap-2 pt-2 hidden">
					<div
						class="w-2 h-5 bg-neon-pink shadow-icon {volume <= 0 ? 'opacity-50' : 'opacity-100'}"
					></div>
					<div
						class="w-2 h-5 bg-neon-pink shadow-icon {volume <= 10 ? 'opacity-50' : 'opacity-100'}"
					></div>
					<div
						class="w-2 h-5 bg-neon-pink shadow-icon {volume <= 20 ? 'opacity-50' : 'opacity-100'}"
					></div>
					<div
						class="w-2 h-5 bg-neon-pink shadow-icon {volume <= 30 ? 'opacity-50' : 'opacity-100'}"
					></div>
					<div
						class="w-2 h-5 bg-neon-pink shadow-icon {volume <= 40 ? 'opacity-50' : 'opacity-100'}"
					></div>
					<div
						class="w-2 h-5 bg-neon-pink shadow-icon {volume <= 50 ? 'opacity-50' : 'opacity-100'}"
					></div>
					<div
						class="w-2 h-5 bg-neon-pink shadow-icon {volume <= 60 ? 'opacity-50' : 'opacity-100'}"
					></div>
					<div
						class="w-2 h-5 bg-neon-pink shadow-icon {volume <= 70 ? 'opacity-50' : 'opacity-100'}"
					></div>
					<div
						class="w-2 h-5 bg-neon-pink shadow-icon {volume <= 80 ? 'opacity-50' : 'opacity-100'}"
					></div>
					<div
						class="w-2 h-5 bg-neon-pink shadow-icon {volume <= 90 ? 'opacity-50' : 'opacity-100'}"
					></div>
				</div>
			</div>

			<div class="w-full relative z-20">
				<div class="flex align-middle flex-nowrap whitespace-nowrap">
					<!-- svelte-ignore a11y-click-events-have-key-events -->
					<span
						class="text-white shadow-text text-ellipsis overflow-hidden md:overflow-visible lg:overflow-visible cursor-pointer"
						on:click={() => (showStationSelector = true)}
						role="button"
						tabindex="0">Now Playing: {videoLoaded ? videoTitle : '. . .'}</span
					>
				</div>
			</div>
		</div>
	{:else}
		<div class="w-full z-20 flex gap-1">
			<span
				class="text-white shadow-text text-ellipsis overflow-hidden md:overflow-visible lg:overflow-visible cursor-pointer"
				>Press a button to start</span
			>
			<div class="w-3 h-6 bg-white shadow-icon {showIndicator ? '' : 'hidden'}"></div>
		</div>
	{/if}

	{#if showStationSelector}
		<StationSelector on:stationClick={stationChanged}></StationSelector>
	{/if}

	<audio src="/sounds/static-noise.mp3" bind:this={staticSound}></audio>
</div>
