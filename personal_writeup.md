# Energy Market Dashboard: Visualizing How Electricity Prices Move in New York
 
**App:** https://advanced-computing-dancing-cactus-streamlit-app-birdi8.streamlit.app/market_analysis
**Github repository:** https://github.com/advanced-computing/dancing-cactus
**Team:** Eisuke, Chenyi

# This page explain the dashboard what we created in the class.

## What we built
 
Our team built an interactive Streamlit dashboard that explores how electricity prices in New York State respond to demand and fuel costs. The app pulls real-time Locational Marginal Prices (LBMP) and actual load data from NYISO, combines them with Henry Hub natural gas spot prices, and lets users explore the relationships across NYISO's eleven zones and across time.
 
The question we originally set out to answer was simple: *How does the LBMP in New York City fluctuate with changes in demand?* 
 
![Dashboard overview](screenshots/overview.png)
 
## The story we ended up telling
 
Early on we assumed the dashboard would mostly show a clean correlation between demand and price — true on average, but not very revealing. The dashboard evolved into a three-step narrative built around moments where intuition fails:
 
1. **Two side-by-side maps of NYISO zones**, one colored by load and one by LBMP, surface an asymmetry: the zones with the highest demand are not always the ones with the highest prices. Demand alone does not explain where electricity is expensive.
![Side-by-side load and price maps](images\side_by_side_map.png)
2. **A scatter plot of load vs. LBMP for a chosen zone**, faceted by month, zooms into one zone (typically NYC) and shows the relationship is non-linear. Prices stay low and stable across most of the load range but become sharply more volatile near system peaks.
![A scatter plot of load vs. LBMP](images\scatter_plot.png)
3. **A dual-axis time series of LBMP and Henry Hub gas prices** shows that fuel costs — the other obvious explanatory variable — also fail to account for the spikes, pointing toward structural factors like transmission congestion and the local generation mix.
![A dual-axis time series of LBMP and Henry Hub gas prices](images\comparison_chart.png)
The dashboard is not meant to give a single answer. It walks a policymaker from "demand drives price" to "price is shaped by *where* demand occurs within a constrained grid."
 
 
## How it's built
 
The system is a thin Streamlit front-end on top of an automated BigQuery pipeline:
 
- **Ingestion.** Two Python scripts per data source — one for the initial bulk backfill, one for daily incremental updates. Splitting them keeps the production path lean and the historical load reproducible.
- **Storage.** Three tables in BigQuery (LBMP, actual load, Henry Hub). Choosing BigQuery over local CSVs let us share data across the team and scale to roughly ten years of hourly data without rethinking the pipeline.
- **Automation.** A GitHub Actions cron job runs the update scripts daily, satisfying the project's "data is automatically updated" requirement.
- **Front-end.** Streamlit, with `@st.cache_data(ttl=3600)` on the BigQuery loaders and URL query parameters for shareable state. Maps use Plotly choropleth on a NYISO zone GeoJSON.
The repo is organized so anyone with a service account can clone it and run the dashboard locally.
 
## What I learned
 
Two lessons stuck with me, and neither is about a specific tool:
 
**Performance forces design choices.** Loading a year of hourly data on every interaction was unworkable, so we exposed a month selector instead of a year-range slider. That UI decision is really a backend constraint in disguise. Splitting the BigQuery loaders into separate cached functions cut typical interactions from ~20 seconds to under one. `@st.cache_data` is powerful, but it also hides bugs — a stale cache once made me think a path fix wasn't working when it actually was.
 
**Visualization choices are arguments.** Putting the two maps side by side is a different argument than overlaying them; faceting the scatter plot by month is a different argument than aggregating across a year. Every chart in the final dashboard exists because we decided what we wanted the viewer to *notice*, then worked backward to the encoding. The biggest improvements late in the project came not from adding features, but from cutting interpretive text and letting the visuals carry more of the argument.
 
The presentation went well, and this is now a project I'd be happy to walk through in an interview.