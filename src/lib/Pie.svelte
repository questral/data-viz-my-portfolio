
<script>
    import * as d3 from 'd3';

    // let arc = arcGenerator({
    //     startAngle: 0,
    //     endAngle: 2 * Math.PI
    // });

    // let data = [1, 2, 3, 4, 5, 5];
    // let data = [
    //     { value: 1, label: "apples" },
    //     { value: 2, label: "oranges" },
    //     { value: 3, label: "mangos" },
    //     { value: 4, label: "pears" },
    //     { value: 5, label: "limes" },
    //     { value: 5, label: "cherries" }
    // ];
    let { data = [], selectedIndex = $bindable(-1) } = $props();
    // let colors = ['gold', 'purple'];
    let colors = d3.scaleOrdinal(d3.schemeTableau10);
    
    let arcGenerator = d3.arc().innerRadius(0).outerRadius(50);

    // let total = 0;
    // for (let d of data) {
    //     total += d;
    // }

    // let angle = 0;
    // let arcData = [];
    // for (let d of data) {
    //     let endAngle = angle + (d / total) * 2 * Math.PI;
    //     arcData.push({ startAngle: angle, endAngle });
    //     angle = endAngle;
    // }
    // let arcs = arcData.map(d => arcGenerator(d));

    // let sliceGenerator = d3.pie();
    let sliceGenerator = d3.pie().value(d => d.value);

    // let arcData = sliceGenerator(data);
    // let arcs = arcData.map(d => arcGenerator(d));
    const arcData = $derived(sliceGenerator(data));
    const arcs = $derived(arcData.map(d => arcGenerator(d)));

    // let { selectedIndex = $bindable(-1) } = $props();
    // let selectedIndex = $state(-1);

    function toggleWedge (index, event) {
        if (!event.key || event.key === "Enter") {
            selectedIndex = selectedIndex === index ? -1 : index;
            // selectedIndex = index;
        }
    }

</script>

pie time

<div class="container">
    <svg viewBox='-50 -50 100 100'>
        <!-- <circle cx='0' cy='0' r='50' fill='red' /> -->
        {#each arcs as arc, index}
            <!-- <path d={arc} fill={colors[i]} /> -->
            <path d={arc} fill={ colors(index) } tabindex="0" role="button" aria-label="{data[index].label}" style="
                --start-angle: { arcData[index]?.startAngle }rad;
                --end-angle: { arcData[index]?.endAngle }rad;"

                class:selected={selectedIndex === index}
                onclick={e => toggleWedge(index, e)}
                onkeyup={e => toggleWedge(index, e)} />
        {/each}
    </svg>

    <ul class="legend">
        {#each data as d, index}
            <li style="--color: { colors(index) }">
                <span class="swatch" class:selected={selectedIndex === index}></span>
                {d.label} <em>({d.value})</em>
            </li>
        {/each}
    </ul>
</div>


<style>
    svg {
        max-width: 20em;
        margin-block: 2em;
        overflow: visible;
    }

    svg:has(path:hover, path:focus-visible) {
        path:not(:hover, :focus-visible) {
            opacity: 50%;
        }

        /* path {
            transition: 300ms;
            cursor: pointer;
        } */
    }

    path {
        transition: 300ms;
        cursor: pointer;
        outline: none;

        --angle: calc(var(--end-angle) - var(--start-angle));
        --mid-angle: calc(var(--start-angle) + var(--angle) / 2);

        &.selected {
            transform: rotate(var(--mid-angle))
	                   translateY(-6px) scale(1.1)
	                   rotate(calc(-1 * var(--mid-angle)));
        }

        transform: rotate(var(--mid-angle))
	               translateY(0)
	               rotate(calc(-1 * var(--mid-angle)));
    }

    .selected {
        --color: oklch(60% 45% 0) !important;

        &:is(path) {
            fill: var(--color);
        }
    }

    .container {
        display: flex;
        align-items: start;
        gap: 2em;
    }

    ul {
        display: grid;
        grid-template-rows: 1;
        grid-template-columns: repeat(auto-fill, minmax(9em, 1fr));
        gap: 0.3em;
        margin: 1em;
        margin-top: 3em;
        padding: 1em;
        border: 1px solid gray;
        flex: 1;

        li {
            display: flex;
            align-items: center;
            gap: 0.5em;
        }
    }

    .swatch {
        width: 1em;
        height: 1em;
        /* aspect-ratio: 1 / 1; */
        display: inline-block;
        background: var(--color);
    }
</style>