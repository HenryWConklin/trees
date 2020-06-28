<script>
    import Branch from "./Branch.svelte"
    export let ang1 = -45;
    export let ang2 = 45;
    export let scale1 = 0.6;
    export let scale2 = 0.6;
    const width = 5;
    const height = 25;
    const color = "#443211"
</script>

<svg width="100%" 
     height="80%" 
     viewBox="-50 -100 100 100"
     xmlns="http://www.w3.org/2000/svg"
     xmlns:xlink="http://www.w3.org/1999/xlink">
    <defs>
        <g id="branch">
            <rect width={width} height={height} x={-width/2.0} y={-height} fill={color} stroke="none" />
            <path d="M{width/2.0},{-height + 0.1} a1,1 0 0,0 {-width},0 z" fill={color} stroke="none" />
        </g>
    </defs>

    <Branch {ang1} {ang2} {scale1} {scale2} />
    
</svg>
