<script lang="ts">
	import { linear } from 'svelte/easing';
	import { fade } from 'svelte/transition';

	export let cssVarStyles;
	const animationOptions = { duration: 100, easing: linear };

	let showToolTip = false;

	function toggleTooltip() {
		showToolTip = !showToolTip;
	}
</script>

<div style={cssVarStyles}>
	<div class="relative">
		<button
			class="cursor-pointer"
			on:mouseenter={() => toggleTooltip()}
			on:mouseleave={() => toggleTooltip()}
		>
			<i class="fa-solid fa-campground fa-lg shadow-icon"></i>
		</button>
		{#if showToolTip}
			<div
				class="absolute -left-6 top-9 p-1 rounded-lg shadow-tooltip z-50"
				transition:fade={animationOptions}
			>
				<span class="shadow-text text-sm">Sounds</span>
			</div>
		{/if}
	</div>
</div>

<style>
	.shadow-icon {
		filter: drop-shadow(0px 0px 2px var(--main-color)) drop-shadow(0px 0px 8px var(--main-color));
	}
</style>
