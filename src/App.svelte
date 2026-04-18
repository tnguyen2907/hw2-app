<script>
  import * as d3 from 'd3'
  import { onMount } from 'svelte'
  import CountyMap from './Map.svelte'
  import Histogram from './Histogram.svelte'
  import Scatterplot from './Scatterplot.svelte'

  let data = []
  let fullData = []
  let filter1 = []
  let filter2 = []
  let filter3 = []
  let filter4 = []
  let filter5 = []
  let var1 = 'sleepAdjPrev'
  let var2 = 'depressionAdjPrev'
  let var3 = 'smokingAdjPrev'
  let var4 = 'copdAdjPrev'

  function parseNumber(value) {
    if (value == null) return null
    const trimmed = String(value).trim()
    if (!trimmed) return null
    const numeric = Number(trimmed.replace(/,/g, ''))
    return Number.isFinite(numeric) ? numeric : null
  }

  onMount(async function () {
    const table = d3.csv('places.csv')
    const geocoord = d3.json('us_counties.json').then((d) => d.features)

    await Promise.all([table, geocoord]).then((values) => {
      const table = values[0]
      const geocoord = values[1]
      const rowByFips = new globalThis.Map(
        table
          .filter((d) => d.CountyFIPS)
          .map((d) => [
            d.CountyFIPS,
            {
              ...d,
              TotalPopulation: parseNumber(d.TotalPopulation),
              TotalPop18plus: parseNumber(d.TotalPop18plus),
              SLEEP_AdjPrev: parseNumber(d.SLEEP_AdjPrev),
              DEPRESSION_AdjPrev: parseNumber(d.DEPRESSION_AdjPrev),
              CSMOKING_AdjPrev: parseNumber(d.CSMOKING_AdjPrev),
              COPD_AdjPrev: parseNumber(d.COPD_AdjPrev),
            },
          ]),
      )

      for (let i = 0; i < geocoord.length; i++) {
        const county = geocoord[i]
        const row = rowByFips.get(county.id)

        if (row) {
          data.push({
            ...county,
            properties: {
              ...county.properties,
              countyFips: county.id,
              stateAbbr: row.StateAbbr,
              stateName: row.StateDesc,
              countyName: row.CountyName,
              totalPopulation: row.TotalPopulation,
              totalPop18plus: row.TotalPop18plus,
              sleepAdjPrev: row.SLEEP_AdjPrev,
              depressionAdjPrev: row.DEPRESSION_AdjPrev,
              smokingAdjPrev: row.CSMOKING_AdjPrev,
              copdAdjPrev: row.COPD_AdjPrev,
            },
          })
        }
      }

      fullData = [...data]
      data = [...fullData]
    })
  })

  function updateData() {
    data = fullData.filter((d) => {
      const sleepValid =
        filter1.length === 0 ||
        (d.properties[var1] >= filter1[0] && d.properties[var1] < filter1[1])

      const depressionValid =
        filter2.length === 0 ||
        (d.properties[var2] >= filter2[0] && d.properties[var2] < filter2[1])

      const scatterValid =
        (filter3.length === 0 ||
          (d.properties[var3] >= filter3[0] && d.properties[var3] < filter3[1])) &&
        (filter4.length === 0 ||
          (d.properties[var4] >= filter4[0] && d.properties[var4] < filter4[1]))

      const mapValid = filter5.length === 0 || filter5.includes(d.properties.countyFips)

      return sleepValid && depressionValid && scatterValid && mapValid
    })
  }
</script>

<main>
  <h1>PLACES County Health Dashboard</h1>

  <div class="flex-container col">
    <div class="map">
      <CountyMap
        data={data}
        fullData={fullData}
        bind:filter={filter5}
        update={updateData}
      ></CountyMap>
    </div>

    <div class="flex-container row">
      <div class="viz">
        <Histogram
          data={data}
          fullData={fullData}
          variable={var1}
          bind:filter={filter1}
          update={updateData}
          title="Short Sleep Duration"
          xLabel="Age-adjusted prevalence (%)"
          yLabel="# of Counties"
        ></Histogram>
      </div>

      <div class="viz">
        <Histogram
          data={data}
          fullData={fullData}
          variable={var2}
          bind:filter={filter2}
          update={updateData}
          title="Depression"
          xLabel="Age-adjusted prevalence (%)"
          yLabel="# of Counties"
        ></Histogram>
      </div>

      <div class="viz">
        <Scatterplot
          data={data}
          fullData={fullData}
          xVar={var3}
          yVar={var4}
          bind:filterX={filter3}
          bind:filterY={filter4}
          update={updateData}
          title="Smoking vs. COPD"
          xLabel="Smoking prevalence (%)"
          yLabel="COPD prevalence (%)"
        ></Scatterplot>
      </div>
    </div>
  </div>
</main>

<style>
  .flex-container {
    display: flex;
    justify-content: center;
    gap: 10px;
  }

  .row {
    flex-direction: row;
  }

  .col {
    flex-direction: column;
  }

  .map {
    flex-grow: 1;
  }

  .viz {
    flex: 1 1 0;
  }
</style>
