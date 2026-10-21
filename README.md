Sentinel Jam — GNSS / RFI Interference Monitor (Copernicus Sentinel-2)

<img width="1917" height="926" alt="image" src="https://github.com/user-attachments/assets/9b1dfa36-c8f2-4d16-a0c7-d890e2cd5ccd" />

Thanks especially to the SAT-JAM Group that allows us these analyzes with publicly available data!

https://sat-jam.com

Obviously, there will be several fixes to be made, and the code isn't clean, and you will find entities that appear to be false positives but are not always false positives. I consider the exercise very interesting for various levels of understanding, knowledge and technicalities.

As always, you can use the plain HTML locally 👋👋👋

To preview GitHub HTML files, use these services by pasting the links:

https://raw.githack.com/ (Tested / working)

Fast Link:

https://raw.githack.com/GiamMaBasedResearchers/Sentinel-Jam---GNSS-RFI-Interference-Monitor-Copernicus-Sentinel-2-/main/sentinel-jam-monitor.html

Thanks to:

https://github.com/neoascetic/rawgithack

Sentinel Jam — GNSS / RFI Interference Monitor (Copernicus Sentinel-2)

Sentinel Jam is a browser-based monitor that surveys GNSS radio-frequency interference using the GNSS receivers flown on the Copernicus Sentinel-2 satellites as worldwide, space-based detectors. It retrieves the daily GNSS RINEX observation files (AUX_GNSSRD) published as open data by the Copernicus Data Space Ecosystem, analyses them epoch by epoch directly in the browser, and displays correlated signal-quality degradations on an interactive world map along the satellites' ground tracks.

The whole application is a single HTML file: no installation, no build step, no server-side components. An optional, small Python helper provides a local download proxy and a one-click launcher.

Why monitoring from space

Terrestrial GNSS jammers are illegal, mobile and difficult to instrument from the ground, and interference hotspots often sit in regions with no monitoring infrastructure at all. A satellite in low Earth orbit that overflies an interfering region sees the event from above: while its line of sight crosses the affected area, several GNSS satellites tracked simultaneously show correlated drops of the received carrier-to-noise density ratio (C/N0). Sentinel-2 operates a sun-synchronous orbit with a 10-day ground-track repeat cycle and carries a GNSS receiver that works continuously, so the mission produces a daily, global and consistent record of the GNSS signal environment. Space-based observation therefore complements ground networks and aviation-based services, adding coverage over oceans, conflict areas and remote regions, with a single well-characterised sensor family.

The daily AUX_GNSSRD products record, every 10 seconds (8,640 epochs per day), which GNSS satellites are tracked and with which signal quality. When a terrestrial or airborne jammer is active, the passes that cross the affected region show simultaneous, correlated C/N0 degradations across multiple satellites, which is exactly the signature this tool extracts.

How the detection works

An epoch is flagged as degraded when any of the following criteria holds (all thresholds are user-adjustable):

- Absolute: at least one third of the tracked satellites are simultaneously below the L1/L2 C/N0 thresholds (defaults: 30 dB-Hz on L1, 20 dB-Hz on L2). Real interference saturates the whole front-end; a single low-elevation satellite is normal geometry.
- Relative baseline: at least half of the tracked satellites (minimum 4) drop at least 8 dB below their own daily median at the same time (sensitivity selectable: balanced 8 dB, sensitive 7 dB, very sensitive 6 dB).
- Cross-constellation: GPS and Galileo degrade together, the signature of a wideband source.

Flagged epochs are grouped into time segments and drawn on the map at the approximate sub-satellite position of the overflying pass. Events seen on a single constellation are reported on their own map layers (Detections GPS only / Detections Galileo only); events seen on both constellations at the same time are classified as wideband. In the default wideband mode the high-confidence cross-GNSS core is complemented by a second per-constellation pass, so that disturbances visible only to GPS or only to Galileo are never discarded.

Two further detectors complete the picture:

