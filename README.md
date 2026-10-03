# NETRA EWS — North East Terrain Risk Assessment & Early Warning System

NETRA EWS is a browser-based disaster-risk monitoring and early-warning prototype designed for hilly and landslide-prone regions. The current prototype is configured for **Assam hill slopes and key transit corridors** and provides a geospatial operations dashboard for monitoring environmental risk, explaining risk factors, logging field observations, collecting citizen hazard reports, and managing an authority-response workflow.

> **Project type:** Front-end prototype / decision-support dashboard  
> **Current version shown in the prototype:** v3.2 — Assam Operations  
> **Status:** Prototype / demonstration system

## Key Features

- **Geospatial monitoring dashboard** using Leaflet.
- **Five Assam monitoring nodes** with latitude, longitude, slope angle, historical vulnerability, population-density and road-importance weights.
- **Live weather telemetry** fetched from the Open-Meteo forecast API.
- **Multi-factor risk scoring** using rainfall, soil-moisture, slope and historical vulnerability.
- **Explainable risk factors (XAI-style contribution view)** showing the relative contribution of the four scoring components.
- **Emergency Priority Score (EPS)** that combines risk, population exposure, road importance and an anomaly multiplier.
- **Risk classification:** Critical, High, Moderate and Stable.
- **Risk-priority queue** that automatically sorts monitored zones by EPS.
- **Interactive map layers:** OpenTopoMap, Esri terrain, Esri satellite imagery and CARTO dark map.
- **Environmental anomaly detection** based on high 24-hour rainfall and soil-moisture conditions.
- **Authority workflow:** New Alert → Acknowledged → Team Assigned → Road Advisory Issued → Resolved.
- **Official field reports** stored locally in the browser and plotted on the map.
- **Citizen hazard reporting** with GPS detection, hazard type, notes, photos/videos and map markers.
- **Multilingual interface:** English, Hindi and Assamese.
- **Browser persistence** through `localStorage`.
- **Live clock and data-source/status indicators.**

## How the Prototype Works

### 1. Monitoring zones

The prototype contains five predefined Assam monitoring nodes:

1. Sonapur Slope Sector (NH-27)
2. Haflong Hill Bypass Corridor
3. Guwahati Narakasur Hill Slopes
4. Jorhat Disoi Valley Slope Node
5. Silchar Kalain Slope Sector (NH-6)

Each zone contains geographic coordinates, slope angle, historical vulnerability weight, population-density weight, road-importance weight and infrastructure/settlement impacts.

### 2. Environmental telemetry

For each zone, the browser calls Open-Meteo using the zone's latitude and longitude. The implementation requests current precipitation/rain and hourly rain and soil-moisture values.

The prototype then derives:

- Current rainfall
- 24-hour rainfall
- Soil-moisture percentage

If the API request fails, the prototype uses fallback values so the dashboard can continue to demonstrate its interface.

### 3. Risk engine

The current scoring implementation is a **deterministic weighted risk model**, not a trained machine-learning model.

The four normalized components are:

| Factor | Weight |
|---|---:|
| 24-hour rainfall score | 35% |
| Soil moisture score | 30% |
| Slope score | 20% |
| Historical vulnerability score | 15% |

The composite risk score is:

`Risk Score = 0.35 × Rainfall + 0.30 × Soil Moisture + 0.20 × Slope + 0.15 × Historical Vulnerability`

Risk categories in the current implementation:

- **CRITICAL:** Risk Score ≥ 75
- **HIGH:** Risk Score ≥ 58
- **MODERATE:** Risk Score ≥ 38
- **STABLE:** Risk Score < 38

### 4. Emergency Priority Score

The prototype also calculates an Emergency Priority Score using:

- Composite risk
- Population-density weight
- Road-importance weight
- Environmental anomaly multiplier

The score is capped at 99 in the current implementation.

### 5. Explainability

The dashboard converts the weighted components into relative percentage contributions and displays them as four XAI-style bars:

- Short-Term Rainfall Spike
- Soil Moisture Saturation
- Slope & Elevation Gradient
- Historical Landslide Vulnerability

This makes the displayed risk score easier to interpret.

## Reporting System

### Official Field Log

Officials can submit:

- Monitoring sector
- Hazard type
- Field notes

Reports are stored in browser `localStorage` and displayed as markers on the Leaflet map.

