<script>
  import * as d3 from 'd3'

  export let data = []
  export let fullData = []
  export let xVar = ''
  export let yVar = ''
  export let filterX = []
  export let filterY = []
  export let update
  export let title = ''
  export let xLabel = ''
  export let yLabel = ''

  let margin = { top: 10, right: 20, bottom: 46, left: 52 }
  let width = 360
  let height = 220
  let chartW = width - margin.left - margin.right
  let chartH = height - margin.top - margin.bottom

  let brushLayer
  let xAxis
  let yAxis

  const brush = d3
    .brush()
    .extent([
      [0, 0],
      [chartW, chartH],
    ])
    .on('brush', brushed)
    .on('end', brushended)

  function brushed(event) {
    if (event && event.selection) {
      filterX = [xScale.invert(event.selection[0][0]), xScale.invert(event.selection[1][0])]
      filterY = [yScale.invert(event.selection[1][1]), yScale.invert(event.selection[0][1])]
      if (update) update()
    }
  }

  function brushended(event) {
    if (event && !event.selection) {
      filterX = []
      filterY = []
      if (update) update()
    }
  }

  $: backgroundPoints =
    xVar && yVar
      ? fullData.filter((d) => d.properties[xVar] != null && d.properties[yVar] != null)
      : []

  $: points =
    xVar && yVar ? data.filter((d) => d.properties[xVar] != null && d.properties[yVar] != null) : []

  $: xScale = d3
    .scaleLinear()
    .range([0, chartW])
    .domain(d3.extent(backgroundPoints.map((d) => d.properties[xVar])))
    .nice()

  $: yScale = d3
    .scaleLinear()
    .range([chartH, 0])
    .domain(d3.extent(backgroundPoints.map((d) => d.properties[yVar])))
    .nice()

  $: {
    if (brushLayer && xAxis && yAxis && backgroundPoints.length > 0) {
      d3.select(brushLayer).call(brush)
      d3.select(xAxis).call(d3.axisBottom(xScale))
      d3.select(yAxis).call(d3.axisLeft(yScale))
    }
  }
</script>

<main>
  <h2>{title}</h2>
  <svg {width} {height}>
    {#if backgroundPoints.length > 0}
      <g transform="translate({margin.left}, {margin.top})">
        {#each backgroundPoints as d}
          <circle
            class="background-point"
            cx={xScale(d.properties[xVar])}
            cy={yScale(d.properties[yVar])}
            r="3"
          />
        {/each}

        {#each points as d}
          <circle
            class="point"
            cx={xScale(d.properties[xVar])}
            cy={yScale(d.properties[yVar])}
            r="3"
          />
        {/each}
      </g>
    {/if}

    <g transform="translate({margin.left}, {margin.top})" bind:this={brushLayer} />
    <g transform="translate({margin.left}, {chartH + margin.top})" bind:this={xAxis} />
    <g transform="translate({margin.left}, {margin.top})" bind:this={yAxis} />

    <text class="axis-label" x={width / 2} y={height - 6} text-anchor="middle">
      {xLabel}
    </text>

    <text
      class="axis-label"
      transform="translate(14, {margin.top + chartH / 2}) rotate(-90)"
      text-anchor="middle"
    >
      {yLabel}
    </text>
  </svg>
</main>

<style>
  h2 {
    margin: 0 0 8px 0;
    font-size: 1.1em;
  }

  .axis-label {
    fill: currentColor;
    font-size: 12px;
  }

  .background-point {
    fill: grey;
    opacity: 0.45;
  }

  .point {
    fill: goldenrod;
    opacity: 0.85;
  }

  :global(.tick text) {
    fill: currentColor;
  }

  :global(.tick line),
  :global(.domain) {
    stroke: currentColor;
  }
</style>
