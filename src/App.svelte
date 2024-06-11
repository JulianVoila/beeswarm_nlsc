<script>
  import data from "$data/data.js";
  console.log(data);

  import { forceSimulation, forceY, forceX, forceCollide } from "d3-force";
  import { scaleLinear, scaleBand, scaleOrdinal, scaleSqrt } from "d3-scale";
  import { extent } from "d3-array";
  import { each } from "svelte/internal";
  import { mean, rollups } from "d3-array";


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
        .y((d) => yScale(d.continent))
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

</script>


<div class="chart-container" bind:clientWidth={width}>
  <svg {width} {height}>
    <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
      <AxisX xScale={xScale} height={innerHeight} width={innerWidth}/>
      {#each nodes as node}
        <circle cx={node.x} cy={node.y} r={radiusScale(node.happiness)} 
        fill={colorScale(node.continent)} stroke="black" />
      {/each}
    </g>
  </svg>
</div>
