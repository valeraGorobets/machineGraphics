<script>
    export let name;
    export let age;

    const nameCopy = name;
</script>

<style>
    div {
        border: 1px solid greenyellow;
        margin: 10px;
        width: 200px;
    }
</style>

<div>
    {name} \ {nameCopy}
    {age}
</div>
