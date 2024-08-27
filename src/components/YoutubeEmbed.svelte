<script lang="ts">
	import { onMount } from 'svelte';
	import { createEventDispatcher } from 'svelte';

	// @ts-ignore
	export let player;
	export let initialVideoId = '';

	const dispatch = createEventDispatcher();

	const ytPlayerId = 'youtube-player';

	onMount(() => {
		const script = document.createElement('script');
		script.src = 'https://www.youtube.com/iframe_api';
		script.async = true;
		script.onload = () => {
			createPlayer();
		};
		document.body.appendChild(script);

		function createPlayer() {
			// @ts-ignore
			if (window.YT && window.YT.Player) {
				// @ts-ignore
				player = new YT.Player(ytPlayerId, {
					height: '100%',
					width: '100%',
					videoId: initialVideoId,
					playerVars: { autoplay: 1, controls: 1, showinfo: 0, rel: 0, modestbranding: 1 },
					events: {
						onStateChange: onPlayerStateChange,
						onReady: onPlayerReady,
						onError: onPlayerError
					}
				});
			} else {
				// @ts-ignore
				window.onYouTubeIframeAPIReady = createPlayer;
			}
		}
	});

	// @ts-ignore
	function onPlayerStateChange(event) {
		// @ts-ignore
		if (event.data == YT.PlayerState.PLAYING) {
			// @ts-ignore
			var videoData = player.getVideoData();
			dispatch('videoLoaded', videoData.title);
		}
	}

	function onPlayerReady() {
		dispatch('playedReady', true);
	}

	function onPlayerError() {
		dispatch('onVideoLoadFailure', 'Failed to load video');
	}
</script>

<div id={ytPlayerId} />
