<script>
    import DisplaySlider from './DisplaySlider.svelte'
    import CanvasTree from './CanvasTree.svelte'
    export let name;
    let ang1 = -45;
    let ang2 = 45;
    let scale1 = 0.6;
    let scale2 = 0.6;
</script>
<style>
    :global(body) :global(html){
        margin: 0;
        padding: 0;
    }
    .slider-container {
        display: flex;
        flex-wrap: wrap;
        padding: 0 1em 1em;
    }
    .container {
        height: 100%;
        display: flex;
        flex-direction: column;
    }
    .canvas-container {
        flex: 1;
    }
</style>

<div class="container">
    <div class="canvas-container">
        <CanvasTree {ang1} {ang2} {scale1} {scale2} />
    </div>
    <div class="controls-container">
        <div class="slider-container">
            <DisplaySlider label="Angle 1" bind:value={ang1} min="-180" max="180" step="5"/>
            <DisplaySlider label="Scale 1" bind:value={scale1} min="0" max="1" step="0.02"/>
        </div>

        <div class="slider-container">
            <DisplaySlider label="Angle 2" bind:value={ang2} min="-180" max="180" step="5"/>
            <DisplaySlider label="Scale 2" bind:value={scale2} min="0" max="1" step="0.02"/>
        </div>
    </div>
</div>
