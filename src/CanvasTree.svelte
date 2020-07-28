<script context="module">
    // Variables in a module script section do not trigger a DOM update when changed
    // If these variables were in the regular script section, they would trigger an infinite loop of redraws
    let renderRunning = 0;
    let latestRender = 0;
    let canvas = null;
</script>
<script>
    import {onMount} from 'svelte';

    export let ang1 = -45;
    export let ang2 = 45;
    export let scale1 = 0.6;
    export let scale2 = 0.6;

    const rootWidth = 10;
    const rootHeight = 0.2;
    const rootMargin = 0.3;

    function draw_tranformed_image(ctx, img, x, y, angle, scale) {
        ctx.save();
        // This reflection is necessary to get the image in the right orientation, shows up mirrored in weird ways
        // otherwise
        ctx.scale(-1, 1)
        ctx.translate(x, y);
        ctx.rotate(Math.PI - (angle * Math.PI / 180));
        ctx.scale(scale, scale);
        ctx.drawImage(img, -img.width / 2, rootMargin * img.height, img.width, -img.height);
        ctx.restore();
    }

    function refine_canvas_bounding_box(canvas, ctx, prev_bbox) {
        // getImageData gives you pixels in transformed space, which would be nice to save a few transformations,
        // but the size of the array that gets returned is in actual pixels, but the width/height values are in
        // transformed units so there's no way to figure out the actual width/height in pixels. So work in pixel space
        ctx.save()
        ctx.setTransform(1, 0, 0, 1, 0, 0);
        let imageData = ctx.getImageData(
                prev_bbox.minX,
                prev_bbox.minY,
                prev_bbox.maxX - prev_bbox.minX + 1,
                prev_bbox.maxY - prev_bbox.minY + 1
        );
        ctx.restore()
        let bbox = {minX: prev_bbox.maxX, maxX: prev_bbox.minX, minY: prev_bbox.maxY, maxY: prev_bbox.minY}

        // Iterate over pixels in the image
        for (let y = 0; y < imageData.height; y += 1) {
            for (let x = 0; x < imageData.width; x += 1) {
                // Image data is a flat array of rows x columns x [RGBA]
                const index = (y * (imageData.width) + x) * 4;

                // If current pixel is not transparent
                if (imageData.data[index + 3] !== 0) {
                    if (x + prev_bbox.minX < bbox.minX) bbox.minX = x + prev_bbox.minX;
                    if (x + prev_bbox.minX > bbox.maxX) bbox.maxX = x + prev_bbox.minX;

                    if (y + prev_bbox.minY < bbox.minY) bbox.minY = y + prev_bbox.minY;
                    if (y + prev_bbox.minY > bbox.maxY) bbox.maxY = y + prev_bbox.minY;
                }
            }
        }
        return bbox;
    }

    function transform_bbox(bbox, transform) {
        let p1 = transform.transformPoint(new DOMPoint(bbox.minX, bbox.minY))
        let p2 = transform.transformPoint(new DOMPoint(bbox.minX, bbox.maxY))
        let p3 = transform.transformPoint(new DOMPoint(bbox.maxX, bbox.minY))
        let p4 = transform.transformPoint(new DOMPoint(bbox.maxX, bbox.maxY))
        return {
            minX: Math.floor(Math.min(p1.x, p2.x, p3.x, p4.x)),
            maxX: Math.ceil(Math.max(p1.x, p2.x, p3.x, p4.x)),
            minY: Math.floor(Math.min(p1.y, p2.y, p3.y, p4.y)),
            maxY: Math.ceil(Math.max(p1.y, p2.y, p3.y, p4.y))
        }
    }

    function bbox_union(bbox1, bbox2) {
        /**
         * Find the smallest bbox that contains both of the input bboxes
         */
        return {
            minX: Math.min(bbox1.minX, bbox2.minX),
            maxX: Math.max(bbox1.maxX, bbox2.maxX),
            minY: Math.min(bbox1.minY, bbox2.minY),
            maxY: Math.max(bbox1.maxY, bbox2.maxY)
        }

    }

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
        try {
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

            const originX = width / 2
            const originY = (1 - rootMargin) * height

            const rootBuffer = rootWidth * Math.max(scale1, scale2) / 2

            let ctx = canvas.getContext("2d");
            ctx.clearRect(0, 0, width, height);

            ctx.strokeStyle = 'rgb(0,0,0)';
            ctx.lineWidth = rootWidth;

            let totalScale = 2;

            ctx.save();
            ctx.scale(1, -1);
            ctx.translate(originX, -originY);

            ctx.save()
            ctx.scale(totalScale, totalScale);
            ctx.beginPath();
            ctx.moveTo(0, 0);
            ctx.lineTo(0, rootHeight * height + rootBuffer);
            ctx.stroke();
            ctx.restore();

            let back_canvas = document.createElement('canvas');
            back_canvas.width = width;
            back_canvas.height = height;
            let back_ctx = back_canvas.getContext("2d");

            // Starter bbox for non-zero pixels in pixel-space (not transformed space)
            let bbox = {
                minX: 0,
                maxX: width,
                minY: 0,
                maxY: height
            };

            for (let i = 0; i < 100; i++) {
                // Save the current tree to another canvas
                back_ctx.clearRect(0, 0, width, height)
                back_ctx.drawImage(canvas, 0, 0);

                // Calculate a bounding box for this next iteration so we can scale things to keep the tree from clipping
                // off the edge of the canvas
                bbox = refine_canvas_bounding_box(canvas, ctx, bbox);
                // Get the base transform, the totalScale factor is technically part of it but is handled in a weird
                // way because of how it affects the children's rendering
                const base_transform = ctx.getTransform().scale(totalScale, totalScale)
                // Apply the transform applied to each of the children to the base transform, then map back to pixel
                // space by using the inverse of the base transform
                const child1_transform = base_transform
                        .translate(0, rootHeight * height)
                        // This interface uses degrees and not radians for some reason, also negated for some reason
                        .rotate(ang1)
                        .scale(scale1, scale1)
                        .multiply(base_transform.inverse())
                const child2_transform = base_transform
                        .translate(0, rootHeight * height)
                        .rotate(ang2)
                        .scale(scale2, scale2)
                        .multiply(base_transform.inverse())
                // Find a reasonable bounding box for the next iteration by combining the current box with the transformed
                // boxes of each of the children
                const child1_bbox = transform_bbox(bbox, child1_transform)
                const child2_bbox = transform_bbox(bbox, child2_transform)
                bbox = bbox_union(bbox, bbox_union(child1_bbox, child2_bbox));

                // Reset the transformation before clearing because finding the boundaries is hard
                // Clear the main canvas so we're not drawing things on top of each other
                ctx.save();
                ctx.setTransform(1, 0, 0, 1, 0, 0);
                ctx.clearRect(0, 0, width, height);
                ctx.restore();

                // Find the appropriate scale factor to put each edge of the tree at 10px away from the border
                const leftScale = (originX - 10) / (bbox.maxX - originX);
                const rightScale = (width - originX - 10) / (originX - bbox.minX);
                const topScale = (originY - 10) / (originY - bbox.minY);
                let bottomScale = (height - originY - 10) / (bbox.maxY - originY)

                // Bottom of the root is above/on the on the origin which gives a negative/infinite scale for the first
                // iteration at least. Replace this with a large number to ignore it
                if (bottomScale < 0) bottomScale = 1e100;

                // Only take scale factors below 1 (i.e. don't grow the tree) because that breaks thing like
                // refine_bounding_box (use a larger initial tree instead), and take the smallest scale factor which
                // indicates biggest change needed to keep the tree inside the canvas
                let scaleIncrement = Math.min(leftScale, rightScale, topScale, bottomScale, 1);

                // Draw the root
                ctx.save()
                ctx.scale(totalScale * scaleIncrement, totalScale * scaleIncrement);
                ctx.beginPath();
                ctx.moveTo(0, 0);
                ctx.lineTo(0, rootHeight * height + rootBuffer);
                ctx.stroke();
                ctx.restore()

                // Children need to only scale by the difference in scale from the previous iteration
                // Scaling by the full scale value causes the scaling to compound each iteration
                ctx.save();
                ctx.scale(scaleIncrement, scaleIncrement);

                // Draw first child (change the position the image is drawn to match the full scale factor, even though
                // the children themselves are only scaled by the increment)
                draw_tranformed_image(ctx, back_canvas, 0, rootHeight * height * totalScale, ang1, scale1)
                // Draw second child
                draw_tranformed_image(ctx, back_canvas, 0, rootHeight * height * totalScale, ang2, scale2)
                ctx.restore()

                totalScale *= scaleIncrement;

                // Set a timeout to free some cycles for the main thread, but let a few iterations finish first so things look smooth
                if (i > 5) {
                    await new Promise(r => setTimeout(r, 20));

                    // Check if this thread needs to exit
                    // If a new render task has been started, exit
                    if (latestRender !== renderInd) {
                        break;
                    }
                }
            }
            ctx.restore();
        } finally {
            renderRunning = 0;
        }
    }

    // Redraw if any parameters update
    $: ang1, ang2, scale1, scale2, redraw();
    // Draw once the component is mounted and the canvas is bound properly
    onMount(redraw);

</script>

<style>
    canvas {
        width: 100%;
        height: 100%;
    }
</style>

<svelte:window on:resize={redraw}/>

<canvas bind:this={canvas}></canvas>

