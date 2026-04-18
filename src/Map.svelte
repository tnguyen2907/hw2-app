<script>
  import * as d3 from 'd3'
  import { legendColor } from 'd3-svg-legend'

  export let data = []
  export let fullData = []
  export let filter = []
  export let update

  let width = 1120
  let height = 500
  let legendWidth = 160
  let mapWidth = width - legendWidth

  $: counties = {
    type: 'FeatureCollection',
    features: fullData,
  }

  $: projection =
    fullData.length > 0 ? d3.geoAlbersUsa().fitSize([mapWidth, height], counties) : null

  $: path = projection ? d3.geoPath().projection(projection) : null

  $: popExtent =
    fullData.length > 0
      ? d3.extent(fullData.map((d) => d.properties.totalPop18plus).filter((d) => d > 0))
      : null

  $: scale =
    popExtent
      ? d3.scaleSequentialLog(d3.interpolateBlues).domain(popExtent)
      : null

  let legend

  $: {
    if (legend && scale) {
      const colorLegend = legendColor()
        .shape('rect')
        .cells(6)
        .labelFormat(d3.format('.2s'))
        .scale(scale)

      d3.select(legend).call(colorLegend)
    }
  }

  let brushLayer

  const brush = d3
    .brush()
    .extent([
      [0, 0],
      [mapWidth, height],
    ])
    .on('brush', brushed)
    .on('end', brushended)

  function brushed(event) {
    if (event && event.selection && path) {
      const [[x0, y0], [x1, y1]] = event.selection

      filter = fullData
        .filter((d) => {
          const [cx, cy] = path.centroid(d)
          return Number.isFinite(cx) && Number.isFinite(cy) && cx >= x0 && cx <= x1 && cy >= y0 && cy <= y1
        })
        .map((d) => d.properties.countyFips)

      if (update) update()
    }
  }

  function brushended(event) {
    if (event && !event.selection) {
      filter = []
      if (update) update()
    }
  }

  $: {
    if (brushLayer && path) {
      d3.select(brushLayer).call(brush)
    }
  }
</script>

<main>
  <h2>Adult Population (log-scale)</h2>
  <svg {width} {height}>
    {#if path && scale}
      {#each data as d}
        <path style="fill: {scale(d.properties.totalPop18plus)};" d={path(d)} />
      {/each}

      {#each fullData as d}
        <path
          class="county-hitarea geooverlay"
          d={path(d)}
        />
      {/each}

      <g bind:this={brushLayer} />
      <g transform="translate({mapWidth + 24}, 40)" bind:this={legend} />
    {/if}
  </svg>
</main>

<style>
  h2 {
    margin: 0 0 8px 0;
    font-size: 1.2em;
  }

  .geooverlay {
    stroke: grey;
    stroke-width: 0.3px;
    fill: transparent;
  }

  .county-hitarea:hover {
    stroke: goldenrod;
    stroke-width: 0.9px;
  }

  :global(.overlay) {
    cursor: crosshair;
  }

  :global(.selection) {
    fill: rgba(218, 165, 32, 0.18);
    stroke: goldenrod;
  }

  :global(.legendCells text),
  :global(.label) {
    fill: white;
  }
</style>
