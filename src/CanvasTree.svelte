<script context="module">
    // Variables in a module script section do not trigger a DOM update when changed
    // If these variables were in the regular script section, they would trigger an infinite loop of redraws
    let renderRunning = 0;
    let latestRender = 0;
    let canvas = null;
</script>
<script>
    import { onMount, afterUpdate } from 'svelte';

    export let ang1 = -45;
    export let ang2 = 45;
    export let scale1 = 0.6;
    export let scale2 = 0.6;

    const rootWidth = 10;
    const rootHeight = 0.2;
    const rootMargin = 0.3;

    async function redraw() {
        // Check if canvas has been initialized, not true for first call to the reactive statement that triggers on changes
        if (!canvas) return;

        // Give self next render index, max range is 9 trillion or so so this should be safe
        const renderInd = latestRender + 1;
        latestRender = renderInd;

        // Wait for critical section (rendering) to be free
        while (renderRunning !== 0) await new Promise(r => setTimeout(r, 0));
        // If another render started while waiting, exit
        if (renderInd !== latestRender) return;
        // Claim the critical section
        renderRunning = renderInd;

        // Zero out the width/height to let the parent flexbox recalculate height properly
        // Otherwise the canvas just grows indefinitely on resize, doesn't shrink properly
        // It's mega-weird
        canvas.width = 0;
        canvas.height = 0;

        // Set canvas resolution to its display resolution
        canvas.width = canvas.offsetWidth;
        canvas.height = canvas.offsetHeight;

        const width = canvas.width;
        const height = canvas.height;

        const rootBuffer = rootWidth * Math.max(scale1, scale2) / 2

        let ctx = canvas.getContext("2d");

        ctx.strokeStyle = '#000000ff';
        ctx.lineWidth = rootWidth;
        ctx.save();
        ctx.scale(1, -1);
        ctx.translate(width/2, -(1 - rootMargin) * height);
        ctx.beginPath();
        ctx.moveTo(0, 0);
        ctx.lineTo(0, rootHeight * height + rootBuffer);
        ctx.stroke();

        let back_canvas = document.createElement('canvas');
        back_canvas.width = width;
        back_canvas.height = height;
        let back_ctx = back_canvas.getContext("2d");

        for (var i = 0; i < 100; i++) {
            // Save the current tree to another canvas
            back_ctx.clearRect(0,0,width,height)
            back_ctx.drawImage(canvas, 0, 0);

            // Reset the transformation before clearing because finding the boundaries is hard 
            // Clear the main canvas so we're not drawing things on top of each other
            ctx.save();
            ctx.setTransform(1, 0, 0, 1, 0, 0);
            ctx.clearRect(0,0,width,height);
            ctx.restore();

            // Draw the root
            ctx.beginPath();
            ctx.moveTo(0, 0);
            ctx.lineTo(0, rootHeight * height + rootBuffer);
            ctx.stroke();

            // Draw first child
            ctx.save();
            ctx.scale(-1, 1)
            ctx.translate(0, rootHeight * height);
            ctx.rotate(Math.PI + (ang1 * Math.PI / 180));
            ctx.scale(scale1, scale1);
            ctx.drawImage(back_canvas, -width/2, rootMargin * height, width, -height);
            ctx.restore();

            // Draw second child
            ctx.save();
            ctx.scale(-1, 1)
            ctx.translate(0, rootHeight * height);
            ctx.rotate(Math.PI + (ang2 * Math.PI / 180));
            ctx.scale(scale2, scale2);
            ctx.drawImage(back_canvas, -width/2, rootMargin * height, width, -height);
            ctx.restore();

            // Set a timeout to free some cycles for the main thread, but let a few iterations finish first so things look smooth
            if (i > 5) {
                // Check if this thread needs to exit
                // If a new render task has been started, exit
                if (latestRender !== renderInd) {
                    break;
                }

                await new Promise(r => setTimeout(r, 20));
            }
        }
        ctx.restore();
        renderRunning = 0;
    }
    $: ang1, ang2, scale1, scale2, redraw();
    onMount(redraw);

    //afterUpdate(redraw);

</script>

<style>
    canvas {
        width: 100%;
        height: 100%;
    }
</style>

<svelte:window on:resize={redraw} />

<canvas bind:this={canvas} ></canvas>

