<script>
	import { onMount } from 'svelte';
    import { createEventDispatcher } from 'svelte';

	// @ts-ignore
	export let player;
	export let initialVideoId = '';

	const dispatch = createEventDispatcher();

	const ytPlayerId = 'youtube-player';

	onMount(() => {
		function load() {
			// @ts-ignore
			player = new YT.Player(ytPlayerId, {
				height: '100%',
				width: '100%',
				videoId: initialVideoId,
				playerVars: { autoplay: 1, controls: 0, showinfo: 0, rel: 0, modestbranding: 1 },
				events: {
					onStateChange: onPlayerStateChange
				}
			});
		}

		// @ts-ignore
		if (window.YT) {
			load();
		} else {
			// @ts-ignore
			window.onYouTubeIframeAPIReady = load;
		}
	});

	// @ts-ignore
	function onPlayerStateChange(event) {
		// @ts-ignore
		if (event.data == YT.PlayerState.PLAYING) {
			onVideoLoaded();
		}
	}

	function onVideoLoaded() {
		// @ts-ignore
		var videoData = player.getVideoData();
		dispatch('videoLoaded', videoData.title);
	}
</script>

<svelte:head>
	<script src="https://www.youtube.com/iframe_api"></script>
</svelte:head>

<div id={ytPlayerId} />
