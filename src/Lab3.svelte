<!--Аппроксимация кривых (Эрмита, Безье или сплайн-функции)-->
<!--https://habr.com/ru/post/247235/-->
<script>
	import { onMount } from 'svelte';

	let chartCanvas;
	const graphicPoints = [
		{ x: 0, y: 200},
		{ x: 50, y: 300},
		{ x: 100, y: 100},
		{ x: 150, y: 170},
		{ x: 200, y: 120},
		{ x: 250, y: 250},
		{ x: 300, y: 300},
		{ x: 350, y: 50},
		{ x: 400, y: 100},
		{ x: 450, y: 200},
		{ x: 500, y: 150},
		{ x: 550, y: 300},
		{ x: 600, y: 120},
		{ x: 650, y: 200},
		{ x: 700, y: 230},
		{ x: 750, y: 200}
	];
	onMount(() => {
		const context = chartCanvas.getContext('2d');
		drawAxis(context);
		drawInitChart(context);
		drawGraphicUsingBezier(context);
	});

	function drawAxis(context) {
		context.beginPath();
		context.moveTo(0, 0);
		context.lineTo(0, 400);
		context.moveTo(0, 200);
		context.lineTo(800, 200);
		graphicPoints.forEach(({ x }) => {
			context.moveTo(x, 190);
			context.lineTo(x, 210);
		});
		context.lineWidth = 2;
		context.strokeStyle = 'black';
		context.stroke();
	}

	function drawInitChart(context) {
		context.beginPath();
		context.moveTo(graphicPoints[0].x, graphicPoints[0].y);
		for (let i = 1; i < graphicPoints.length; i++) {
			context.lineTo(graphicPoints[i].x, graphicPoints[i].y);
		}
		context.stroke();
	}

	function drawGraphicUsingBezier(context) {
		const getK = (prev, current, next) => {
			const leftDistance = current.y - prev.y;
			const rightDistance = current.y - next.y;
			if (next.y - prev.y) {
				return (Math.sqrt((xStretchSqr + leftDistance ** 2) * (xStretchSqr + rightDistance ** 2)) - xStretchSqr - leftDistance * rightDistance) / (xStretch * (next.y - prev.y));
			} else {
				return 0;
			}
		};
		const getDeltaX = (k) => xStretch / 2 * Math.sqrt(1 / (1 + k * k));

		const getLeftControlPoint = (current, k,) => {
			const deltaX = getDeltaX(k);
			return { x: current.x - deltaX, y: current.y - k * deltaX };
		};
		const getRightControlPoint = (current, k) => {
			const deltaX = getDeltaX(k);
			return { x: current.x + deltaX, y: current.y + k * deltaX };
		};

		const xStretch = 50;
		const xStretchSqr = xStretch * xStretch;
		let current = graphicPoints[0], rightControlPoint;

		context.beginPath();
		context.moveTo(current.x, current.y);

		for (let i = 1; i < graphicPoints.length; i++) {
			let prev = current, k, leftControlPoint;
			const isLastTurn = i === graphicPoints.length - 1;
			current = graphicPoints[i];

			if (!isLastTurn) {
				k = getK(prev, current, graphicPoints[i + 1]);
				leftControlPoint = getLeftControlPoint(current, k);
			}

			if (i === 1) {
				context.quadraticCurveTo(leftControlPoint.x, leftControlPoint.y, current.x, current.y);
			} else if (isLastTurn) {
				context.quadraticCurveTo(rightControlPoint.x, rightControlPoint.y, current.x, current.y);
			} else {
				context.bezierCurveTo(rightControlPoint.x, rightControlPoint.y, leftControlPoint.x, leftControlPoint.y, current.x, current.y);
			}

			rightControlPoint = getRightControlPoint(current, k);
		}
		context.lineWidth = 2;
		context.strokeStyle = 'red';
		context.stroke();
	}
</script>

<canvas bind:this={chartCanvas} width='800' height='400'></canvas>

