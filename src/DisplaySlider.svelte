<script>
    export let label;
    export let value;
    export let min=0;
    export let max=100;
    export let step=1;
</script>
<style>
    .inner-container {
        margin: 0;
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
    <fieldset class="inner-container">
        <div class="inner-container">
            <legend>{label}</legend>
            <input class="slider" aria-label="{label} slider" type="range" bind:value={value} min="{min}" max="{max}" step="{step}"/>
            <input class="display" aria-label="{label} display" type="number" bind:value={value} min="{min}" max="{max}" step="{step}" />
        </div>
    </fieldset>
</span>
