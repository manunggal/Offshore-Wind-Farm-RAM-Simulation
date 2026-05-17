# Offshore Wind Farm RAM Simulation

> 🚧 **Status: In development** — architecture and methodology defined, implementation in progress.

A map-based interactive tool to estimate the **availability of an offshore wind farm** by integrating wind resource data, component reliability modeling, and metocean-constrained maintenance scheduling — all driven by Monte Carlo simulation.

This project extends the generic RAM engine in [`RAM-Monte-Carlo-Simulation`](https://github.com/manunggal/RAM-Monte-Carlo-Simulation) with offshore-specific factors that are absent from conventional availability calculators: wind variability, weather access windows, and vessel logistics.

---

## Motivation

Offshore wind availability analysis is typically performed using commercial tools (e.g. Maros, TARO, or ECN Operation & Maintenance Tool) that treat wind resource and metocean conditions as fixed inputs. In reality, these variables are stochastic — wind speed determines both energy production and the ability to dispatch maintenance vessels — and their interaction with component reliability has a significant effect on achievable availability.

This tool aims to make that interaction explicit and explorable, grounded in real location-specific data rather than generic assumptions.

The design approach follows the pattern established in [`pv_solar_app`](https://github.com/manunggal/pv_solar_app) — a map-based Django application where the user selects a geographic point to retrieve location-specific resource data, then runs a simulation based on that data.

---

## Planned functionality

### 1. Location selection
The user selects a point on an interactive map. The application retrieves:
- **Wind speed data** at 10 m height (from ERA5 or equivalent reanalysis dataset)
- **Significant wave height** (Hs) and **ocean current speed** from metocean datasets for the selected location

### 2. Wind farm configuration
The user defines:
- Number of wind turbines
- Rated power per turbine
- Component-level Reliability Block Diagram (RBD) for each turbine — specifying failure rates (λ), time-to-repair (TTR), and redundancy arrangement (series / hot redundancy / cold redundancy)

### 3. Wind resource and production profile
Wind speed data is used to:
- Estimate the annual energy production (AEP) profile based on a turbine power curve
- Identify periods where wind speed falls outside operational limits (cut-in / cut-out)

### 4. Metocean-constrained weather windows
Significant wave height and current speed thresholds are applied to determine **access windows** — periods during which maintenance vessels can safely operate. This directly affects:
- Corrective maintenance response time (how quickly a failed component can be repaired)
- Preventive maintenance scheduling feasibility

### 5. Monte Carlo simulation
The simulation integrates all of the above:
- Stochastic time-to-failure generation for each component (exponential distribution via inverse transform method, consistent with `RAM-Monte-Carlo-Simulation`)
- Weather window constraints applied to maintenance durations
- System availability estimated as mean uptime across *n* iterations

### 6. Outputs
- **Technical availability** — component and system level
- **Energy-based availability** — actual production vs. theoretical maximum
- **Expected number of maintenance events** per year
- **Downtime attribution** — proportion due to component failure vs. weather access restriction
- **Sensitivity analysis** — how availability changes with wave height threshold, vessel speed, or component reliability

---

## Architecture

Same Django-based map application pattern as [`pv_solar_app`](https://github.com/manunggal/pv_solar_app):

```
offshore-wind-ram/
├── core/               # Monte Carlo simulation engine (Python)
├── metocean/           # Metocean data retrieval and weather window logic
├── windfarm/           # Wind resource processing and AEP estimation
├── reliability/        # RBD configuration and component failure modeling
├── map/                # Interactive map interface (Leaflet.js)
├── templates/          # Django HTML templates
├── static/             # Frontend assets
└── manage.py
```

**Key data sources planned:**
- [ERA5 reanalysis](https://cds.climate.copernicus.eu/) — wind speed, wave height, current
- [IRENA Global Atlas](https://globalatlas.irena.org/) — offshore wind resource reference
- User-defined turbine power curve

**Tech stack:**
- Python / Django — backend and simulation engine
- Leaflet.js — interactive map interface
- Plotly — simulation output charts
- Pandas / NumPy — data processing
- CDS API (cdsapi) — ERA5 data retrieval

---

## Background

This project grows directly out of professional experience in offshore energy — specifically tidal energy operations at [Sabella](https://www.sabella.bzh/), where maintenance access constraints and component reliability interact in exactly the way this tool aims to model.

The RAM methodology is documented and implemented in [`RAM-Monte-Carlo-Simulation`](https://github.com/manunggal/RAM-Monte-Carlo-Simulation), which serves as the simulation engine foundation for this project.

---

## Relationship to other projects

| Repository | Role |
|---|---|
| [`RAM-Monte-Carlo-Simulation`](https://github.com/manunggal/RAM-Monte-Carlo-Simulation) | Core RAM engine — RBD, Monte Carlo, reliability/availability outputs |
| [`pv_solar_app`](https://github.com/manunggal/pv_solar_app) | Architectural reference — map-based Django app retrieving location-specific resource data |
| This repo | Offshore extension — integrates wind resource, metocean constraints, and vessel logistics |

---

## Author

**Manunggal Sukendro** — Reliability engineer with experience in offshore energy systems (tidal and wind).  
[github.com/manunggal](https://github.com/manunggal) · [linkedin.com/in/manunggal-sukendro-3077b91a](https://www.linkedin.com/in/manunggal-sukendro-3077b91a/)
