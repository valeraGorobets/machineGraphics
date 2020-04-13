<!--
	Построение гистограммы изображения.
	Преобразование цветного изображения в полутоновое.
	Бинаризация полутоновых изображений
	Устранение шумов на бинарном изображении.
	Устранение шумов на полутоновом изображении (усредняющий и медианный фильтр)
	Выделение границ объектов на бинарном изображении.
	Выделение границ объектов на полутоновом изображении
-->
<!--https://en.wikipedia.org/wiki/Otsu%27s_method-->
<!--https://habr.com/ru/post/128768/-->
<!--https://habr.com/ru/company/gilalgorithms/blog/67594/-->

<script>
	export let src;
	import { onMount } from 'svelte';

	const imgWidth = 500;
	const imgHeight = 300;

	const times = 1;

	let img, histogramCanvas, grayScaleCanvas, binaryCanvas, medianCanvas, averageCanvas, grayScaleBorderCanvas, binaryBorderCanvas, threshold = 30;
	onMount(() => setTimeout(() => processImage(), 100));

	function processImage() {
		setImageDimensions();
		const imageData = getImageData(binaryCanvas.getContext('2d'));
		const histogram = getHistogram(imageData);
		const grayscaleImage = getGrayscaleImage(imageData);
		let averageReductionImage = getNoiseReductionAverageMethod(grayscaleImage);

		for(let i = 1; i < times ; i++) {
			averageReductionImage = getNoiseReductionAverageMethod(averageReductionImage);
		}

		const threshold = calculateThreshold(histogram, imageData.length / 4);
		const binaryImage = getBinaryImage(grayscaleImage, threshold);

		let medianReductionImage = getNoiseReductionMedianMethod(binaryImage);
		for(let i = 1; i < times ; i++) {
			medianReductionImage = getNoiseReductionMedianMethod(medianReductionImage);
		}
		calculateGrayscaleImageWithBorders();
		const binaryImageWithBorder = getBinaryImageWithBorder(medianReductionImage);

		drawHistogram(histogramCanvas, histogram);
		drawOnCanvas(grayScaleCanvas, grayscaleImage);
		drawOnCanvas(binaryCanvas, binaryImage);
		drawOnCanvas(medianCanvas, medianReductionImage);
		drawOnCanvas(averageCanvas, averageReductionImage);
		drawOnCanvas(binaryBorderCanvas, binaryImageWithBorder);
	}

	function calculateGrayscaleImageWithBorders() {
		const imageData = getImageData(grayScaleBorderCanvas.getContext('2d'));
		const grayscaleImage = getGrayscaleImage(imageData);
		let averageReductionImage = getNoiseReductionAverageMethod(grayscaleImage);

		for(let i = 1; i < times ; i++) {
			averageReductionImage = getNoiseReductionAverageMethod(averageReductionImage);
		}
		const grayscaleImageWithBorder = getGrayscaleImageWithBorder(averageReductionImage);
		drawOnCanvas(grayScaleBorderCanvas, grayscaleImageWithBorder);
	}

	function getImageData(context) {
		context.drawImage(img, 0, 0, imgWidth, imgHeight);
		const data = new Uint8ClampedArray(context.getImageData(0, 0, imgWidth, imgHeight).data.buffer);
		return data.map(d => d & 0xFF);
	}

	function getHistogram(src) {
		const histogram = new Array(256).fill(0);
		for (let i = 0; i < src.length; i += 4) {
			const r = src[i];
			const g = src[i + 1];
			const b = src[i + 2];
			histogram[r]++;
			histogram[g]++;
			histogram[b]++;
		}
		return histogram.map(e => e / 3);
	}

	function getGrayscaleImage(src) {
		const imageData = [];
		for (let i = 0; i < src.length; i += 4) {
			const r = src[i];
			const g = src[i + 1];
			const b = src[i + 2];
			// const gray = 0.2125 * r + 0.7154 * g + 0.0721 * b;
			const gray = (r + g + b) / 3;
			imageData.push(...[gray, gray, gray, src[i + 3]]);
		}
		return imageData;
	}

	function calculateThreshold(histogram, pixelAmount) {
		// Otsu's method
		let sum = 0;
		for (let i = 0; i < 256; i++) {
			sum += i * histogram[i];
		}

		let sumBackground = 0,
				frequencyBackground = 0,
				maxDispersion = 0,
				threshold = 0;

		for (let i = 0; i < 256; i++) {
			frequencyBackground += histogram[i];
			const frequencyForeground = pixelAmount - frequencyBackground;
			sumBackground += i * histogram[i];
			const meanBackground = sumBackground / frequencyBackground;            // Mean Background
			const meanForeground = (sum - sumBackground) / frequencyForeground;    // Mean Foreground
			const interClassDispersion = frequencyBackground * frequencyForeground * (meanBackground - meanForeground) ** 2;

			if (interClassDispersion > maxDispersion) {
				maxDispersion = interClassDispersion;
				threshold = i;
			}
		}

		return threshold;
	}

	function getBinaryImage(imageData, threshold) {
		const result = [];
		for (let i = 0; i < imageData.length; i += 4) {
			const pixel = imageData[i] >= threshold ? 255 : 0;
			result.push(...[pixel, pixel, pixel, 255]);
		}
		return result;
	}

	function getNoiseReductionAverageMethod(imageData) {
		const result = [];
		const width = imgWidth * 4;

		for (let i = 0; i < imageData.length; i += 4) {
			const pixel = imageData[i];
			if (i < width + 4 || i >= imageData.length - width - 4){
				result.push(...[pixel, pixel, pixel, 255]);
			} else {
				let neighbourPixels = [
					imageData[i - width - 4], imageData[i - width], imageData[i - width - 8],
					imageData[i - 4],				pixel,				imageData[i + 4],
					imageData[i + width - 4], imageData[i + width], imageData[i + width + 4]
				];
				let smoothedValue = getSmoothedValue(neighbourPixels, average);
				result.push(...[smoothedValue, smoothedValue, smoothedValue, 255]);
			}
		}
		return result;
	}

	function getNoiseReductionMedianMethod(imageData) {
		const result = [];
		const width = imgWidth * 4;

		for (let i = 0; i < imageData.length; i += 4) {
			const pixel = imageData[i];
			if (i < width + 4 || i >= imageData.length - width - 4){
				result.push(...[pixel, pixel, pixel, 255]);
			} else {
				let neighbourPixels = [
					imageData[i - width - 4], imageData[i - width], imageData[i - width - 8],
					imageData[i - 4],				pixel,				imageData[i + 4],
					imageData[i + width - 4], imageData[i + width], imageData[i + width + 4]
				];
				let smoothedValue = getSmoothedValue(neighbourPixels, median);
				neighbourPixels.splice(4, 1);
				neighbourPixels.sort();
				if (pixel === neighbourPixels[0]) {
					smoothedValue = neighbourPixels[1];
				} else if (pixel === neighbourPixels[8]) {
					smoothedValue = neighbourPixels[7];
				}

				result.push(...[smoothedValue, smoothedValue, smoothedValue, 255]);
			}
		}
		return result;
	}

	function getSmoothedValue(pixels, func) {
		const M1 = func([pixels[4], pixels[1], pixels[7]]);
		const M2 = func([pixels[4], pixels[3], pixels[5]]);
		const M3 = func([pixels[4], pixels[2], pixels[6]]);
		const M4 = func([pixels[4], pixels[0], pixels[8]]);

		const Ma = func([pixels[4], M1, M2]);
		const Mb = func([pixels[4], M3, M4]);

		return func([pixels[4], Ma, Mb]);
	}

	function median(pixels) {
		const mid = Math.floor(pixels.length / 2);
		pixels = [...pixels].sort((a, b) => a - b);
		return pixels.length % 2 !== 0 ? pixels[mid] : (pixels[mid - 1] + pixels[mid]) / 2;
	}

	function average(pixels) {
		return pixels.reduce((a,b) => a + b, 0) / pixels.length;
	}

	function getGrayscaleImageWithBorder(imageData) {
		const result = [];
		const width = imgWidth * 4;
		for (let i = 4; i < imageData.length - 4; i += 4) {
			const pixel = imageData[i];
			if ((pixel > imageData[i - 4] + threshold) ||
					(pixel<imageData[i - 4]-threshold) ||
					(pixel>imageData[i + 4]+threshold) ||
					(pixel<imageData[i + 4]-threshold) ||
					(pixel>imageData[i - width]+threshold) ||
					(pixel<imageData[i - width]-threshold) ||
					(pixel>imageData[i + width]+threshold) ||
					(pixel<imageData[i + width]-threshold)
			){
				result.push(...[0, 0, 0, 255]);
			} else {
				result.push(...[255, 255, 255, 255]);
			}
		}
		return result;
	}

	function getBinaryImageWithBorder(imageData) {
		const result = [];
		const width = imgWidth * 4;
		for (let i = 4; i < imageData.length - 4; i += 4) {
			const pixel = imageData[i];
			let neighbourPixels = [
				imageData[i - width - 4], imageData[i - width], imageData[i - width - 8],
				imageData[i - 4],				pixel,				imageData[i + 4],
				imageData[i + width - 4], imageData[i + width], imageData[i + width + 4]
			];
			if (neighbourPixels.some(p => p !== pixel)) {
				result.push(...[0, 0, 0, 255]);
			} else {
				result.push(...[255, 255, 255, 255]);
			}
		}
		return result;
	}

	function drawHistogram(canvas, histBrightness) {
		const context = canvas.getContext('2d');
		const maxBrightness = Math.max(...histBrightness);

		const startYPosition = histogramCanvas.height;
		const tickWidth = histogramCanvas.width / 256;
		const tickHeight = startYPosition / maxBrightness;
		context.lineWidth = tickWidth;

		for (let i = 0; i < 256; i++) {
			const position = i * tickWidth;
			context.strokeStyle = "#000";
			context.beginPath();
			context.moveTo(position, startYPosition);
			context.lineTo(position, startYPosition - histBrightness[i] * tickHeight);
			context.closePath();
			context.stroke();
		}
	}

	function drawOnCanvas(canvas, grayscaleImage) {
		const context = canvas.getContext('2d');
		const imageData = context.createImageData(imgWidth, imgHeight);
		for (let i = 0; i < imageData.data.length; i++) {
			imageData.data[i] = grayscaleImage[i];
		}
		context.putImageData(imageData, 0, 0);
	}

	function setImageDimensions() {
		img.width = imgWidth;
		img.height = imgHeight;
	}
