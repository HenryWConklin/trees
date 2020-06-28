<script>
    import Tree from './Tree.svelte'
    import DisplaySlider from './DisplaySlider.svelte'
    let ang1 = -45;
    let ang2 = 45;
    let scale1 = 0.6;
    let scale2 = 0.6;
</script>
<style>
    .slider-container {
        display: flex;
        margin: 2em;
    }
</style>


<Tree {ang1} {ang2} {scale1} {scale2} />
<div class="slider-container">
    <DisplaySlider label="Angle 1" bind:value={ang1} min="-180" max="180" step="5"/>
    <DisplaySlider label="Scale 1" bind:value={scale1} min="0" max="0.9" step="0.02"/>
</div>

<div class="slider-container">
    <DisplaySlider label="Angle 2" bind:value={ang2} min="-180" max="180" step="5"/>
    <DisplaySlider label="Scale 2" bind:value={scale2} min="0" max="0.9" step="0.02"/>
</div>
