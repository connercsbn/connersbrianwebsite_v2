<script lang="ts">
	import { onMount } from 'svelte';
	import { setBrian } from '../lib/brian';

	const X = 0;
	const Y = 1;
	let innerWidth: number;
	let canvas: HTMLCanvasElement;
	let brian: HTMLImageElement;
	let ctx: CanvasRenderingContext2D;
	let comp: GlobalCompositeOperation;
	let hasBegun = false;
	let music: HTMLAudioElement;
	let introLength = 7450;
	// bpm is 96.484;
	// roughly 1860 frames per bar
	// 465 frames per beat if 10ms apart
	let totalFrames = 185.9;
	let lastLoop = new Date();
	$: {
		if (ctx) {
			ctx.globalCompositeOperation = comp;
		}
	}
	$: innerWidth && resizeCanvas();
	$: xBound = canvas?.width / 1.5;
	$: yBound = canvas?.height / 1.5;
	$: xBoundOffset = canvas?.width / 2 - xBound / 2;
	$: yBoundOffset = canvas?.height / 2 - yBound / 2;

	let comps = {
		'1': 'luminosity',
		'2': 'color',
		'3': 'saturation',
		'4': 'hue',
		// '5': 'exclusion',
		// '6': 'difference',
		// '7': 'soft-light',
		// '8': 'hard-light',
		// '9': 'color-burn',
		'0': 'lighten',
		q: 'darken',
		w: 'overlay',
		e: 'multiply',
		r: 'xor',
		t: 'destination-atop',
		y: 'destination-out',
		u: 'destination-in',
		i: 'destination-over',
		o: 'source-out'
		// 'p': 'source-in'                ,
		// '[': 'source-over'              ,
		// 'space': 'copy'
	};

	let compsValues = Object.values(comps);

	onMount(() => {
		music = new Audio('redboneedited.opus');
		console.log(music);
		let tempCtx = canvas.getContext('2d');
		if (!tempCtx) return;
		ctx = tempCtx;
		ctx.imageSmoothingEnabled = true;
		canvas.width = window?.innerWidth;
		canvas.height = window?.innerHeight;
		console.log(ctx);
		console.log(canvas);
		brian = setBrian(brian);
	});

	let frame = 0;
	let startpoint = [100, 100];
	let endpoint = [0, 0];
	let curr = [0, 0];
	let fx = (thisFrame: number) => {
		return startpoint[X] + (endpoint[X] - startpoint[X]) * (thisFrame / totalFrames);
	};
	let fy = (thisFrame: number) => {
		return startpoint[Y] + (endpoint[Y] - startpoint[Y]) * (thisFrame / totalFrames);
	};

	let interval: NodeJS.Timer | void;
	let currBeat = 0;
	let ranBeat = Math.ceil(Math.random() * 8);
	let nextBeat = 0;
	function loop() {
		console.log(xBoundOffset);
		if (!startpoint) {
			startpoint = [canvas.width / 2, canvas.height / 2];
		}
		interval = setInterval(() => {
			// if (frame % Math.floor(totalFrames / 16) == 0) {
			// 	if (ranBeat == currBeat) {
			// 		ctx.globalCompositeOperation = compsValues[
			// 			Math.floor(Math.random() * compsValues.length)
			// 		] as GlobalCompositeOperation;
			// 		nextBeat = (nextBeat + 3) % 8;
			// 		// currBeat = -1;
			// 		// ranBeat = Math.ceil(Math.random() * 8);
			// 	}
			// 	console.log(ctx.globalCompositeOperation);
			// 	console.log(currBeat, ranBeat);
			// 	currBeat++;
			// }
			// if (frame % totalFrames > 1) {
			if (frame % totalFrames < 1) {
				// new endpoint if frame is 0 (just starting again)
				endpoint = [
					Math.floor(Math.random() * xBound) + xBoundOffset,
					Math.floor(Math.random() * yBound) + yBoundOffset
				];
			}
			curr = [fx(frame % totalFrames), fy(frame % totalFrames)];
			// drawPath(curr[X], curr[Y]);
			draw(curr[X], curr[Y]);
			if ((frame % totalFrames) + Math.floor(Math.random() * 200) < 1) {
				ctx.globalCompositeOperation = compsValues[
					Math.floor(Math.random() * compsValues.length)
				] as GlobalCompositeOperation;
			}
			frame++;
			if (frame % totalFrames < 1) {
				startpoint = endpoint;
				// frame = 0;
				interval = clearInterval(interval as NodeJS.Timer);
				console.log('Last: ', Math.abs(lastLoop.getMilliseconds() - new Date().getMilliseconds()));
				lastLoop = new Date();
				loop();
			}
			// start position x at 0; then increment each draw to calculate position
		}, 10);
	}

	function drawPath(x: number, y: number) {
		ctx.beginPath();
		ctx.arc(x, y, 5, 0, 2 * Math.PI);
		ctx.stroke();
	}
	function draw(x: number, y: number) {
		x = canvas.width - x;
		y = canvas.height - y;
		const regular_w = canvas.width * 0.1;
		const regular_h = regular_w;
		const middle = [canvas.width / 2, canvas.height / 2];
		const distance = [Math.abs(x - middle[X]), Math.abs(y - middle[Y])];
		const distance_percentage = [distance[X] / middle[X], distance[Y] / middle[Y]];
		let w = 200 * (3.0 * distance_percentage[X]) ** 3 + regular_w;
		let h = 200 * (3.0 * distance_percentage[Y]) ** 3 + regular_h;

		let offset = [w / 2, h / 2];
		ctx.drawImage(brian, x - offset[X], y - offset[Y], w, h);
	}
	function resizeCanvas() {
		if (canvas) {
			console.log('called resizeCanvas');
			canvas.width = window?.innerWidth;
			canvas.height = window?.innerHeight;
		}
	}
	let introTimeOut: NodeJS.Timer;
	function handleBegin() {
		music.play().then(() => {
			hasBegun = true;
			setTimeout(() => {
				console.log(music.readyState);
				loop();
			}, introLength);
		});
		// wait until music is at a certain point, then loop.
	}
	function handleExit() {
		clearTimeout(introTimeOut);
		music.pause();
		// stop timing since song began
	}
</script>

<svelte:window bind:innerWidth />

<main>
	{#if !hasBegun && music}
		<div>{music.readyState}</div>
		<div class="beginExperienceContainer">
			<button class="beginExperience" on:click={handleBegin}>Begin experience</button>
		</div>
	{/if}

	<canvas bind:this={canvas} class="canvas-thing" />
</main>

<style>
	*,
	main {
		padding: 0;
		margin: 0;
	}
	.beginExperienceContainer {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 100vw;
		height: 100vh;
	}
	.beginExperience {
		padding: 10px;
	}
</style>