</script>

<style>
	.wrapper {
		display: flex;
		margin: 0 10px;
	}
	canvas {
		border: 1px solid black;
	}
</style>

<div class="wrapper">
	<div>
		<img bind:this={img} src={src} alt="">

		<h3>Построение гистограммы изображения</h3>
		<canvas bind:this={histogramCanvas} width={imgWidth} height={imgHeight}></canvas>

		<h3>Полутоновое изображение</h3>
			<canvas bind:this={grayScaleCanvas} width={imgWidth} height={imgHeight}></canvas>

			<h3>Устранение шумов: усредняющий фильтр</h3>
			<canvas bind:this={averageCanvas} width={imgWidth} height={imgHeight}></canvas>

			Threshold: {threshold} <input type=range bind:value={threshold} on:change={calculateGrayscaleImageWithBorders} min=0 max=255>
			<h3>Выделение границ объектов на полутоновом изображении</h3>
			<canvas bind:this={grayScaleBorderCanvas} width={imgWidth} height={imgHeight}></canvas>

		<h3>Бинаризация изображения</h3>
			<canvas bind:this={binaryCanvas} width={imgWidth} height={imgHeight}></canvas>

			<h3>Устранение шумов: медианный фильтр</h3>
			<canvas bind:this={medianCanvas} width={imgWidth} height={imgHeight}></canvas>

			<h3>Выделение границ объектов на бинарном изображении</h3>
			<canvas bind:this={binaryBorderCanvas} width={imgWidth} height={imgHeight}></canvas>
	</div>
</div>
