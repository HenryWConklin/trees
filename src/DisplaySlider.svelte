<script>
    export let label;
    export let value;
    export let min=0;
    export let max=100;
    export let step=1;
</script>
<style>
    .inner-container {
        margin: 0 2em 0 1em;
        flex-grow: 1;
        display: flex;
    }
    .slider {
        flex-grow: 1;
    }
    .display {
        width: 5em;
    }
    .container {
        display: flex;
        flex-grow: 1;
    }
</style>

<span class="container">
<label for="slider">{label}</label>
<div class="inner-container">
    <input id="slider" class="slider" type="range" bind:value={value} min="{min}" max="{max}" step="{step}"/>
    <input id="display" class="display" type="number" bind:value={value} min="{min}" max="{max}" step="{step}" />
</div>
</span>
