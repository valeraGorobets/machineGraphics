<script>
	import Segments from './Segments.svelte'

	const imgSrc = [
		// './img/sample82.jpg',
		// './img/sample7.jpg',
		'./img/sample11.jpg',
	];
</script>

<div style="display: flex">
	{#each imgSrc as src (src)}
		<Segments src={src}></Segments>
	{/each}
</div>
