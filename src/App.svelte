<script>
    import Tree from './Tree.svelte'
    import DisplaySlider from './DisplaySlider.svelte'
    import CanvasTree from './CanvasTree.svelte'
    export let name;
    let ang1 = -45;
    let ang2 = 45;
    let scale1 = 0.6;
    let scale2 = 0.6;
</script>
<style>
    :global(body) {
        margin: 0;
        padding: 0;
        overflow-y: hidden;
    }
    .slider-container {
        display: flex;
        flex-wrap: wrap;
        padding: 1em;
        padding-top: 0;
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


<!-- 
<Tree {ang1} {ang2} {scale1} {scale2} />
-->
<div class="container">
    <div class="canvas-container">
        <CanvasTree {ang1} {ang2} {scale1} {scale2} />
    </div>
    <div class="controls-container">
        <div class="slider-container">
            <DisplaySlider label="Angle 1" bind:value={ang1} min="-180" max="180" step="5"/>
            <DisplaySlider label="Scale 1" bind:value={scale1} min="0" max="0.9" step="0.02"/>
        </div>

        <div class="slider-container">
            <DisplaySlider label="Angle 2" bind:value={ang2} min="-180" max="180" step="5"/>
            <DisplaySlider label="Scale 2" bind:value={scale2} min="0" max="0.9" step="0.02"/>
        </div>
    </div>
</div>
