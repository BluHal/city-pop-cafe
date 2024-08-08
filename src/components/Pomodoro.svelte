<script lang="ts">
	import { onDestroy } from 'svelte';
	import { linear } from 'svelte/easing';
	import { fade } from 'svelte/transition';

	export let cssVarStyles;

	const animationOptions = { duration: 100, easing: linear };

	let duration: number = 25 * 60;
	let alarmSound: any;
	let showToolTip = false;
	let showPomodoroFunctions = false;

	let timerStarted = false;
	let remainingTime: number = duration;
	let intervalId: number | null = null;
	let timeString: string = formatTime(remainingTime);
	let isPaused = false;

	function formatTime(seconds: number): string {
		const minutes = Math.floor(seconds / 60);
		const secs = seconds % 60;
		return `${padZero(minutes)}:${padZero(secs)}`;
	}

	function padZero(num: number): string {
		return num < 10 ? `0${num}` : num.toString();
	}

	function startTimer(): void {
		if (intervalId) return;

		intervalId = window.setInterval(() => {
			if (remainingTime > 0) {
				remainingTime--;
				timeString = formatTime(remainingTime);
				timerStarted = true;
			} else {
				stopTimer();
				playSound();
				setTimeout(() => {
					stopSound();
				}, 2000);
			}
		}, 1000);
	}

	function stopTimer(): void {
		timerStarted = false;
		remainingTime = duration;
		isPaused = false;
		if (intervalId !== null) {
			clearInterval(intervalId);
			intervalId = null;
		}
	}

	function pauseTimer(): void {
		if (!intervalId) return;

		clearInterval(intervalId);
		intervalId = null;
		isPaused = true;
	}

	function resumeTimer(): void {
		if (!isPaused) return;

		startTimer();
		isPaused = false;
	}

	function startBreak(): void {
		remainingTime = 5 * 60;
		startTimer();
	}

	onDestroy(() => {
		stopTimer();
	});

	$: remainingTime = duration;
	$: timeString = formatTime(remainingTime);

	function toggleTooltip() {
		showToolTip = !showToolTip;
	}

	function playSound() {
		alarmSound.play();
	}

	function stopSound() {
		alarmSound.pause();
		alarmSound.currentTime = 0;
	}
</script>

<div class="relative" style={cssVarStyles}>
	{#if timerStarted}
		<img
			class="z-30 absolute scale-[1.6] top-2 -left-[1px] shadow-icon"
			src="/gifs/loading.gif"
			alt="loading"
		/>
	{/if}

	<button
		class="cursor-pointer"
		on:mouseenter={() => toggleTooltip()}
		on:mouseleave={() => toggleTooltip()}
		on:click={() => (showPomodoroFunctions = !showPomodoroFunctions)}
	>
		<i class="fa-solid fa-hourglass-end fa-lg shadow-icon {timerStarted ? 'opacity-0' : ''}"></i>
	</button>
	{#if showToolTip}
		<div
			class="absolute -left-9 top-8 p-1 rounded-lg shadow-tooltip z-50"
			transition:fade={animationOptions}
		>
			<span class="shadow-text text-sm">Pomodoro</span>
		</div>
	{/if}

	{#if showPomodoroFunctions}
		<div class="absolute -left-7 top-9 p-2 flex flex-col gap-2">
			<span class="shadow-text text-sm cursor-pointer">{timeString}</span>

			{#if timerStarted}
				{#if isPaused}
					<div class="flex flex-col gap-2 items-baseline">
						<button on:click={() => stopTimer()} class="shadow-text text-xs cursor-pointer"
							>STOP</button
						>
						<button on:click={() => resumeTimer()} class="shadow-text text-xs cursor-pointer"
							>CONTINUE</button
						>
					</div>
				{:else}
					<button on:click={() => pauseTimer()} class="shadow-text text-xs cursor-pointer"
						>PAUSE</button
					>
				{/if}
			{:else}
				<button on:click={() => startTimer()} class="shadow-text text-xs cursor-pointer"
					>START</button
				>
			{/if}

			<button on:click={() => startBreak()} class="shadow-text text-xs cursor-pointer text-left"
				>BREAK</button
			>
		</div>
	{/if}
	<audio src="/sounds/alarm.mp3" bind:this={alarmSound}></audio>
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
