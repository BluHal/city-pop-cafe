<script lang="ts">
	import { createEventDispatcher } from 'svelte';
	import { linear } from 'svelte/easing';
	import { fade } from 'svelte/transition';
	import type { VideoInfo } from '../lib/types';

	export let cssVarStyles;

	const storedLinksKey = 'storedLinks';
	const animationOptions = { duration: 100, easing: linear };
	const dispatch = createEventDispatcher();

	let showToolTip = false;
	let showAddToLibModal = false;
	let showToast = false;
	let toastError = '';

	let addToLibInputValue: string = '';
	let addToLibInputRef: HTMLInputElement | null = null;

	function toggleTooltip() {
		showToolTip = !showToolTip;
	}

	function toggleAddToLibModal() {
		showAddToLibModal = !showAddToLibModal;
		dispatch('addToLibModalToggle', showAddToLibModal);
	}
	function onAddToLibEnter(event: KeyboardEvent): void {
		if (event.key === 'Enter') {
			const ytUrl = new URL(addToLibInputValue);
			const ytUrlSearchParams = new URLSearchParams(ytUrl.search);

			if (!ytUrlSearchParams.has('v')) {
				dispatch('videoInfoAdded', null);
				showAndHandleToast('Invalid YouTube URL!');
				return;
			}

			const ytId = ytUrlSearchParams.get('v');
			const newVideoInfo = { id: ytId };
			const storedLinksString = localStorage.getItem(storedLinksKey);

			if (storedLinksString) {
				const storedLinks: VideoInfo[] = JSON.parse(storedLinksString);
				const isPresent = storedLinks.find((storedLink) => storedLink.id === ytId);

				if (isPresent) {
					dispatch('videoInfoAdded', null);
					showAndHandleToast('Video already present!');
				} else {
					localStorage.setItem(storedLinksKey, JSON.stringify([...storedLinks, newVideoInfo]));
					dispatch('videoInfoAdded', newVideoInfo);
					showAndHandleToast();
				}
			} else {
				localStorage.setItem(storedLinksKey, JSON.stringify([newVideoInfo]));
				dispatch('videoInfoAdded', newVideoInfo);
				showAndHandleToast();
			}

			addToLibInputValue = '';
			setTimeout(() => {
				addToLibInputRef?.focus();
			}, 1);
		}
	}

	function showAndHandleToast(error: string = '') {
		showToast = true;
		toastError = error;

		setTimeout(() => {
			showToast = false;
			toastError = '';
		}, 2000);
	}
</script>

<div style={cssVarStyles}>
	<div class="relative">
		<button
			class="cursor-pointer"
			on:mouseenter={() => toggleTooltip()}
			on:mouseleave={() => toggleTooltip()}
			on:click={() => toggleAddToLibModal()}
		>
			<i class="fa-solid fa-cloud-arrow-up fa-lg shadow-icon"></i>
		</button>
		{#if showToolTip}
			<div
				class="absolute -left-6 top-9 p-1 rounded-lg shadow-tooltip z-50"
				transition:fade={animationOptions}
			>
				<span class="shadow-text text-sm">Add</span>
			</div>
		{/if}
	</div>

	{#if showAddToLibModal}
		<div
			class="w-dvw h-dvh fixed inset-0 bg-black bg-opacity-20 backdrop-blur-sm flex justify-center items-center z-[100]"
		>
			<div
				class="relative w-3/4 md:2/3 lg:w-2/5 bg-black bg-opacity-90 flex flex-col justify-center text-center rounded-2xl p-4 md:p-8"
			>
				<div class="absolute text-white top-0 right-0 m-3">
					<button class="cursor-pointer" on:click={() => toggleAddToLibModal()}
						><i class="fa-solid fa-xmark fa-lg shadow-icon"></i></button
					>
				</div>
				<span class="text-2xl mb-5">Add New Videos</span>
				<form>
					<input
						type="text"
						class="text-white text-lg rounded w-full h-14 placeholder:italic bg-transparent border px-2 todoInput applyBorderColor"
						placeholder="Youtube video URL"
						on:keydown={onAddToLibEnter}
						bind:this={addToLibInputRef}
						bind:value={addToLibInputValue}
					/>
				</form>
			</div>
			{#if showToast}
				<div
					class="absolute top-10 right-10 {toastError ? 'bg-red-500' : 'bg-green-500'} rounded-lg"
					transition:fade={animationOptions}
				>
					<div class="w-full h-full px-5 py-2 flex justify-center items-center">
						<p>{toastError ? toastError : 'Video Added!'}</p>
					</div>
				</div>
			{/if}
		</div>
	{/if}
</div>

<style>
	.shadow-icon {
		filter: drop-shadow(0px 0px 2px var(--main-color)) drop-shadow(0px 0px 8px var(--main-color));
	}
</style>
