<script>
	import { onMount } from 'svelte';
	let canvas, ctx;
	let draw = false;
	let oscillator, audioCtx, gain;
	let vol;
	let freq;
	let oscType = "sine";

	let sustain = true;
 	let position = null
	

	const [minFreq, maxFreq] = [20, 2000];


	$: {
		console.log("!!!")
		if (oscillator)
			oscillator.type = oscType;
	}

	$: {
		console.log("sass")
		if (audioCtx) {
		if (sustain)
			update(position)
		else
			setGain(0)}
	}
	const deadline  = () => audioCtx.currentTime + 0.03;

	onMount(() => {
		ctx = canvas.getContext('2d');
	});

	function audioContextInit() {
		window.AudioContext = window.AudioContext || window.webkitAudioContext;
  		audioCtx = new AudioContext();
		oscillator = audioCtx.createOscillator();
		gain = audioCtx.createGain();
		gain.connect(audioCtx.destination);
		oscillator.connect(gain);
		oscillator.type = oscType;
		setGain(0);
		oscillator.start();
	}

	function formatValue(val) {
		if (isNaN(val))
		   return "-";
		return Math.floor(val);
	}


	function update({clientX, clientY}) {
		position = {clientX, clientY}
		const x = clientX - canvas.offsetLeft;
		const y = clientY - canvas.offsetTop;
		freq = (maxFreq - minFreq) / canvas.width * x  + minFreq;
		vol = 0.00001 + 1 - (y / canvas.height);
		setGain(vol);
		oscillator.frequency.exponentialRampToValueAtTime(freq, deadline());
		ctx.clearRect(0, 0, canvas.width, canvas.height);
		ctx.fillRect(x, y, 10, 10);
	}


	function setGain(val) {
		val = Math.min(Math.max(0, val), 1);
		if (val === 0) {
			gain.gain.exponentialRampToValueAtTime(0.001, deadline());
			gain.gain.value = 0;
			return;
		}
		if (gain.gain <= 0)
			gain.gain.value = 0.0001;
		gain.gain.exponentialRampToValueAtTime(val, deadline());
	}

	function onMouseDown(e) {
		if (!audioCtx)
			audioContextInit();
		draw = true;
		update(e);
	}

	function onMouseUp(e) {
		draw = false;
		if(!sustain)
			setGain(0);
	}


	function onMouseMove(e) {
		if (!draw)
			return;
		update(e);
	}

</script>

<svelte:body on:mouseup={onMouseUp} />

<div class="container">
	<h1>teremin.</h1>
	<div class="info">
		<span>{formatValue(freq)} hz</span>
		<select class="middle" name="oscType" id="oscType" bind:value={oscType}>
			<option value="sine">sine</option>
			<option value="square">square</option>
			<option value="sawtooth">sawtooth</option>
			<option value="triangle">triangle</option>
		</select>
		<label><input type="checkbox" name="sustain" bind:checked={sustain}/> sustain</label>
		<span>{formatValue(vol * 100)} %</span>
	</div>	 
	<canvas
		on:mousedown={onMouseDown}
		on:mousemove={onMouseMove}
		class="canvas"
		bind:this={canvas}
		width="640"
		height="480"
	/>
</div>


<style>

	.container {
		display: flex;
		align-items: center;
		flex-direction: column;
	}
	.info {
		width: 640px;
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		min-height: 2em;
		align-items: center;
	}


	canvas {
		background: #eee;
		border: solid 1px black;
	}
</style>