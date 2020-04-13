<script>

	import Person from './Person.svelte';

	let name = 'hello';
	const number = 45;
	const src = 'https://www.himgs.com/imagenes/hello/social/hello-fb-logo.png';
	const htmlText = '<strong>I AM STRONG</strong>';
	let click = false;

	let counter = 0;
	let nameToAdd = '';

	const positions = {
		x: 0,
		y: 0,
	};

	$: buttonClass = counter % 2 === 0 ? 'red' : 'blue';
	$: if(counter % 5 === 0) {
		buttonClass = 'green';
	}

	let value = 'input';

	let people = [
		{age: 10, name: 'aaa'},
		{age: 20, name: 'bbb'},
		{age: 30, name: 'ccc'},
	]

	function onMove(event) {
		positions.x = event.x;
		positions.y = event.y;
	}
	function addNewTodo() {
		const newTodo = {
			age: 40,
			name: nameToAdd,
		};
		// people.push(newTodo);
		people = [
			...people,
			newTodo,
		]
	}

	function removeFirst() {
		people = people.slice(1);
	}
</script>

<style>
	img {
		width: 100px;
	}

	.red {
		color: red;
	}

	.blue {
		color: blue;
	}

	.green {
		color: green;
	}

	.playground {
		width: 300px;
		height: 200px;
		border: 2px solid black;
	}
</style>

<h1>Hello {name}!</h1>
<h1>{number}</h1>
<img {src} alt="" >
<p>{htmlText}</p>
<p>{
	@html htmlText
}</p>

<div class="playground"
	 on:mousemove={onMove}
	 on:mousedown|once={() => click = true}
	 on:mouseup={() => click = false}
>
	<h1>X: {positions.x} - Y: {positions.y}</h1>
	<h2>{click}</h2>
</div>

<h1 class={buttonClass}>Counter: {counter}</h1>
<button on:click={() => counter++}>Inc</button>

<div>
	<input bind:value>
	{#if value.length < 3}
		Too short value
	{:else if value.length > 10}
		Too long value
	{/if}
</div>

{#each people as person}
	<Person {...person}></Person>
{/each}


<input bind:value={nameToAdd}>
<button on:click={addNewTodo}>Add todo</button>
<button on:click={removeFirst}>Remove first</button>
{#each people as {name, age} (name)}
	<Person {name} {age}></Person>
	{:else}
		No people
{/each}