### Public Hazard Incident Upload

Citizens can submit:

- GPS coordinates using browser geolocation
- Hazard type
- Additional details
- Multiple photos/videos

The uploaded media is previewed in the browser and stored locally as data URLs. The report is then represented by a map marker.

> This is a prototype implementation. Reports are **not uploaded to a remote government/server backend** in the current version.

## Authority Workflow

The right-side decision console provides a simple state-machine style workflow:

```text
NEW ALERT
   ↓
ACKNOWLEDGED
   ↓
TEAM ASSIGNED
   ↓
ROAD ADVISORY ISSUED
   ↓
RESOLVED
```

The current implementation changes the selected zone's workflow state in browser memory.

## Technology Stack

- **HTML5**
- **CSS3**
- **Vanilla JavaScript**
- **Leaflet.js 1.9.4** — interactive maps
- **Open-Meteo API** — live weather/forecast telemetry
- **Browser Geolocation API** — citizen GPS capture
- **Browser FileReader API** — image/video preview and local media handling
- **localStorage** — local prototype persistence
- **OpenStreetMap / OpenTopoMap**
- **Esri map tiles**
- **CARTO map tiles**
- **Font Awesome 6.4.0**
- **Google Fonts — Inter & JetBrains Mono**

## Project Structure

```text
NETRA-EWS/
│
├── index.html          # Complete working prototype
├── README.md           # Project documentation
└── .gitignore          # Recommended Git ignore rules
```

The current prototype is intentionally contained in one HTML file so it can be opened and demonstrated easily.

## Running Locally

No build system or package installation is required for the current prototype.

### Option 1 — Open directly

Open `index.html` in a modern browser.

### Option 2 — Use a local server

For better browser compatibility, especially for geolocation:

```bash
python -m http.server 8000
```

Then open:

```text
http://localhost:8000
```

A local server is recommended because browser geolocation and some external-resource behaviour can be restricted when an HTML file is opened directly from the filesystem.

## Internet Requirement

The dashboard uses external resources, including:

- Open-Meteo
- Leaflet CDN
- Font Awesome CDN
- Google Fonts
- OpenTopoMap
- Esri map services
- CARTO map tiles

Therefore, the full prototype experience requires an internet connection.

## Important Implementation Notes

### This is not yet a production disaster-warning system

NETRA EWS is currently a **prototype / decision-support demonstration**. It should not be treated as an operational emergency-warning service.

The displayed risk values are produced by the JavaScript scoring formula in the browser. They are not the output of a validated or trained predictive model.

### Data-source transparency

The footer currently labels rainfall as Open-Meteo and also displays SoilGrids/Copernicus DEM as conceptual data sources. In the supplied HTML, the actual environmental API request is made to **Open-Meteo**. SoilGrids and Copernicus DEM are not directly fetched by the current JavaScript implementation.

### Local-only reports

Field reports and citizen reports are stored using browser `localStorage`. Clearing site data/browser storage will remove those locally stored records.

### GPS fallback

If browser geolocation is unavailable or permission is denied, the citizen-report form falls back to the currently selected monitoring zone coordinates.

### Media storage

Citizen-uploaded images/videos are converted into data URLs and stored locally in browser storage as part of the report object. This is suitable for a prototype but should be replaced with object storage/backend upload for production.

## Suggested Future Development

- Add a real backend API and database.
- Replace localStorage with authenticated server-side incident management.
- Integrate verified satellite/DEM/soil datasets directly.
- Add a proper trained ML model using historical landslide/flash-flood datasets.
- Add model validation, calibration and uncertainty estimates.
- Add real-time sensor/IoT integration.
- Add automated SMS/email/push alerts.
- Add role-based access for authorities, field teams and citizens.
- Add server-side media storage and moderation.
- Add audit logs for authority workflow actions.
- Add stronger mobile/responsive support.
- Add automated route-risk analysis and verified road-network data.
- Add monitoring, authentication, security and privacy controls before any operational deployment.

## Disclaimer

NETRA EWS is an academic/prototype system intended for demonstration and decision-support research. It does not replace official disaster-management authorities, professional geotechnical assessment, emergency services, or validated warning systems.

## License

No license is currently declared for this prototype. If this repository will be shared publicly, add a license that reflects the ownership and reuse terms agreed by the project team.
