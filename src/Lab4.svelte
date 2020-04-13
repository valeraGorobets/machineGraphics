<script>
	import Grayscale from './Grayscale.svelte'

	const imgSrc = [
		'./img/sample82.jpg',
		'./img/sample2.jpg',
		'./img/sample7.jpg',
	];
</script>

<div style="display: flex">
    {#each imgSrc as src (src)}
		<Grayscale src={src}></Grayscale>
    {/each}
</div>
