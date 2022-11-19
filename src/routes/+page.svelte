<script lang="ts">
	import Counter from './Counter.svelte';
	import welcome from '$lib/images/svelte-welcome.webp';
	import welcome_fallback from '$lib/images/svelte-welcome.png';
	import { readable } from 'svelte/store';
	import { onMount } from 'svelte';
	import beepSrc from '$lib/audio/one_beep.mp3';
	import stopSrc from '$lib/audio/stop.mp3';

	let precision = 0.01;
	let current = 60;
	let interval: NodeJS.Timer | undefined;
	let beep = new Audio(beepSrc);
	let stop = new Audio(stopSrc);
	let extensionUsed = false;

	$: warning = current <= 10 && current > 5;
	$: document.querySelector('.app')?.classList.toggle('warning', warning);
	$: hurry = current <= 5 && current !== 0;
	$: document.querySelector('.app')?.classList.toggle('hurry', hurry);
	$: timefoul = Math.abs(current) < precision;
	$: document.querySelector('.app')?.classList.toggle('timefoul', timefoul);

	$: hurry &&
		Math.abs(current - Math.round(current)) <= precision &&
		current >= precision &&
		(beep.cloneNode() as HTMLAudioElement).play();
	$: timefoul && playStop(2, 200);

	$: timefoul && clearInterval(interval);

	function playStop(times: number, gap: number) {
		let count = 0;
		const audioInterval = setInterval(() => {
			if (count === times) {
				clearInterval(audioInterval);
			} else {
				(stop.cloneNode() as HTMLAudioElement).play();
				count++;
			}
		}, gap);
	}

	function useExtension() {
		if (!extensionUsed) {
			current += 30;
			extensionUsed = true;
		}
	}

	function startTimer() {
		if (!interval) {
			interval = setInterval(() => {
				current -= 0.1;
			}, 100);
		}
	}

	function pauseTimer() {
		clearInterval(interval);
		interval = undefined;
	}

	function resetTimer(time = 0) {
		pauseTimer();
		current = time || 30;
		extensionUsed = false;
		startTimer();
	}

	function keydownHandler(event: KeyboardEvent) {
		if (event.key === 'z') {
			resetTimer();
		}
		if (event.key === 'x') {
			useExtension();
		}
		if (event.key === 'c') {
			startTimer();
		}
		if (event.key === 'v') {
			pauseTimer();
		}
		if (event.key === 's') {
			startMatch();
		}
	}

	function startMatch() {
		resetTimer(60);
	}
</script>

<svelte:head>
	<title>Pool Timer</title>
	<meta name="description" content="Timer for biliards" />
</svelte:head>

<svelte:body on:keydown={keydownHandler} />

<section class="timer">
	<div class="digits">
		{current.toFixed(1)}
	</div>
	<button on:click={() => resetTimer(30)}>Reset = Z</button>
	<button on:click={() => useExtension()}>Use Extenstion = X</button>
	<button on:click={() => startTimer()}>Resume Timer = C</button>
	<button on:click={() => pauseTimer()}>PauseTimer = V</button>
	<button on:click={() => startMatch()}>Start Match = S </button>
</section>

<!-- <audio src="audio/stop.mp3"></audio> -->
<style>
	section {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		flex: 0.6;
		--color-bg-1: red;
	}

	.digits {
		font-size: 512px;
	}

	:global(.warning) {
		background-color: rgb(256, 256, 0);
	}

	:global(.hurry) {
		background-color: rgb(256, 128, 0);
	}

	:global(.timefoul) {
		background-color: rgb(256, 0, 0);
	}
</style>
