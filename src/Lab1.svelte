<!--Рисование отрезков, окружности и эллипса-->
<script>
	import { onMount } from 'svelte';

	let lineCanvas, circleCanvas, ellipseCanvas;

	onMount(() => {
		drawLine();
		drawCircle();
		drawEllipse();
	});

	function drawLine() {
		const context = lineCanvas.getContext('2d');
		context.beginPath();
		context.moveTo(50, 50);
		context.lineTo(250, 150);
		context.stroke();
	}

	function drawCircle() {
		const context = circleCanvas.getContext('2d');
		context.lineWidth = 5;
		context.arc(200, 200, 100, 0, 2 * Math.PI,true);
		context.stroke();
	}

	function drawEllipse() {
		const context = ellipseCanvas.getContext('2d');
		context.ellipse(200, 200, 150, 100, 0, 0, Math.PI*2);
		context.fillStyle = "#00e764";
		context.fill();
	}
</script>

<canvas bind:this={lineCanvas} width='500' height='400'></canvas>
<canvas bind:this={circleCanvas} width='500' height='400'></canvas>
<canvas bind:this={ellipseCanvas} width='500' height='400'></canvas>
