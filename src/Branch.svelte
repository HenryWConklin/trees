<script>
    export let ang1 = -45;
    export let ang2 = 45;
    export let scale1 = 0.6;
    export let scale2 = 0.6;
    export let depth=1;
    export let maxDepth=10;
    const width = 5;
    const height = 25;
    const color = "#443211"
</script>
<use xlink:href="#branch" />
{#if depth < maxDepth }
<g transform="translate(0, {-height}) scale({scale1},{scale1}) rotate({ang1})">
    <svelte:self {ang1} {ang2} {scale1} {scale2} depth={depth+1} />
</g>
<g transform="translate(0, {-height}) scale({scale2},{scale2}) rotate({ang2})">
    <svelte:self {ang1} {ang2} {scale1} {scale2} depth={depth+1} />
</g>
{/if}

