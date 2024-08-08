<script lang="ts">
	import { linear } from 'svelte/easing';
	import { fade } from 'svelte/transition';
	import ColorPicker from 'svelte-awesome-color-picker';
	import { createEventDispatcher } from 'svelte';

	export let hex;
	export let cssVarStyles;

	const animationOptions = { duration: 100, easing: linear };
	const dispatch = createEventDispatcher();

	let showSettingsModal = false;
	let showToolTip = false;

	function toggleSettingsModal() {
		showSettingsModal = !showSettingsModal;
		dispatch('settingsModalToggle', showSettingsModal);
	}
</script>

<div style={cssVarStyles}>
	<div class="relative">
		<button
			class="cursor-pointer"
			on:mouseenter={() => (showToolTip = !showToolTip)}
			on:mouseleave={() => (showToolTip = !showToolTip)}
			on:click={() => toggleSettingsModal()}
		>
			<i class="fa-solid fa-gear fa-lg shadow-icon"></i>
		</button>

		{#if showToolTip}
			<div
				class="absolute -left-6 top-8 p-1 rounded-lg shadow-tooltip z-50"
				transition:fade={animationOptions}
			>
				<span class="shadow-text text-sm text-nowrap">Settings</span>
			</div>
		{/if}
	</div>
	{#if showSettingsModal}
		<div
			class="w-dvw h-dvh fixed inset-0 bg-black bg-opacity-20 backdrop-blur-sm flex justify-center items-center z-[100]"
		>
			<div
				class="relative sm:w-5/6 md:w-3/6 lg:w-2/6 bg-black bg-opacity-90 flex flex-col justify-center text-center rounded-2xl p-4 md:p-8"
			>
				<div class="absolute text-white top-0 right-0 m-3">
					<button class="cursor-pointer" on:click={() => toggleSettingsModal()}
						><i class="fa-solid fa-xmark fa-lg shadow-icon"></i></button
					>
				</div>
				<span class="text-2xl mb-5">Settings</span>
				<div class="dark">
					<ColorPicker
						{hex}
						isAlpha={false}
						on:input={(event) => {
							dispatch('colorChanged', event.detail.hex);
						}}
					/>
				</div>
			</div>
		</div>
	{/if}
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
	.dark {
		--cp-bg-color: #333;
		--cp-border-color: white;
		--cp-text-color: white;
		--cp-input-color: #555;
		--cp-button-hover-color: #777;
	}

	/* .applyBorderColor {
		border-color: var(--main-color);
	} */
</style>
