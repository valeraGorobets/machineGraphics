<!-- Сегментация изображений. Метод роста областей. Разделение и слияние областей.-->
<script>
	export let src;
	import {onMount} from 'svelte';

	const imgWidth =  320;
	const imgHeight = 320;

	let img, regionGrowingSegmentsCanvas, splitAndMergeSegmentsCanvas;
	onMount(() => setTimeout(() => processImage(), 100));

	function processImage() {
		setImageDimensions();

		const imageData = getImageData(splitAndMergeSegmentsCanvas.getContext('2d'));
		const regionGrowingSegments = getRegionGrowingSegments(imageData);
		const splitAndMergeSegments = getSplitAndMergeSegments(splitOnPixels(imageData));
		drawOnCanvas(regionGrowingSegmentsCanvas, regionGrowingSegments);

		const pixelValuesInOneRow = getPixelValuesInOneRow(splitAndMergeSegments);
		drawOnCanvas(splitAndMergeSegmentsCanvas, pixelValuesInOneRow);
	}

	function getImageData(context) {
		context.drawImage(img, 0, 0, imgWidth, imgHeight);
		const data = new Uint8ClampedArray(context.getImageData(0, 0, imgWidth, imgHeight).data.buffer);
		return data.map(d => d & 0xFF);
	}

	function splitOnPixels(imageData) {
		const res = [];
		for (let i = 0; i < imageData.length; i += 4) {
			res.push({
				r: imageData[i],
				g: imageData[i + 1],
				b: imageData[i + 2],
				a: imageData[i + 3],
			})
		}
		return res;
	}

	function getRegionGrowingSegments(src) {
		const visitedPixels = {};
		const observingQueue = [];
		const imageData = [...src];
		const pixelsAmount = src.length / 4;
		const width = imgWidth * 4;
		const threshold = 110;
		const possiblePixelPositions = Array.from(Array(src.length).keys());

		const isPixelNotChecked = index => index % 4 === 0 && index > 0 && index <= imageData.length - width - 4 && !visitedPixels[index];
		const findAvailablePixel = () => possiblePixelPositions.find((index) => isPixelNotChecked(index));
		const getNeighbourPixelToCheck = curIndex => [curIndex + 4, curIndex + width, curIndex - width, curIndex - 4].find((index) => isPixelNotChecked(index));
		const distance = (r1, r2, g1, g2, b1, b2) => Math.sqrt((r1 - r2) ** 2 + (g1 - g2) ** 2 + (b1 - b2) ** 2);

		const run = (currentPixel) => {
			visitedPixels[currentPixel] = true;
			observingQueue.push(currentPixel);

			while (observingQueue.length !== 0) {
				const neighbourPixel = getNeighbourPixelToCheck(currentPixel);
				if (neighbourPixel) {
					const r1 = src[currentPixel], r2 = src[neighbourPixel];
					const g1 = src[currentPixel + 1], g2 = src[neighbourPixel + 1];
					const b1 = src[currentPixel + 2], b2 = src[neighbourPixel + 2];
					if (distance(r1, r2, g1, g2, b1, b2) < threshold) {
						imageData[neighbourPixel] = 255 - imageData[neighbourPixel];
						imageData[neighbourPixel + 1] = 255 - imageData[neighbourPixel + 1];
						imageData[neighbourPixel + 2] = 255 - imageData[neighbourPixel + 2];
						observingQueue.push(neighbourPixel);
					}
					visitedPixels[neighbourPixel] = true;
				} else {
					observingQueue.shift();
					currentPixel = observingQueue[0];
				}
			}
		};

		while (pixelsAmount - Object.keys(visitedPixels).length > 0.05 * pixelsAmount) {
			const availablePixel = findAvailablePixel();
			if (availablePixel) {
				run(availablePixel);
			}
		}
		return imageData;
	}

	function getSplitAndMergeSegments(src) {
		const split = areas => areas.map((area, i, array) => {
			if (shouldAreaBeSplitted(area.pixels)) {
				const newAreas = splitOn4Areas(area);
				updateNeighboursAfterSplitting(array, newAreas, area);
				return split(newAreas);
			} else {
				return area;
			}
		}).flat(Infinity);

		const splitted = split([new Area('', src, 0)]);
		mergeCommonSplittedAreas(splitted);
		return src;
	}

	function shouldAreaBeSplitted(pixels) {
		const threshold = 60;
		const grayPixels = [];
		for (let i = 0; i < pixels.length; i++) {
			grayPixels.push(getGrayColor(pixels[i]));
		}
		return Math.max(...grayPixels) - Math.min(...grayPixels) >= threshold;
	}

	function getGrayColor({r, g, b}) {
		return 0.2125 * r + 0.7154 * g + 0.0721 * b;
	}

	function splitOn4Areas({id, pixels, neighbours}) {
		const rowsAndCol = Math.sqrt(pixels.length) / 2;
		const area1Pixels = getArea1Pixels(pixels, rowsAndCol);
		const area2Pixels = getArea2Pixels(pixels, rowsAndCol);
		const area3Pixels = getArea3Pixels(pixels, rowsAndCol);
		const area4Pixels = getArea4Pixels(pixels, rowsAndCol);

		const area1 = new Area(`${id}1`, area1Pixels.pixels, area1Pixels.offset);
		const area2 = new Area(`${id}2`, area2Pixels.pixels, area2Pixels.offset);
		const area3 = new Area(`${id}3`, area3Pixels.pixels, area3Pixels.offset);
		const area4 = new Area(`${id}4`, area4Pixels.pixels, area4Pixels.offset);

		setNeighbours(area1, area2, area3, area4, id, neighbours);
		return [area1, area2, area3, area4];
	}

	function getArea1Pixels(pixels, rowsAndCol) {
		const area = [];
		for (let i = 0; i < rowsAndCol; i++) {
			for (let j = 0; j < rowsAndCol; j++) {
				const pixInd = i * (rowsAndCol * 2) + j;
				area.push(pixels[pixInd]);
			}
		}
		return {
			pixels: area,
			offset: 0,
		};
	}

	function getArea2Pixels(pixels, rowsAndCol) {
		const area = [];
		for (let i = 0; i < rowsAndCol; i++) {
			for (let j = rowsAndCol; j < rowsAndCol * 2; j++) {
				const pixInd = i * (rowsAndCol * 2) + j;
				area.push(pixels[pixInd]);
			}
		}
		return {
			pixels: area,
			offset: rowsAndCol,
		};
	}

	function getArea3Pixels(pixels, rowsAndCol) {
		const area = [];
		for (let i = rowsAndCol; i < rowsAndCol * 2; i++) {
			for (let j = 0; j < rowsAndCol; j++) {
				const pixInd = i * (rowsAndCol * 2) + j;
				area.push(pixels[pixInd]);
			}
		}
		return {
			pixels: area,
			offset: rowsAndCol * (rowsAndCol * 2),
		};
	}

	function getArea4Pixels(pixels, rowsAndCol) {
		const area = [];
		for (let i = rowsAndCol; i < rowsAndCol * 2; i++) {
			for (let j = rowsAndCol; j < rowsAndCol * 2; j++) {
				const pixInd = i * (rowsAndCol * 2) + j;
				area.push(pixels[pixInd]);
			}
		}
		return {
			pixels: area,
			offset: rowsAndCol * (rowsAndCol * 2) + rowsAndCol,
		};
	}

	function setNeighbours(area1, area2, area3, area4, id, neighbours) {
		area1.neighbours.push(area2, area3);
		area2.neighbours.push(area1, area4);
		area3.neighbours.push(area1, area4);
		area4.neighbours.push(area2, area3);
		const idEndsWith = id.slice(-1);
		const ne1 = neighbours.find((r => r.id.endsWith('1')));
		const ne2 = neighbours.find((r => r.id.endsWith('2')));
		const ne3 = neighbours.find((r => r.id.endsWith('3')));
		const ne4 = neighbours.find((r => r.id.endsWith('4')));

		if (idEndsWith === '1') {
			if (ne1) {
				area1.neighbours.push(ne1);
				area3.neighbours.push(ne1);
			}
			if (ne2) {
				area2.neighbours.push(ne2);
				area4.neighbours.push(ne2);
			}
			if (ne3) {
				area3.neighbours.push(ne3);
				area4.neighbours.push(ne3);
			}
			if (ne4) {
				area1.neighbours.push(ne4);
				area2.neighbours.push(ne4);
			}
		} else if (idEndsWith === '2') {
			if (ne1) {
				area1.neighbours.push(ne1);
				area3.neighbours.push(ne1);
			}
			if (ne2) {
				area2.neighbours.push(ne2);
				area4.neighbours.push(ne2);
			}
			if (ne3) {
				area1.neighbours.push(ne3);
				area2.neighbours.push(ne3);
			}
			if (ne4) {
				area3.neighbours.push(ne4);
				area4.neighbours.push(ne4);
			}
		} else if (idEndsWith === '3') {
			if (ne1) {
				area3.neighbours.push(ne1);
				area4.neighbours.push(ne1);
			}
			if (ne2) {
				area1.neighbours.push(ne2);
				area3.neighbours.push(ne2);
			}
			if (ne3) {
				area1.neighbours.push(ne3);
				area2.neighbours.push(ne3);
			}
			if (ne4) {
				area3.neighbours.push(ne4);
				area4.neighbours.push(ne4);
			}
		} else if (idEndsWith === '4') {
			if (ne1) {
				area3.neighbours.push(ne1);
				area4.neighbours.push(ne1);
			}
			if (ne2) {
				area1.neighbours.push(ne2);
				area2.neighbours.push(ne2);
			}
			if (ne3) {
				area1.neighbours.push(ne3);
				area3.neighbours.push(ne3);
			}
			if (ne4) {
				area2.neighbours.push(ne4);
				area4.neighbours.push(ne4);
			}
		}
	}

	function updateNeighboursAfterSplitting(allAreas, splitted, splittedArea) {
		allAreas.forEach((area) => {
			let index = area.neighbours.findIndex(n => n.id === splittedArea.id);
			if (index > -1) {
				const areaId = area.id.slice(-1);
				const splittedAreaId = splittedArea.id.slice(-1);
				if (splittedAreaId === '1' && areaId === '2') {
					area.neighbours.splice(index, 1, splitted[1], splitted[3]);
				} else if (splittedAreaId === '1' && areaId === '3') {
					area.neighbours.splice(index, 1, splitted[2], splitted[3]);
				} else if (splittedAreaId === '2' && areaId === '1') {
					area.neighbours.splice(index, 1, splitted[0], splitted[2]);
				} else if (splittedAreaId === '2' && areaId === '4') {
					area.neighbours.splice(index, 1, splitted[2], splitted[3]);
				} else if (splittedAreaId === '3' && areaId === '1') {
					area.neighbours.splice(index, 1, splitted[2], splitted[3]);
				} else if (splittedAreaId === '3' && areaId === '4') {
					area.neighbours.splice(index, 1, splitted[0], splitted[2]);
				} else if (splittedAreaId === '4' && areaId === '2') {
					area.neighbours.splice(index, 1, splitted[2], splitted[3]);
				} else if (splittedAreaId === '4' && areaId === '3') {
					area.neighbours.splice(index, 1, splitted[1], splitted[3]);
				}
			}
		})
	}

	function mergeCommonSplittedAreas(splitted) {
		const observingQueue = [splitted[0]];

		while (observingQueue.length !== 0) {
			const area = observingQueue[0];
			observingQueue.shift();

			if (area.isVisited) {
				continue;
			}
			area.neighbours.forEach((neighbour => {
				if (!neighbour.isVisited && shouldAreasBeMerged(area.pixels, neighbour.pixels)) {
					const color = area.colorForMerging || neighbour.colorForMerging || {
						r: 255 - area.pixels[0].r,
						g: 255 - area.pixels[0].g,
						b: 255 - area.pixels[0].b,
					};
					colorArea(area.pixels, color);
					colorArea(neighbour.pixels, color);
					[area, neighbour].forEach(a => {
						if (!a.colorForMerging) {
							a.colorForMerging = color;
						}
					});
				}
			}));
			area.isVisited = true;
			observingQueue.push(...area.neighbours);
		}
	}

	function shouldAreasBeMerged(area1, area2) {
		return !shouldAreaBeSplitted([...area1, ...area2]);
	}

	function colorArea(pixels, {r, g, b}) {
		for (let i = 0; i < pixels.length; i++) {
			pixels[i].r = r;
			pixels[i].g = g;
			pixels[i].b = b;
		}
	}

	function drawOnCanvas(canvas, image) {
		const context = canvas.getContext('2d');
		const imageData = context.createImageData(imgWidth, imgHeight);
		for (let i = 0; i < imageData.data.length; i++) {
			imageData.data[i] = image[i];
		}
		context.putImageData(imageData, 0, 0);
	}

	function getPixelValuesInOneRow(image) {
		const res = [...image];
		for (let i = 0; i < image.length; i++) {
			const {r, g, b, a} = image[i];
			res.splice(i * 4, 1, r, g, b, a);
		}
		return res;
	}

	function setImageDimensions() {
		img.width = imgWidth;
		img.height = imgHeight;
	}

	class Area {
		constructor(id, pixels = [], offset, neighbours = []) {
			this.id = id;
			this.pixels = pixels;
			this.offset = offset;
			this.neighbours = neighbours;
			this.isColored = false;
			this.isVisited = false;
			this.colorForMerging = null;
		}
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

		<h3>Сегментация методом роста областей</h3>
		<canvas bind:this={regionGrowingSegmentsCanvas} width={imgWidth} height={imgHeight}></canvas>
		<h3>Сегментация методом разделения и слияния областей</h3>
		<canvas bind:this={splitAndMergeSegmentsCanvas} width={imgWidth} height={imgHeight}></canvas>
	</div>
</div>
