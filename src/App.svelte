<script>
  import data from "$data/data.js";
  console.log(data);

  import { forceSimulation, forceY, forceX, forceCollide } from "d3-force";
  import { scaleLinear, scaleBand } from "d3-scale";
  import { extent } from "d3-array";
  import { each } from "svelte/internal";
  import { mean, rollups } from "d3-array";

  const RADIUS = 5;

  $: simulation = forceSimulation(data)
    .force(
      "x",
      forceX()
        .x((d) => xScale(d.happiness))
        .strength(0.08),
    )
    .force(
      "y",
      forceY()
        .y((d) => yScale(d.continent))
        .strength(0.02),
    )
    .force("collide", forceCollide().radius(RADIUS));

  $: nodes = simulation.nodes();

  // console.log(simulation);

  let width = 400,
    height = 400;

  const margin = {
    top: 0,
    right: 0,
    bottom: 20,
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
    .domain([0, 10])
    .range([0, innerWidth]);

  let yScale = scaleBand()
    .domain(continents)
    .range([innerHeight, 0])
    .paddingOuter(0.5);
</script>

<div class="chart-container" bind:clientWidth={width}>
  <svg {width} {height}>
    <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
      {#each nodes as node}
        <circle cx={node.x} cy={node.y} r={RADIUS} fill="steelblue" />
      {/each}
    </g>
  </svg>
</div>