- Coverage gaps: missing epochs in the RINEX record are reported as data outages along the ground track. The minimum outage is selectable: 30 seconds (2 or more missing epochs, default), 1 minute (5 or more) or 2 minutes (11 or more). The nominal 10-second boundary between daily files never counts as a gap.
- Speed / kinematic check (anti-spoofing and receiver quality): the AUX_GNSSRD files contain no positions, velocities or navigation data, only pseudorange, carrier phase and receiver clock at 10-second intervals. The check verifies the physical consistency of exactly these observables with three metrics that are all independent of the receiver clock: a correlated carrier-phase versus code rate mismatch on three or more satellites at once (the signature of spoofed signals with non-physical dynamics), line-of-sight acceleration deviating from the cross-satellite median beyond physical bounds (nominal spread on real files: p99 of about 8 to 10 m/s2), and receiver-clock steps at the millisecond scale (real -10 ms events have been observed on Sentinel-2C). Levels: Normal (anti-spoofing, default), High sensitivity and Off. Flagged intervals are reported as speed / kinematic alerts on a dedicated map layer. Single-satellite multipath and ordinary cycle slips stay below the rules by design.

Platforms

<img width="730" height="256" alt="image" src="https://github.com/user-attachments/assets/583d2b30-e82b-4dad-bbee-cfe2ddc8f428" />


What the dashboard provides

- Interactive hexagonal map with two modes: Anomalies (event collection, one cell colour per worst observed event, chronological popups, minimum-events filter) and Heatmap (share of affected passes per cell, fixed eight-colour ramp, quantile filter). Hexagon size auto-adaptive or manual from about 75 km to about 600 km, adjustable opacity, four base maps (Dark, Streets, Satellite, Terrain).
- Map layers: detections per constellation, coverage gaps, speed / kinematic alerts, indicative RFI zones derived from public reports, reference ground tracks, live spacecraft positions.
- Timeline chart with one point per 10-second epoch (mean number of tracked satellites, mean C/N0, degraded flags), line toggles and an expandable explorer with a measurement crosshair.
- Per-file results list with detection counters (GPS, Galileo, wideband, gaps, speed alerts), clickable entries that fly to the corresponding map object, an analysis metadata line with the active detection settings, and run statistics (epochs analysed, minimum epoch C/N0, unreadable files).
- Multi-language interface; every setting persists in the browser; shareable URL parameters (?from=YYYY-MM-DD&to=YYYY-MM-DD) reproduce any analysis window exactly.

Data source

- Catalogue: Copernicus Data Space Ecosystem (CDSE), public OData API. The tool queries three auxiliary product families of Sentinel-2: AUX_GNSSRD (daily GNSS RINEX observations), AUX_POEORB (precise orbits) and AUX_PROQUA (attitude quaternions). Browsing the catalogue requires no login.
- Downloading the binary products requires a free CDSE account, created at https://dataspace.copernicus.eu.
- The daily files are published with the normal pipeline consolidation delay of about 5 to 6 days after the observed day; the dashboard shows a data availability notice so that the missing last days are not mistaken for an anomaly.
- Files are RINEX 3.03 observation records (pseudorange, carrier phase, C/N0) sampled at 10 seconds, about 8,640 epochs per day and platform.

Three ways to run the analysis

1. Standalone HTML: double-click sentinel-jam-monitor.html. The catalogue browser, the map and the spacecraft panels work immediately; downloading products directly from the browser is limited by cross-origin rules, so for full RINEX analysis use one of the two options below.
2. Python package (recommended): extract sentinel-jam-monitor-python.zip, start it with one double click; the local server serves the application and relays the RINEX downloads with the CDSE token issued at login. Requirements: Python 3 only.
3. Local files without login: analyse AUX_GNSSRD products already on disk (.TGZ products from CDSE, or bare .RNX / .RNX.gz), up to 120 files per run, with results bit-identical to an account run over the same files. The companion script download-datasets.py fetches ready-made six-day datasets (sample windows of October 2018, October 2022 and October 2026) into separate folders, each with a manifest of the products.
All three paths share the same parser and the same detection parameters: whatever you set applies to account runs and to local files, and changing a parameter re-analyses the results already on screen without downloading anything again.

