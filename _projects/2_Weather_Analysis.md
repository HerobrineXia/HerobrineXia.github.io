---
name: United States Weather Interactive Analysis
tools: [Python, Pandas, Altair, Vega-Lite]
image: 
description: This is an analysis of the Weather data in the United States using Altair and Vega-Lite for interactive visualizations.
custom_js:
  - vega.min
  - vega-lite.min
  - vega-embed.min
  - justcharts
---


# United States Weather Interactive Analysis
## Chart 1:
<vegachart schema-url="{{ site.baseurl }}/assets/json/state_temperature_interactive.json" style="width: 100%"></vegachart>

### Description

This view links the geographic distribution of BFRO reports to seasonal temperature patterns. A choropleth map colors each state by the number of sightings, and clicking any state reveals a set of monthly temperature curves showing how that location’s highs, lows, and mid-range values evolve across the year. This allows viewers to explore whether warmer states report more sightings, and how seasonal temperature fluctuations differ across the states.

### Design Choices

The map uses an Albers USA projection with a `Blues` sequential palette so denser report clusters read immediately, while the selected state receives an overlaid red outline for emphasis. The temperature panel reuses the same selection, plots three colored lines from the `category10` scheme, keeps month labels horizontal for readability, and adds a dynamic text header (“Average Monthly Temperatures in …”) so readers always know which state they are inspecting.

### Data Transformation

In the notebook I use `dropna` to drop the rows lacking dates or temperature fields, convert `date` to `datetime`, and extract `month`. I then aggregate per state to obtain `report_stats`, compute monthly mean high/low/mid temperatures, rename the series, and `melt` them into a long form (`temps_melt`). The map enriches the GeoJSON via `transform_lookup` with reports counts, and the temperature chart filters `temps_melt` with the shared `states_selected` parameter.

## Chart 2:
<vegachart schema-url="{{ site.baseurl }}/assets/json/uv_temp_correlation_interactive.json" style="width: 100%"></vegachart>

### Description

This pair of charts focuses on how UV index and temperature interact across seasons. The left scatter plot UV index against the reported high temperature, colored by season; brushing any region filters the right-hand bars, which instantly report the Pearson correlation between UV and temperature for each season within the brushed subset. This allows viewers to explore whether higher UV levels correspond to hotter days, and how that relationship varies seasonally.

### Design Choices

I use the `category10` palette for scatter points so seasons remain consistent with the chart. I use 0.7 opacity to prevent overplotting, and tooltips exposing season, UV, and temperature values. The correlation bars keep a fixed Spring→Winter order, reuse the seasonal colors, and format coefficients to two decimals. An interval selection attached to the scatter’s x/y channels lets viewers compare how strongly the two metrics move together by season.

### Data Transformation

I start from `data2 = data_raw.dropna(subset=['uv_index','temperature_high','season'])`. Within Altair, `transform_calculate` derives `xy`, `x2`, and `y2`, `transform_aggregate` sums them per season (yielding `sum_x`, `sum_y`, `sum_xy`, `sum_x2`, `sum_y2`, and `count`), and a second `transform_calculate` applies the Pearson formula `(n*sum_xy - sum_x*sum_y) / sqrt(...)`. A guard `transform_filter("datum.count > 1")` removes seasons with insufficient brushed samples. Because the same brush drives both panels, the correlation bars always reflect exactly the records highlighted in the scatter.

<div class="left">
{% include elements/button.html link="https://raw.githubusercontent.com/UIUC-iSchool-DataViz/is445_data/main/bfro_reports_fall2022.csv" text="The Data" %}
</div>

<div class="right">
{% include elements/button.html link="https://github.com/HerobrineXia/HerobrineXia.github.io/blob/main/python_notebooks/Workbook.ipynb" text="The Analysis" %}
</div>
