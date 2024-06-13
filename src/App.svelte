<script>
  import data from "$data/data.js";
  console.log(data);

  import { forceSimulation, forceY, forceX, forceCollide } from "d3-force";
  import { scaleLinear, scaleBand, scaleOrdinal, scaleSqrt } from "d3-scale";
  import { extent } from "d3-array";
  import { each } from "svelte/internal";
  import { mean, rollups } from "d3-array";
  import { fade } from "svelte/transition"
  import { group, index } from "d3-array";
    
  const RADIUS = 5;
  const simulation = forceSimulation(data)


  let nodes = [];
  simulation.on("tick", () => {
    nodes = simulation.nodes();
  });


$: {
  simulation
  .force(
      "x",
      forceX()
        .x((d) => xScale(d.happiness))
        .strength(0.08),
    )
    .force(
      "y",
      forceY()
        .y((d) =>  groupByContinent ? yScale(d.continent) : innerHeight/2)
        .strength(0.04),
    )
    .force("collide", forceCollide().radius(d => radiusScale(d.happiness)))
    .alpha(1)
    .alphaDecay(0.0005)
    .restart()
}

  // console.log(simulation);

  let width = 400,
    height = 400;

  const margin = {
    top: 0,
    right: 0,
    bottom: 25,
    left: 0,
  };

  $: innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;


  const continents = rollups(
    data,
    v => mean(v, d => d.happiness),
    d => d.continent
  ) // Group data by continent and return the group-wide mean
    .sort((a, b) => a[1] - b[1]) // Sort according to value
    .map(d => d[0]); // Grab the continent name


  $: xScale = scaleLinear()
    // .domain(extent(data, (d) => Math.round(d.happiness)))
    .domain([1, 9])
    .range([0, innerWidth]);

  let yScale = scaleBand()
    .domain(continents)
    .range([innerHeight, 0])
    .paddingOuter(0.5);

  const colorRange = ["#dda0dd", "#fe7f2d", "#fcca46", "#a1c181", "#619b8a", "#eae2b7"];
    const colorScale = scaleOrdinal()
    .domain(continents)
    .range (colorRange)

   $: radiusScale = scaleSqrt()
      .domain([0,10])
      .range(width < 568 ? [2,6]:[3,8])

  import AxisX from "$components/AxisX.svelte";
  import AxisY from "$components/AxisY.svelte";
  import Legend from "$components/Legend.svelte";
  import Tooltip from "$components/Tooltip.svelte";

  let hovered;
  let hoveredContinent;

let groupByContinent = false
</script>

<h1>The Happiest Countries in the World</h1>
<Legend {colorScale}  bind:hoveredContinent/>
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div class="chart-container" bind:clientWidth={width}
 on:click={() => {
  groupByContinent = !groupByContinent;
  hovered = null;
 }}
 
 >
  <svg {width} {height}>

   <!-- Reference line -->
   {#if hovered}
   <line
       transition:fade
       x1={hovered.x}
       x2={hovered.x}
       y1={height - margin.bottom}
       y2={hovered.y + margin.top + radiusScale(hovered.happiness)}
       stroke={colorScale(hovered.continent)}
       stroke-width="2"
   />
{/if}
    
    <g class="inner-chart" transform="translate({margin.left}, {margin.top})"
    on:mouseleave={() => (hovered=null)}>
      <AxisX xScale={xScale} height={innerHeight} width={innerWidth}/>
      <AxisY yScale={yScale} {groupByContinent}/>


    {#each nodes as node, i}
    <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
    <circle
        cx={node.x}
        cy={node.y}
        r={radiusScale(node.happiness)}
        stroke={hovered || hoveredContinent
        ? hovered === node || hoveredContinent === node.continent
            ? "black"
            : "transparent"
        : "#00000090"}
        fill={colorScale(node.continent)}
        title={node.country}
        opacity={hovered || hoveredContinent
        ? hovered === node || hoveredContinent === node.continent
            ? 1
            : 0.3
        : 1}
        on:mouseover={() => (hovered = node)}
        on:focus={() => (hovered = node)}
        tabindex="0"
        on:click={(e) => {
        e.stopPropagation();
        }}
    />
{/each}
    </g>
  </svg>

  {#if hovered}
  <Tooltip data={hovered} {colorScale} {width}/>
  {/if}

</div>

<style>
:global(.tick text, .axis-title){
  font-size: 12px;
  font-weight: 400;
  user-select: none;
  fill: hsla(212, 10%, 53%, 1)
}

h1{
  font-size: 22px;
  margin: 0 0 6px 0;
  font-weight: 600;
  text-align: center;
}

circle {
    transition: stroke 300ms ease, opacity 300ms ease;
    cursor: pointer;
}

</style>