Getting started (recommended path)

1. Extract the zip archive into a new folder.
2. On Windows, double-click start-rinex.bat; on macOS or Linux, run bash start-rinex.sh from a terminal in that folder. Python 3 is the only requirement.
3. The browser opens automatically on http://localhost:8123.
4. Click the CDSE chip in the top bar and enter your free Copernicus Data Space credentials once.
5. In the RINEX panel choose the spacecraft (all, S2A, S2B or S2C), the UTC date range (up to one year) and the maximum number of days per run (up to 60, about two files per day), then click ANALYZE RINEX.
6. Read the map, the results list and the timeline; refine the detection parameters if needed. Clicking any list entry flies to the corresponding event on the map.


Detection parameters

<img width="730" height="387" alt="image" src="https://github.com/user-attachments/assets/0c193c8f-ecda-48ae-a3c0-362eed8764f4" />

Reading results responsibly: natural causes matter

Not every correlated anomaly is intentional interference. Several natural and technical effects produce the same signature, and a responsible reading must always take them into account:

- Ionospheric scintillation produces correlated multi-satellite C/N0 fades, concentrated at high latitudes (above about 55 degrees) and over the equatorial oceans after sunset. High-latitude detections are tagged in the results.
- Multipath and low-elevation geometry usually affect single satellites and are filtered by the multi-satellite rules, but they remain visible in the per-satellite charts.
- Receiver and platform events, such as clock steps, cycle slips, receiver reboots or hardware ageing, appear in the record; the speed / kinematic check surfaces clock steps explicitly as alerts.
- Solar radio bursts can degrade C/N0 on all satellites of the sunlit hemisphere simultaneously.
- Severe ionospheric storms and more rare atmospheric effects can also mimic short interference episodes.

As a rule of thumb, detections inside the marked RFI zones or over known conflict areas are consistent with jamming, while high-latitude or open-ocean detections are usually natural. A single source is never proof: cross-check independent services, such as daily GNSS interference maps derived from ADS-B data (for example GPSJam), public RFI reports and official notices, before drawing operational conclusions.

Limitations

- The method is a screening technique based on C/N0 telemetry: it does not geolocate jammers and does not measure their emitted power. Events are placed at the approximate sub-satellite position of the overflying pass.
- The 10-second sampling can miss or smooth short events.
- The analysis is not real-time: products appear with a delay of about 5 to 6 days.
- No Sentinel-2A data since 21 January 2025 due to the failure of its GNSS receiver.
- Sensitivity differs between a GPS-only and a GPS-plus-Galileo receiver; use the relative-baseline criterion for cross-platform comparisons.
- Positions are drawn along nominal sun-synchronous ground tracks and are approximate by design.

Privacy and security

- All processing runs locally in the browser; nothing is uploaded to any server other than the Copernicus services themselves.
- The CDSE password is used once, directly against the official Copernicus identity service, to obtain an OAuth2 token; only the token is kept in the browser and it can be discarded at any moment with the logout chip.
- The optional local Python proxy relays requests to Copernicus using the token provided by the browser for each request; it stores no credentials and sends nothing anywhere else.
- No telemetry, no analytics, no third-party tracking. Map tiles are retrieved from public Esri and OpenTopoMap services.

Repository contents

<img width="727" height="261" alt="image" src="https://github.com/user-attachments/assets/fd731acb-bab7-4bb0-801c-38a00c946743" />

File added VALIDATION-REPORT.md

This report is our attempt to be as open as possible about what our tool does — and about what we can and cannot yet prove. It documents the data, the algorithms, the tests we ran and the limits we still have, with all evidence and references included. We do our best to give credibility to everything we publish: we believe honesty about open points is part of that effort, and we keep working on them with the same care and dedication.

File added GNSS-MONITORING-RESOURCES.md

