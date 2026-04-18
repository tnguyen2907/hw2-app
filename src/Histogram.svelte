<script>
  import * as d3 from 'd3'

  export let data = []
  export let fullData = []
  export let variable = ''
  export let filter = []
  export let update
  export let title = ''
  export let xLabel = ''
  export let yLabel = ''

  let margin = { top: 10, right: 30, bottom: 46, left: 52 }
  let width = 360
  let height = 220
  let chartW = width - margin.left - margin.right
  let chartH = height - margin.top - margin.bottom

  let brushLayer
  let xAxis
  let yAxis

  const brush = d3
    .brushX()
    .extent([
      [0, 0],
      [chartW, chartH],
    ])
    .on('brush', brushed)
    .on('end', brushended)

  function brushed(event) {
    if (event && event.selection) {
      filter = [xScale.invert(event.selection[0]), xScale.invert(event.selection[1])]
      if (update) update()
    }
  }

  function brushended(event) {
    if (event && !event.selection) {
      filter = []
      if (update) update()
    }
  }

  $: values = variable ? fullData.map((d) => d.properties[variable]).filter((d) => d != null) : []
  $: xScale = d3.scaleLinear().range([0, chartW]).domain(d3.extent(values))
  $: binData = d3
    .histogram()
    .value((d) => d.properties[variable])
    .domain(xScale.domain())
    .thresholds(xScale.ticks(30))
  $: backgroundBins = variable ? binData(fullData.filter((d) => d.properties[variable] != null)) : []
  $: bins = variable ? binData(data.filter((d) => d.properties[variable] != null)) : []
  $: yScale = d3
    .scaleLinear()
    .range([chartH, 0])
    .domain([0, d3.max(backgroundBins, (d) => d.length) || 0])

  $: {
    if (brushLayer && xAxis && yAxis && values.length > 0) {
      d3.select(brushLayer).call(brush)
      d3.select(xAxis).call(d3.axisBottom(xScale))
      d3.select(yAxis).call(d3.axisLeft(yScale))
    }
  }
</script>

<main>
  <h2>{title}</h2>
  <svg {width} {height}>
    {#if values.length > 0}
      <g transform="translate({margin.left}, {margin.top})">
        {#each backgroundBins as d}
          <rect
            class="backgroundbar"
            x={xScale(d.x0)}
            y={yScale(d.length)}
            width={xScale(d.x1) - xScale(d.x0)}
            height={chartH - yScale(d.length)}
          />
        {/each}
        {#each bins as d}
          <rect
            class="bar"
            x={xScale(d.x0)}
            y={yScale(d.length)}
            width={xScale(d.x1) - xScale(d.x0)}
            height={chartH - yScale(d.length)}
          />
        {/each}
      </g>
    {/if}

    <g transform="translate({margin.left}, {margin.top})" bind:this={brushLayer} />
    <g transform="translate({margin.left}, {chartH + margin.top})" bind:this={xAxis} />
    <g transform="translate({margin.left}, {margin.top})" bind:this={yAxis} />

    <text class="axis-label" x={margin.left + chartW / 2} y={height - 8} text-anchor="middle">
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

  .bar {
    fill: goldenrod;
    stroke: white;
    stroke-width: 1px;
  }

  .backgroundbar {
    fill: grey;
  }

  .axis-label {
    fill: currentColor;
    font-size: 12px;
  }

  :global(.tick text) {
    fill: currentColor;
  }

  :global(.tick line),
  :global(.domain) {
    stroke: currentColor;
  }
</style>
