<script>
	import Grayscale from './Grayscale.svelte'

	const imgSrc = [
		'./img/face.jpg',
		'./img/sample2.jpg',
		'./img/sample13.jpg',
	];
</script>

<div style="display: flex">
    {#each imgSrc as src (src)}
		<Grayscale src={src}></Grayscale>
    {/each}
</div>