A curated catalogue of the monitoring services and reference networks we rely on — space weather, ionosphere, satellite clocks, interference monitoring from LEO, and CORS / RTK / RINEX providers for GPS, GLONASS, BeiDou and Galileo. Every entry was checked against public documentation and carries its source, so that anyone can verify the same references we use. It is a small but sincere piece of our ongoing commitment to careful, dedicated work.

Credits

This code is provided by: "GiamMa-based researchers SDR R&D IoT" | @giammaiot2 — https://x.com/giammaiot2

Sentinel-2 auxiliary products are used under the Copernicus open data licences; the application contains Copernicus Sentinel-2 data processed for monitoring purposes.

Related projects by the same author

Other open-source projects of the "GiamMa-based researchers SDR R&D IoT":

1. GNSS-RO Multi-Portal — GNSS Radio Occultation & Scatterometer: real-time, multi-source dashboard monitoring radio-frequency interference, signal anomalies and data-quality degradation in GNSS Radio Occultation observations (bending angles and refractivity), aggregating live feeds from ECMWF, EUMETSAT ROM-SAF and space-situational-awareness platforms with an interactive 3D anomaly globe: https://github.com/GiamMaBasedResearchers/GNSS-RO-Multi-Portal--GNSS-Radio-Occultation-Scatterometer

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/9fb2e23d-94a3-46cc-b9d6-baaad8c279fb" />


2. SMAP-RFI-Analysis: A client-side web application designed to visualize and analyze Radio Frequency Interference (RFI) data from NASA's Soil Moisture Active Passive (SMAP) satellite mission. This tool enables researchers to detect temporal changes in interference patterns, identifying zones where RFI has appeared ("Turned ON") or disappeared ("Turned OFF") over time: https://github.com/GiamMaBasedResearchers/SMAP-RFI-Analysis

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a9948f7b-a4dc-4092-8321-26db78203470" />


3. ADS-B Sky Tracker Pro: browser-based aviation tracking application focused on GNSS/GPS jamming detection, signal integrity analysis, flight anomaly detection and Secondary Surveillance Radar (SSR) analysis, aggregating data from multiple ADS-B providers: https://github.com/GiamMaBasedResearchers/ADS-B-Sky-Tracker-Pro

<img width="1918" height="918" alt="image" src="https://github.com/user-attachments/assets/2a737595-c9ab-4b62-9375-0160e82da4a3" />


References

1. Copernicus Data Space Ecosystem, portal and free registration: https://dataspace.copernicus.eu

2. Copernicus Data Space Ecosystem, OData API documentation: https://documentation.dataspace.copernicus.eu/APIs/OData.html

3. Sentinel-2 mission, ESA Sentinel Online: https://sentinels.copernicus.eu/web/sentinel/missions/sentinel-2

4. Copernicus POD Product Handbook (AUX_GNSSRD, AUX_POEORB, AUX_PROQUA product definitions): https://sentinels.copernicus.eu/documents/d/sentinel/copernicus-pod-product-handbook

5. RINEX, The Receiver Independent Exchange Format, Version 3.03, IGS: https://files.igs.org/pub/formats/rinex/rinex303.pdf

6. GPSJam, daily GNSS interference maps derived from ADS-B data (independent cross-check): https://gpsjam.org

7. Stanford University, GNSS Interference Detection using ADS-B, monitoring and analysis dashboard: https://waas-nas.stanford.edu/

8. Stanford University, GNSS Interference Map, ADS-B/NIC-based RFI monitoring and research (GPS Research Lab): https://rfi.stanford.edu/

9. Flightradar24, GPS jamming map derived from ADS-B navigation integrity data: https://www.flightradar24.com/data/gps-jamming

10. Wingbits, live GPS jamming and spoofing map: https://wingbits.com/gps-jamming?layers=both

11. sat-jam.com, independent comparable GNSS interference monitoring service (cross-check only, not part of this project): https://sat-jam.com


Author profile and updates: https://x.com/giammaiot2


This code provide from: "GiamMa-based researchers SDR R&D IoT" | @GiammaIoT2 License

This project is provided for research and educational purposes.
