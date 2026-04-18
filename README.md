# PLACES County Health Dashboard

## Live Demo

[https://luminous-vacherin-33ee15.netlify.app/](https://luminous-vacherin-33ee15.netlify.app/)

## Submission Note

When I uploaded the zip file to Gradescope, Gradescope extracted the archive and then froze. This is why I need to upload the codebase to GitHub: https://github.com/tnguyen2907/hw2-app
## Project Overview

The goal of the dashboard is to let a user explore county-level health patterns in the United States through a choropleth map and three additional linked visualizations.

The dashboard uses the CDC **PLACES: County Data (GIS Friendly Format), 2025 release** dataset together with a US counties GeoJSON file. The CSV provides county-level health measures, and the GeoJSON provides the county boundary polygons needed for the choropleth map.

## Dataset And Chosen Columns

The PLACES dataset contains many county-level health variables. For this dashboard, I selected the following columns:

- `TotalPop18plus`
  Adult population age 18 and older. I use this for the choropleth map.
- `SLEEP_AdjPrev`
  Age-adjusted prevalence of short sleep duration among adults (as a percentage).
- `DEPRESSION_AdjPrev`
  Age-adjusted prevalence of diagnosed depression among adults (as a percentage).
- `CSMOKING_AdjPrev`
  Age-adjusted prevalence of current smoking among adults (as a percentage).
- `COPD_AdjPrev`
  Age-adjusted prevalence of chronic obstructive pulmonary disease among adults (as a percentage).

The county geometries are joined to the PLACES table by county FIPS code.

## Visualization Choices

The dashboard contains four views:

1. **Choropleth map**
   The map shows US counties and colors them by `TotalPop18plus` using a sequential **log color scale**. I chose a log scale because adult population varies a lot across counties, and a linear scale made the smaller counties hard to distinguish.

2. **Histogram of short sleep**
   This histogram shows the distribution of `SLEEP_AdjPrev` across counties.

3. **Histogram of depression**
   This histogram shows the distribution of `DEPRESSION_AdjPrev` across counties.

4. **Scatter plot of smoking vs. COPD**
   This plot uses `CSMOKING_AdjPrev` on the x-axis and `COPD_AdjPrev` on the y-axis to show the relationship between the two measures.

## Interaction Design

All four views are linked through crossfiltering.

- The **map** uses a rectangular bounding-box brush. Counties are selected if their projected centroid falls inside the brushed rectangle.
- Each **histogram** uses a horizontal brush to select a numeric range (similar to the demo).
- The **scatter plot** uses a rectangular brush to select ranges on both x and y.
