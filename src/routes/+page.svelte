<script lang="ts">
	import type { VideoInfo } from '$lib/types';
	import { onMount } from 'svelte';
	import AddToLibrary from '../components/AddToLibrary.svelte';
	import Pomodoro from '../components/Pomodoro.svelte';
	import Settings from '../components/Settings.svelte';
	import StationSelector from '../components/StationSelector.svelte';
	import Todo from '../components/Todo.svelte';
	import YoutubeEmbed from '../components/YoutubeEmbed.svelte';
	import videoInfoString from '../data/videoUrls.json';

	const storedLinksKey = 'storedLinks';
	const availableGifIndexes = [1, 2, 3, 4, 5, 6, 7, 8];
	const styles = {
		'main-color': '#FF00FF'
	};

	$: cssVarStyles = Object.entries(styles)
		.map(([key, value]) => `--${key}:${value}`)
		.join(';');

	let videoInfos: VideoInfo[] = videoInfoString;
	let staticSound: any;
	let isPlayerReady = false;
	let firstStart = true;
	let player: any;
	let videoTitle: string;
	let selectedVideo: VideoInfo | null;
	let videoLoaded = false;
	let videoPlaying = true;
	let gifIndex = getRandomElement(availableGifIndexes);
	let showStationSelector = false;
	let volume = 100;
	let showStaticEffect = false;
	let showOriginalVideo = false;
	let showIndicator = true;
	let modalOpened = false;

	onMount(() => {
		let interval = setInterval(updateIndicator, 750);

		window.addEventListener('keydown', handleKeyDown);
		window.addEventListener('click', handleMouseClick);

		const storedMainColor = localStorage.getItem('mainColor');
		if (storedMainColor) {
			styles['main-color'] = JSON.parse(storedMainColor);
		}

		loadVideoInfosFromStorage();

		selectedVideo = getRandomElement(videoInfos) || videoInfos[0];
	});

	function handleKeyDown(event: KeyboardEvent) {
		if (firstStart) {
			firstStart = false;
			toggleVideo();
			return;
		}

		if (modalOpened) return;

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
		if (!firstStart || modalOpened) return;

		firstStart = false;
		toggleVideo();
	}

	function updateIndicator() {
		showIndicator = !showIndicator;
	}

	function onPlayerReady(event: any) {
		isPlayerReady = event.detail;
	}

	function onVideoLoaded(event: any) {
		videoLoaded = true;
		videoTitle = event.detail;
		stopStaticSound();
	}

	function onVideoLoadFailure() {
		shuffleVideo();
	}

	const toggleVideo = () => {
		if (!isPlayerReady) return;

		if (videoLoaded) {
			if (videoPlaying) player.pauseVideo();
			else player.playVideo();

			videoPlaying = !videoPlaying;
		} else {
			loadVideoById();
		}
	};

	const changeGif = () => {
		if (!gifIndex) return;

		gifIndex = getNextElement(availableGifIndexes, gifIndex - 1);
	};

	const shuffleVideo = () => {
		if (!selectedVideo) return;
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
			if (!selectedVideo) return;

			player.loadVideoById(selectedVideo.id, selectedVideo.startAt || 0);
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

	function onModalToggle(event: any) {
		modalOpened = event.detail;
	}

	function onColorChanged(event: any) {
		styles['main-color'] = event.detail;
		localStorage.setItem('mainColor', JSON.stringify(event.detail));
	}

	function getRandomElement<T>(array: T[]): T | undefined {
		if (array.length === 0) return undefined;
		const randomIndex = Math.floor(Math.random() * array.length);
		return array[randomIndex];
	}

	function getNextElement<T>(array: T[], currentIndex: number): T | undefined {
		if (array.length === 0) return undefined;
		const nextIndex = currentIndex + 1;
		if (nextIndex >= array.length) return array[0];

		return array[nextIndex];
	}

	function loadVideoInfosFromStorage(): void {
		const storedVideoLinks = localStorage.getItem(storedLinksKey);
		if (storedVideoLinks) {
			videoInfos.push(...JSON.parse(storedVideoLinks));
		}
	}

	function onVideoInfoAdded(event: any): void {
		const response = event.detail;

		if (!event.detail) return;

		videoInfos.push(response);
	}
</script>

<div class="w-dvw h-dvh flex flex-col p-10 items-stretch justify-between" style={cssVarStyles}>
	<div class="w-dvw h-dvh inset-0 {showOriginalVideo ? 'absolute z-20' : 'hidden'}">
		<YoutubeEmbed
			bind:player
			on:videoLoaded={onVideoLoaded}
			on:onVideoLoadFailure={onVideoLoadFailure}
			on:playedReady={onPlayerReady}
		/>
	</div>

	{#if showStaticEffect}
		<div
			class="w-dvw h-dvh absolute inset-0 z-20 bg-cover"
			style="background-image: url('/gifs/static.gif');"
		></div>
	{/if}

	<!-- svelte-ignore a11y-click-events-have-key-events -->
	{#if !firstStart && !modalOpened}
		<div
			class="absolute inset-1/4 cursor-pointer z-30 flex items-middle justify-center centerControllerWrapper"
			on:click={() => toggleVideo()}
			role="button"
			tabindex="0"
		>
			<button class="text-white scale-0">
				<i class="fa-solid {videoLoaded && videoPlaying ? 'fa-pause' : 'fa-play'} fa-lg shadow-icon"
				></i>
			</button>
		</div>
	{/if}

	<div
		class="w-dvw h-dvh absolute inset-0 z-10 bg-cover"
		style="background-image: url('/gifs/gif-{gifIndex}.gif');"
	></div>

	<div
		class="w-dvw h-dvh absolute inset-0 z-10 bg-cover"
		style="background-image: url('/gifs/gif-{gifIndex}.gif');"
	></div>

	<div class="vignette w-dvw h-dvh z-30"></div>

	<div class="z-20 text-white flex justify-end gap-5">
		{#if !firstStart && videoLoaded}
			<!-- <BackgroundSoundSelector {cssVarStyles} /> -->
			<AddToLibrary
				on:addToLibModalToggle={onModalToggle}
				{cssVarStyles}
				on:videoInfoAdded={onVideoInfoAdded}
			/>
			<Todo on:todoModalToggle={onModalToggle} {cssVarStyles} />
			<Pomodoro {cssVarStyles} />

			<button on:click={() => (showOriginalVideo = !showOriginalVideo)}
				><i class="fa-brands fa-youtube fa-lg shadow-icon"></i></button
			>

			<Settings
				{cssVarStyles}
				hex={styles['main-color']}
				on:settingsModalToggle={onModalToggle}
				on:colorChanged={onColorChanged}
			/>
		{:else}
			<span></span>
		{/if}
	</div>

	{#if !firstStart}
		<div class="flex flex-col gap-5 w-full z-10">
			<div class="text-white flex gap-5">
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
			</div>

			<div class="w-full relative">
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
	{:else if isPlayerReady}
		<div class="w-full z-20 flex gap-1">
			<span
				class="text-white shadow-text text-ellipsis overflow-hidden md:overflow-visible lg:overflow-visible cursor-pointer"
				>Press a button to start</span
			>
			<div class="w-3 h-6 bg-white shadow-icon {showIndicator ? '' : 'hidden'}"></div>
		</div>
	{/if}

	{#if showStationSelector}
		<StationSelector
			on:stationClick={stationChanged}
			on:stationEsc={() => {
				showStationSelector = false;
			}}
		></StationSelector>
	{/if}

	<audio src="/sounds/static-noise.mp3" bind:this={staticSound}></audio>
</div>

<style>
	.shadow-icon {
		filter: drop-shadow(0px 0px 2px var(--main-color)) drop-shadow(0px 0px 8px var(--main-color));
	}

	.shadow-text {
		text-shadow:
			0px 0px 2px var(--main-color),
			0px 0px 15px var(--main-color),
			0px 0px 60px var(--main-color);
	}
</style>
