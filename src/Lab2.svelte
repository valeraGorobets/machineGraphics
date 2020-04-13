<!--Закрашивание многоугольников, Отсечение-->
<script>
	import { onMount } from 'svelte';

	let lineCanvas, clipCanvas;
	onMount(() => {
		drawStar();
		drawIntersection();
	});

	function drawStar() {
		const coordinates = [
			[200, 100],
			[230, 160],
			[300, 160],
			[250, 200],
			[270, 260],
			[200, 220],
			[135, 260],
			[150, 200],
			[100, 160],
			[170, 160]
		];
		const context = lineCanvas.getContext('2d');
		context.beginPath();
		context.moveTo(coordinates[0][0], coordinates[0][1]);
		for (let i = 1; i < coordinates.length; i++) {
			context.lineTo(coordinates[i][0], coordinates[i][1]);
		}
		context.fillStyle = '#FF6400';
		context.fill();
	}

	function drawIntersection() {
		const context = clipCanvas.getContext('2d');
		const figure = [[50, 250], [130, 120], [250,50], [350,200], [240,300], [90,345]];
		const window = [[100, 100], [300, 100], [300, 300], [100, 300]];
		const clipped = clipAlgorithmSutherlandHodgman(figure, window);

		drawPolygon (context, figure, '#009900','#009900');
		setTimeout(() => {
			drawPolygon (context, window, '#990000','#990000');
		}, 1000);
		setTimeout(() => {
			drawPolygon(context, clipped, '#000099','#000099');
		}, 2500);
	}

	function clipAlgorithmSutherlandHodgman(figure, window) {
		function isInside(point, pointFrom, pointTo) {
			return (pointTo[0]-pointFrom[0])*(point[1]-pointFrom[1]) > (pointTo[1]-pointFrom[1])*(point[0]-pointFrom[0]);
		}

		function intersection(windowPointFrom, windowPointTo, figurePointFrom, figurePointTo) {
			const windowsDiff = [windowPointFrom[0] - windowPointTo[0], windowPointFrom[1] - windowPointTo[1]],
					figureDiff = [figurePointFrom[0] - figurePointTo[0], figurePointFrom[1] - figurePointTo[1]],
					n1 = windowPointFrom[0] * windowPointTo[1] - windowPointFrom[1] * windowPointTo[0],
					n2 = figurePointFrom[0] * figurePointTo[1] - figurePointFrom[1] * figurePointTo[0],
					n3 = windowsDiff[0] * figureDiff[1] - windowsDiff[1] * figureDiff[0];
			return [(n1*figureDiff[0] - n2*windowsDiff[0]) / n3, (n1*figureDiff[1] - n2*windowsDiff[1]) / n3];
		}

		let outputList = figure;
		let windowPointFrom = window[window.length-1];
		window.forEach((windowPointTo) => {
			const inputList = outputList;
			outputList = [];
			let figurePointFrom = inputList[inputList.length - 1];
			inputList.forEach((figurePointTo) => {
				if (isInside(figurePointTo, windowPointFrom, windowPointTo)) {
					if (!isInside(figurePointFrom, windowPointFrom, windowPointTo)) {
						outputList.push(intersection(windowPointFrom, windowPointTo, figurePointFrom, figurePointTo));
					}
					outputList.push(figurePointTo);
				}
				else if (isInside(figurePointFrom, windowPointFrom, windowPointTo)) {
					outputList.push(intersection(windowPointFrom, windowPointTo, figurePointFrom, figurePointTo));
				}
				figurePointFrom = figurePointTo;
			});
			windowPointFrom = windowPointTo;
		});
		return outputList;
	}

	function drawPolygon (context, polygon, strokeStyle, fillStyle) {
		context.strokeStyle = strokeStyle;
		context.fillStyle = fillStyle;
		context.beginPath();

		context.moveTo(polygon[0][0],polygon[0][1]);
		for (let i = 1; i < polygon.length; i++){
			context.lineTo(polygon[i][0],polygon[i][1]);
		}
		context.lineTo(polygon[0][0],polygon[0][1]);

		context.fill();
		context.stroke();
		context.closePath();
	}
</script>

<canvas bind:this={lineCanvas} width='500' height='400'></canvas>
<canvas bind:this={clipCanvas} width="400" height="400"></canvas>
