# GNSS Monitoring Services and Reference Networks

**Scope:** GPS · GLONASS · BeiDou · Galileo — space weather, ionosphere, satellite clocks, interference monitoring from orbit, and ground-based RINEX / RTK / CORS infrastructure

**Document type:** annotated reference catalogue (companion to the Sentinel Jam Scientific and Technical Validation Report)

**Author:** GiamMa-based researchers SDR R&amp;D IoT | @giammaiot2 — https://x.com/giammaiot2

**Document version:** 1.0 — 21 October 2026

**Availability:** all services and networks listed in this catalogue are publicly documented and accessible through the cited URLs; access conditions (free, free-with-registration, or commercial subscription) are indicated for each entry.

---

## Executive overview

This document is a curated catalogue of the monitoring services, dashboards and reference-station networks that surround and support the Sentinel Jam project, a browser-based monitor that surveys GNSS radio-frequency interference (RFI) using the GNSS receivers of the Copernicus Sentinel-2 satellites as worldwide, space-based detectors. The catalogue contains **thirteen entries**: nine international monitoring resources operated by governmental agencies, space agencies and the open-source community (Entries 1–9), and four ground-based geodetic reference networks operating over the Russian Federation and Eurasia that provide RTK corrections, CORS data and RINEX archives (Entries 10–13).

The thirteen entries are not a random collection: each of them plays a specific, nameable role in the workflow of interference screening and validation. Space-weather and ionospheric services (Entries 1, 2, 3 and 7) provide the *exclusion criteria* that must be checked before attributing a correlated C/N0 degradation to a terrestrial jammer, because geomagnetic storms, ionospheric scintillation and solar radio bursts produce natural signatures that mimic interference. Constellation-level and instrument-monitoring resources (Entries 4, 5, 8 and 9) provide *health cross-checks*: if a satellite is officially unhealthy, or if a receiver clock event is documented, an anomaly seen in the telemetry has a benign explanation. The ESA NAVISP project (Entry 6) provides the *independent methodological anchor*: it demonstrates, with a full-scale industrial programme on the Spire LEO constellation, the very same detection principle that Sentinel Jam exploits on Sentinel-2. Finally, the ground-based networks (Entries 10–13) provide *reference data and correction streams* — RTK, DGNSS/VRS, PPP and RINEX — that enable ground-side comparison over the same territories observed from orbit.

Note on numbering: Entries 3 and 7 refer to the same service, the Ionosphere Monitoring and Prediction Center (IMPC) of the German Aerospace Center (DLR). The near-real-time European TEC map and its public image archive are listed as separate entries because they are used at different stages of the validation workflow — the near-real-time map for operational screening, the archive for retrospective analysis of past anomalies.

---

## 1. Purpose, scope and how to use this catalogue

### 1.1 Purpose

The purpose of this catalogue is to give the user of Sentinel Jam — and, more generally, anyone working on GNSS interference monitoring, ionospheric screening, RINEX analysis, RTK positioning or CORS network operations — a single, structured entry point to the official resources that a rigorous analysis must consult. Every entry is described with its operator, its primary URL, the products it publishes, and an explicit statement of why it matters for interference work. The catalogue deliberately covers resources from the four global navigation satellite systems (GPS, GLONASS, BeiDou, Galileo), because a multi-constellation anomaly screen needs a multi-constellation set of ground references, and because different operators publish different — and sometimes uniquely valuable — views of their own system.

### 1.2 Conventions

Each entry carries a sequential number (1–13), a primary URL, and a compact description. Access is classified as *open* (no registration), *open with registration* (free account), or *subscription* (commercial). Quantitative statements about physical effects (Entry 8) are supported by peer-reviewed literature cited in the References section. The symbols and acronyms follow the terminology already established in the companion validation report: C/N0 for carrier-to-noise-density ratio, SV for space vehicle (satellite), TEC for total electron content, VTEC for vertical TEC, NRT for near real time, and DOP for dilution of precision.

### 1.3 Quick index of the thirteen entries

| # | Entry | Operator | Primary URL | Access |
| --- | --- | --- | --- | --- |
| 1 | Global Positioning System (GPS) Community Dashboard | NOAA / SWPC (USA) | https://swpc.noaa.gov/communities/global-positioning-system-gps-community-dashboard | open |
| 2 | ESA Space Weather Service Network — Current Space Weather | ESA (Europe) | https://swe.ssa.esa.int/current-space-weather | open |
| 3 | Ionosphere Monitoring and Prediction Center — NRT TEC map, Europe | DLR (Germany) | https://impc.dlr.de/products/total-electron-content/near-real-time-tec/near-real-time-tec-map-europe#panel-44-1 | open |
| 4 | BeiDou Navigation Satellite System — Monitoring and Evaluation | CSNO (China) | http://en.beidou.gov.cn/SYSTEMS/Monitoringandevaluation/202401/t20240103_27615.html | open |
| 5 | GLONASS Application Consumer Center — Information and Analytical Center (IAC) | Roscosmos / TsNIIMash (Russia) | https://glonass-iac.ru | open |
| 6 | GNSS Interference Monitoring from LEO (NAVISP EL2-117) | ESA NAVISP | https://navisp.esa.int/uploads/files/documents/NAVISP%20EL2-117%20Presentation%20Final.pdf | open (PDF) |
| 7 | Ionosphere Monitoring and Prediction Center — TEC archive | DLR (Germany) | https://impc.dlr.de/SWE/Total_Electron_Content/TEC_Near_Real-Time/DLR_GNSS_GCG_L4_VTEC-NTCM-SCM_NC_EUROPE/v2.0.0/ | open |
| 8 | GNSS Monitoring Portal — Satellite Clocks (gravitational time dilation) | DLR (Germany) | https://gnss-monitoring.dlr.de/satellite-clock | open |
| 9 | galmon.eu — Coverage and DOP maps | community project (B. Hubert) | http://galmon.eu/geo/coverage.html | open, open source |
| 10 | SmartNet Russia | Leica Geosystems / Hexagon | https://smartnetrtk.ru | subscription |
| 11 | Topnet Live Russia | Topcon (GSI, Russia) | https://topnet-gsi.ru | subscription |
| 12 | HIVE — RTK and RINEX | Industrial Geodetic Systems R&D | https://hive.geosystems.aero | subscription |
| 13 | PiDATA — GNSS base-station network | PiDATA community network | https://pidt.net | community / subscription |

---

## 2. Part A — International monitoring dashboards and services (Entries 1–9)

### Entry 1 — Global Positioning System (GPS) Community Dashboard (NOAA SWPC)

**Primary URL:** https://swpc.noaa.gov/communities/global-positioning-system-gps-community-dashboard

The GPS Community Dashboard is the single page with which the NOAA Space Weather Prediction Center (Boulder, Colorado) condenses, for the satellite-navigation user community, the space-weather conditions that currently matter for GNSS. The page presents current conditions on the standard NOAA scales — radio blackouts (R1–R5), solar radiation storms (S1–S5) and geomagnetic storms (G1–G5) [1][2] — together with solar activity, total electron content products and model-based nowcasts of the ionosphere. Each NOAA scale level is mapped directly onto the expected impact class on GPS-based systems and services, which makes the dashboard usable even by readers who are not space-weather specialists.

For interference monitoring the dashboard is the first exclusion authority of the workflow. A strong geomagnetic storm, an equatorial plasma-bubble evening or an L-band solar radio burst can depress C/N0 on many satellites simultaneously, over areas reaching thousands of kilometres, producing signatures that a naive screen would classify as jamming [1]. The operating rule adopted in this project is therefore to consult the NOAA dashboard (and the equivalent European services of Entries 2 and 3) *before* attributing any correlated multi-satellite C/N0 fade to a terrestrial interference source — with particular attention at high latitudes, where auroral-oval activity concentrates.

### Entry 2 — ESA Space Weather Service Network — Current Space Weather

**Primary URL:** https://swe.ssa.esa.int/current-space-weather

The Current Space Weather page of the ESA Space Weather Service Network (SWE) provides, in the portal's own words, "a snapshot of the current space weather conditions based on the latest products from the SWE Network" [3][4]. The network is a federation of European expert service centres coordinated by ESA under the Space Safety Programme, covering the four classical domains of space weather: solar and upper-atmosphere weather, heliospheric weather, ionospheric weather, and geomagnetic conditions. The snapshot page summarises solar wind parameters, flare and proton-event status, geomagnetic indices and the ionospheric products of the federated centres, with pointers to the underlying product archives.

The value of Entry 2 for interference work is twofold. First, it is the European counterpart of the NOAA dashboard (Entry 1), so a European anomaly can be screened against both American and European space-weather assessments, which reduces the chance of missing a regional ionospheric event. Second, the federated structure of SWENet gives access to specialised European ionospheric and scintillation products — including the historical SWENET archives — that are not available through any single national provider. Cross-checking two independent service networks (NOAA and ESA) before flagging interference is an inexpensive and effective way to enforce the "natural cause first" discipline that the validation report identifies as essential for any C/N0-based screen.

### Entry 3 — Ionosphere Monitoring and Prediction Center — Near-Real-Time TEC Map, Europe (DLR)

**Primary URL:** https://impc.dlr.de/products/total-electron-content/near-real-time-tec/near-real-time-tec-map-europe#panel-44-1

The Ionosphere Monitoring and Prediction Center (IMPC), maintained by DLR at Neustrelitz (Germany), is a near-real-time information and data service on the current state of the ionosphere [5]. Its flagship product relevant here is the near-real-time vertical TEC map of Europe, generated from the European GNSS ground-station network with the NTCM-SCM model (product identifier `DLR_GNSS_GCG_L4_VTEC-NTCM-SCM_NC_EUROPE`, version 2.0.0) [6][8]. The IMPC publishes global TEC maps on a 2.5° × 5.0° grid, one-hour forecast TEC maps for Europe with a latency of a few minutes, and warnings for ionospheric storms and travelling ionospheric disturbances (TIDs).

This map is the natural screening instrument for anomalies observed over the European sector: a strong TEC gradient, a storm-enhanced density structure or a TID passage changes the propagation conditions of GNSS signals and can produce correlated C/N0 fluctuations on the ground and, to a lesser extent, on overflying LEO receivers. The operational procedure adopted in this project is to compare the epoch of every European anomaly candidate with the IMPC map for the same interval; anomalies that coincide with documented storm activity are reclassified as ionospheric rather than man-made. The companion archive of the same product is catalogued separately as Entry 7.

### Entry 4 — BeiDou Navigation Satellite System — Monitoring and Evaluation (CSNO)

**Primary URL:** http://en.beidou.gov.cn/SYSTEMS/Monitoringandevaluation/202401/t20240103_27615.html

The linked page is an official monitoring-and-evaluation document of the China Satellite Navigation Office (CSNO), the government authority for the BeiDou Navigation Satellite System (BDS). The specific document (numbered 2024-003, dated January 2024) publishes a global "BeiDou Visible Satellites Number" map, colour-coded by the number of BeiDou satellites simultaneously visible at every location on Earth [9]. The text explains the fundamental requirement that at least four satellites must be observed simultaneously to solve for the three position parameters and the receiver clock offset, and thereby delineates the basic coverage envelope of the system. The parent section of the CSNO portal additionally hosts constellation status information, signal characteristics and interface control documents, and notices of on-orbit maintenance operations [10].

In the context of this project, the official visibility map provides the *nominal baseline* of BeiDou coverage: the number of BeiDou satellites a receiver should expect to see as a function of latitude. When a spaceborne receiver shows a BeiDou-specific degradation — a drop of tracked BeiDou signals without a corresponding GPS or Galileo degradation — the official visibility and status pages are consulted to decide whether the behaviour is consistent with the published constellation state or points to a local interference or receiver issue.

### Entry 5 — GLONASS Application Consumer Center — Information and Analytical Center (IAC)

**Primary URL:** https://glonass-iac.ru

**Ionosphere service:** https://glonass-iac.ru/en/iono/about/

**SDCM graphic (ESA):** https://www.esa.int/ESA_Multimedia/Images/2013/07/Russia_s_SDCM_system

The Information and Analytical Center for Time Coordinates and Navigation Support (IAC) of TsNIIMash / Roscosmos operates the GLONASS Application Consumer Center, the official Russian portal for navigation-support information [11]. The centre performs monitoring and analysis of all four global constellations — GLONASS, GPS, Galileo and BeiDou — together with an ionosphere section that publishes regional total-electron-content products and their methodology [12], and acts as the national collection centre for navigation monitoring data. The portal publishes constellation status, almanac and clock information, and the official documentation of the Russian navigation infrastructure, including the System of Differential Corrections and Monitoring (SDCM) [13].

The SDCM is Russia's satellite-based augmentation system (SBAS), developed by Roscosmos as the functional counterpart of the American WAAS and the European EGNOS. As documented by ESA and by the United Nations Office for Outer Space Affairs, SDCM is conceived to perform integrity monitoring of both GPS and GLONASS satellites and to broadcast differential corrections over the Russian Federation and neighbouring regions [14][15]; the ESA multimedia page catalogued above provides the classical system diagram. For interference monitoring over Eurasia, the IAC is the authoritative regional source: it is the only operator that combines official GLONASS status, multi-constellation monitoring and a dedicated ionosphere service for the same territory, and its ionosphere products serve as an additional exclusion layer for anomalies observed over Russian and CIS regions.

### Entry 6 — GNSS Interference Monitoring from LEO (ESA NAVISP EL2-117)

**Primary URL:** https://navisp.esa.int/uploads/files/documents/NAVISP%20EL2-117%20Presentation%20Final.pdf

**Project page:** https://navisp.esa.int/project/117-gnss-interference-monitoring-from-leo

This entry is the most direct methodological anchor of the whole catalogue. NAVISP EL2-117, "GNSS Interference Monitoring from LEO", is a completed project of Element 2 (Competitiveness) of ESA's NAVISP programme, started on 23 July 2021 and carried out on the commercial LEO constellation of Spire Global [16]. The project's objective is to detect interference against GNSS receivers — both jamming and spoofing — from orbit and to geolocate the sources on the ground: it builds on the GNSS radio-occultation receivers already flying on the Spire constellation, develops the detection and geolocation signal processing, identifies the geographic areas where detection is statistically likely, and demonstrates the concepts on real flight data [16]. The final presentation PDF at the URL above summarises methods, results and lessons learned.

The relevance to Sentinel Jam is structural, not incidental: EL2-117 demonstrates, at industrial scale and on an independent platform, that correlated C/N0 degradations observed by LEO-borne GNSS receivers are a viable, publishable and fundable observable for terrestrial interference detection — exactly the principle on which Sentinel Jam is built, applied there to the Spire fleet and here to the Sentinel-2 fleet. The peer-reviewed foundation of the same approach was laid by Murrian et al., who analysed three years of LEO GNSS telemetry and showed systematic detection of known jamming areas [17]. Every positive control performed in this project (Black Sea, eastern Mediterranean, Baltic) is consistent with the detection geography described in these two independent sources.

### Entry 7 — Ionosphere Monitoring and Prediction Center — TEC Archive (DLR)

**Archive URL:** https://impc.dlr.de/SWE/Total_Electron_Content/TEC_Near_Real-Time/DLR_GNSS_GCG_L4_VTEC-NTCM-SCM_NC_EUROPE/v2.0.0/

**Service portal:** https://impc.dlr.de

The IMPC publishes the near-real-time European TEC product of Entry 3 not only as a live map but also as a versioned public archive of the generated images. The URL above opens the directory of the `DLR_GNSS_GCG_L4_VTEC-NTCM-SCM_NC_EUROPE` product, version 2.0.0, where the historical near-real-time TEC maps of Europe are stored by date and time and remain freely browsable after publication [6][7]. The archive is the retrospective mirror of the live service: what the operational map shows today, the archive preserves for tomorrow.

The archive is the instrument of the *retrospective half* of the exclusion workflow. When an anomaly is discovered late — for example during a batch analysis of past Sentinel-2 RINEX files, which are published with a latency of several days after acquisition — the live IMPC map for that epoch no longer exists on screen, but the archived images allow the ionospheric conditions of exactly that epoch to be reconstructed. In the validation methodology of the companion report, this archive is the designated source for re-classifying historical European detections: an anomaly co-registered with an archived ionospheric storm is attributed to the ionosphere; an anomaly during quiet archived conditions retains its interference candidacy.

### Entry 8 — Gravitational Time Dilation and the DLR Satellite-Clock Monitoring Portal

**Primary URL:** https://gnss-monitoring.dlr.de/satellite-clock

The DLR GNSS performance-monitoring portal includes a satellite-clock section that tracks the temporal behaviour of the navigation satellite clocks of the monitored constellations: broadcast clock corrections, their drift, discontinuities (clock steps) and stability [18]. Clock-step events are among the few receiver- and platform-level artefacts that can mimic interference or spoofing signatures in GNSS telemetry; the validation report of this project documents real millisecond-scale clock steps observed on Sentinel-2C and used as natural test vectors. An independent, continuously updated view of what the satellite clocks themselves are doing — as provided by this portal — is therefore a valuable third-party instrument for separating satellite-side events from propagation-side events.

*The relativistic background, and a note on an apparent Doppler effect.* GNSS satellite clocks do not tick at the same rate as clocks on the ground, and this is engineered into the system. At the altitude of a GPS satellite (~20,200 km), the weaker gravitational potential makes the onboard clock run faster than a ground clock by approximately +45.7 microseconds per day (gravitational time dilation, general relativity), while the satellite's orbital velocity makes it run slower by approximately −7.2 microseconds per day (kinematic time dilation, special relativity); the net effect is ≈ +38.6 microseconds per day, compensated at factory by offsetting the nominal 10.23 MHz clock frequency to 10.22999999543 MHz (fractional offset −4.4647 × 10⁻¹⁰) [19][20]. On top of this secular offset sits a periodic modulation driven by orbital eccentricity: the satellite's radial motion modulates the gravitational potential it experiences, and the standard broadcast correction — proportional to e·√a·sin E — reaches approximately ±23 ns for a GPS eccentricity of 0.01, and up to ≈ 45 ns for the largest broadcast eccentricities [21].

This relativistic periodic term is very likely what produces the "Doppler-like" appearance noticed by the author in clock-residual plots: a smooth, orbit-locked sinusoidal wiggle, mathematically reminiscent of a Doppler signature. The distinction is worth stating precisely, because the two effects live in different observables. The classical Doppler shift is a *carrier-frequency* offset produced by the line-of-sight relative velocity between satellite and receiver — it reaches a few kHz for ground users (≈ ±4–5 kHz at GPS L1) and ≈ ±40 kHz for a LEO receiver such as Sentinel-2 — and it is absorbed by the receiver's tracking loops and reported, when at all, as a separate Doppler observable. It does *not* appear in clock-offset plots. The periodic component visible in clock plots is instead dominated by the relativistic eccentricity modulation (nanoseconds, orbital period, same sinusoidal shape) and, secondarily, by orbit- and clock-determination residuals. The observation reported by the author is therefore real and explainable, but its physical origin is relativistic and estimation-side rather than a classical Doppler shift [19][20][21].

### Entry 9 — galmon.eu — Coverage and DOP Maps

**Primary URL:** http://galmon.eu/geo/coverage.html

**Project portal:** https://galmon.eu — **Source code:** https://github.com/berthubert/galmon

galmon is an open-source and open-data GNSS monitoring project initiated by Bert Hubert, run by a worldwide community of volunteer station operators, that monitors GPS, GLONASS, Galileo and BeiDou together with the space-based augmentation systems (WAAS, EGNOS, GAGAN) [22]. The project ingests raw receiver data from the volunteer network and publishes, fully openly: live observer maps, per-satellite ephemeris, almanac and clock health, signal-health flags, orbit-versus-TLE comparisons, and — the entry relevant here — the geo/coverage map, which shows where a position fix is possible (enough satellites above the horizon) and companion HDOP / VDOP / PDOP maps, which show where the expected positioning precision will be poor [22].

The value of galmon for interference work is its total transparency: every satellite flagged unhealthy by the community network is traceable to the raw navigation bits that produced the flag, and the official Galileo constellation status is cross-referenced with the European GNSS Service Centre notifications (NAGUs). When Sentinel Jam observes a coverage gap or a single-satellite degradation, galmon provides an independent, ground-based answer to the question "was the satellite itself healthy, and was coverage nominal at that place and time?" — one more exclusion layer that costs nothing to consult.

## 3. Part B — Ground reference networks: CORS, RTK and RINEX over Russia and Eurasia (Entries 10–13)

The four entries of this Part are geodetic reference networks of a different class from the monitoring dashboards of Part A. A CORS (Continuously Operating Reference Station) network is a set of permanently installed, geodetically surveyed GNSS receivers that stream observations continuously; an RTK (Real-Time Kinematic) service distributes differential corrections from those stations to rover receivers over the NTRIP protocol (typically RTCM3 messages), enabling centimetre-level real-time positioning; and the same stations archive their raw observations in RINEX format, the standard exchange format of the GNSS community, for post-processing, PPP (Precise Point Positioning) and research. The four networks below operate over the Russian Federation and neighbouring Eurasian regions — territories of direct interest for this project, since they include several of the interference areas detected from orbit — and together cover the commercial (Leica, Topcon), aggregation-platform and community segments of the market.

These networks contribute to the validation workflow in two complementary ways. First, their RINEX archives constitute ground-based observation records for the same epochs and territories observed by Sentinel-2 from orbit: a ground receiver inside a jammed area records the event as a loss of tracking and a C/N0 depression from below, while Sentinel-2 records it from above, and co-registration of the two views is a powerful corroboration. Second, their health-monitoring layers (Entry 13 in particular) provide an independent picture of where ground infrastructure itself is reporting anomalies.

### Entry 10 — SmartNet Russia

**Primary URL:** https://smartnetrtk.ru — **Global service brand:** https://hxgnsmartnet.com/services/smartnet-nrtk

SmartNet Russia is the Russian instance of SmartNet, the Network RTK correction-service brand of Leica Geosystems (Hexagon), operated commercially in cooperation with the manufacturer [23]. The global SmartNet concept is built on the Leica GNSS Spider reference-station software platform and delivers open-standard Network RTK corrections that can be consumed by any GNSS rover over the mobile internet; the Russian deployment covers the European part of the country and provides, in addition to real-time corrections, RINEX data download from the reference stations. Within the network, standard RTK, DGNSS and VRS (Virtual Reference Station) correction modes are offered, so that a user anywhere inside the coverage area receives corrections synthesised for the user's own position rather than from a single distant base.

In this catalogue, SmartNet Russia represents the high-end commercial reference infrastructure over European Russia. Its RINEX archives are the practically relevant ground dataset for this project's territory class: geodetic-quality, continuously operating stations, distributed across the same regions where several of the orbit-detected interference areas (Black Sea region, western Russia, Baltic approaches) are located.

### Entry 11 — Topnet Live Russia

**Primary URL:** https://topnet-gsi.ru — **Global service brand:** https://www.topconpositioning.com/us/en/solutions/technology/infrastructure-software-and-services/topnet-live-corrections

Topnet Live Russia is the Russian deployment of Topcon's Topnet Live real-time GNSS correction service, operated on the TopNET+ platform by GSI, Topcon's partner in Russia [24]. Topnet Live is Topcon's global real-time correction infrastructure: Topcon manages, operates and monitors the reference networks of the service, delivering high-quality RTK corrections to receivers used in surveying, construction, GIS, mapping and agriculture [24]. The Russian instance provides NTRIP real-time data streams for network RTK positioning and PPP (Precise Point Positioning) services for areas outside dense network coverage.

Within the catalogue, Topnet Live Russia is the second commercial pillar of the Russian reference infrastructure and complements Entry 10 on both the manufacturer side (Topcon rather than Leica) and the service side (explicit PPP delivery in addition to network RTK). Its reference stations contribute additional ground RINEX coverage over the monitored territories, and the PPP mode is of particular methodological interest: PPP solutions are sensitive to clock and orbit quality and to ionospheric conditions, which makes them a usable, if coarse, ground-side indicator of the same propagation anomalies that Sentinel-2 observes from orbit.

### Entry 12 — HIVE — RTK and RINEX from reference stations

**Primary URL:** https://hive.geosystems.aero — **Operator:** https://geosystems.aero

HIVE is a cloud platform of the Russian company Industrial Geodetic Systems R&D (geosystems.aero) that aggregates continuously operating reference stations from multiple contributing networks and organisations into a single subscription service [25]. The platform's own description is concise and accurate: "HIVE is a source of RTK corrections and RINEX files from reference stations. HIVE connects 61 Russian regions" [25]. The service therefore offers two deliverables to its users: real-time RTK corrections over NTRIP for surveying and positioning work, and automated RINEX file download from the aggregated station base, with station geography and pricing documented on the platform's map.

Within this catalogue, HIVE is the aggregation-platform representative: rather than deploying its own homogeneous manufacturer network, it connects heterogeneous reference stations across about sixty regions of the Russian Federation and exposes them through one interface. For the validation workflow this is operationally convenient, because a territory-wide ground-RINEX request (for example, all available stations inside a hexagonal anomaly cell during a suspected jamming event) can be addressed through a single platform instead of through many individual network operators.

### Entry 13 — PiDATA — GNSS base-station network and node monitoring

**Primary URL:** https://pidt.net — **Wiki:** https://wiki.pidt.net/books/pigopidata/page/pidata-base-station — **GIS view:** https://gis.pidt.net

PiDATA is a distributed, community-operated GNSS base-station network with an associated node-monitoring layer [26]. The network model is deliberately simple: an operator installs a PiDATA base station at their location, connects it to the internet and to power, and the station automatically determines its coordinates and begins streaming RTCM3 correction messages to the PiDATA NTRIP caster (the public caster endpoint runs on pidt.net, port 1234, as documented in the project wiki) [26]. The platform monitors the status and health of the connected GNSS nodes in real time and exposes the network through a GIS interface [26], giving an at-a-glance picture of which nodes across Eurasia are online, streaming and healthy.

PiDATA rounds out Part B with the community and diagnostics segment. Its monitoring layer is the element most relevant to this project: a real-time map of ground GNSS node health across Eurasia is, in effect, a coarse ground-based anomaly screen, and anomalies co-observed by the PiDATA network (nodes losing fix quality or dropping out) at the epoch of a Sentinel-2 detection provide independent ground-side corroboration of the same event.

---

## 4. Cross-resource comparison tables

The three tables below summarise the thirteen entries by functional class, in a compact form suitable for quick consultation during operations.

**Table 1 — Space-weather and ionospheric monitoring services (Entries 1, 2, 3, 7).**

| # | Service | Operator | Coverage | Primary products | Update |
| --- | --- | --- | --- | --- | --- |
| 1 | GPS Community Dashboard | NOAA / SWPC (USA) | global | conditions on NOAA R/S/G scales, TEC products, radio-blackout nowcasts | minutes |
| 2 | Current Space Weather (SWE Network) | ESA (Europe) | global | solar wind, flares, SEP events, geomagnetic and ionospheric products | minutes–hours |
| 3 | NRT TEC map, Europe | DLR / IMPC (Germany) | Europe + global map | VTEC maps (NTCM-SCM, `DLR_GNSS_GCG_L4_VTEC-NTCM-SCM_NC_EUROPE`), 1-h forecasts, storm and TID warnings | ≈ 5 min |
| 7 | TEC archive | DLR / IMPC (Germany) | Europe | historical NRT TEC map images, product version 2.0.0 | on publication |

**Table 2 — Constellation-level, relativistic and independent monitoring (Entries 4, 5, 6, 8, 9).**

| # | Resource | Operator | Scope | Key outputs |
| --- | --- | --- | --- | --- |
| 4 | BeiDou Monitoring and Evaluation | CSNO (China) | BeiDou | visible-satellite number map, constellation status, ICDs, maintenance notices |
| 5 | GLONASS IAC / Application Consumer Center | Roscosmos / TsNIIMash (Russia) | GLONASS, GPS, Galileo, BeiDou, ionosphere, SDCM | constellation status, almanac/clock data, regional TEC, differential-correction documentation |
| 6 | NAVISP EL2-117 — GNSS Interference Monitoring from LEO | ESA NAVISP / Spire | GNSS jamming and spoofing from orbit | detection and geolocation algorithms, detection-geography analysis, flight demonstrations |
| 8 | GNSS Monitoring Portal — Satellite Clocks | DLR (Germany) | GNSS satellite clocks | broadcast clock corrections, drift, steps, stability |
| 9 | galmon.eu | community (B. Hubert) | GPS, GLONASS, Galileo, BeiDou + SBAS | coverage map, HDOP/VDOP/PDOP maps, ephemeris and signal health, NAGU cross-references |

**Table 3 — Ground reference networks: CORS, RTK and RINEX providers (Entries 10–13).**

| Network | Technology platform | Services | Website | Coverage |
| --- | --- | --- | --- | --- |
| SmartNet Russia | Leica (Hexagon) SmartNet, GNSS Spider platform | RTK, DGNSS and VRS corrections; RINEX download | smartnetrtk.ru | European Russia |
| Topnet Live Russia | Topcon TopNET+ platform (GSI) | NTRIP real-time streams; PPP | topnet-gsi.ru | Russia |
| HIVE | cloud CORS aggregation (Industrial Geodetic Systems R&D) | RTK subscription; automated RINEX download | hive.geosystems.aero | 61 Russian regions |
| PiDATA | community base stations + NTRIP caster (RTCM3) | RTK corrections via NTRIP; real-time node status monitoring | pidt.net | Eurasia |

**Table 4 — Role of each entry in the interference-monitoring workflow.**

| Workflow stage | Entries | Question answered |
| --- | --- | --- |
| Space-weather and ionospheric exclusion | 1, 2, 3, 7 | Could the anomaly be a geomagnetic storm, scintillation, a solar radio burst or a TID? |
| Constellation-health cross-check | 4, 5, 9 | Is the affected satellite officially unhealthy, or was coverage nominal? |
| Instrument and clock sanity | 8 | Is a documented satellite clock event a benign explanation? |
| Independent method anchor | 6 (with [17]) | Is space-based interference detection an established, independently demonstrated method? |
| Ground-side data and corroboration | 10, 11, 12, 13 | What did ground receivers in the same area and epoch observe? |

---

## 5. Concluding remarks

The thirteen entries of this catalogue form the external reference frame against which the Sentinel Jam monitor is operated and validated. The design principle behind the selection is economical: each entry closes a specific epistemic gap that a single-project tool cannot close by itself — no project can operate its own global ionosphere service, its own constellation status authority, its own relativistic clock monitor or its own ground CORS fleet, and none needs to, because official, community and commercial infrastructures already publish these views openly. The catalogue also documents, through Entry 6, that the space-based detection principle used by the project is independently demonstrated by an ESA-funded industrial programme on a different LEO constellation, which is the strongest single piece of external context supporting the method.

The catalogue is intended to be maintained as a living document: monitoring portals evolve, archives migrate, and the Russian reference-network market in particular is dynamic. Additions and corrections follow the same review discipline as the rest of the project — an entry is added only when its operator, access conditions and role in the workflow are verifiable from public documentation.

---

## References

[1] NOAA Space Weather Prediction Center, "Global Positioning System (GPS) Community Dashboard". https://swpc.noaa.gov/communities/global-positioning-system-gps-community-dashboard

[2] NOAA Space Weather Prediction Center, "NOAA Space Weather Scales — explanation". https://www.swpc.noaa.gov/noaa-scales-explanation

[3] ESA Space Weather Service Network, "Current Space Weather" dashboard. https://swe.ssa.esa.int/current-space-weather

[4] ESA Space Weather Service Network — portal. https://swe.ssa.esa.int

[5] DLR (German Aerospace Center), Ionosphere Monitoring and Prediction Center (IMPC) — portal. https://impc.dlr.de — research-infrastructure page: https://www.dlr.de/en/research-and-transfer/research-infrastructure/ionosphere-monitoring-and-prediction-center-impc-en

[6] DLR IMPC, "Near Real-Time TEC Map, Europe" (product `DLR_GNSS_GCG_L4_VTEC-NTCM-SCM_NC_EUROPE`, v2.0.0). https://impc.dlr.de/products/total-electron-content/near-real-time-tec/near-real-time-tec-map-europe

[7] DLR IMPC, TEC Near Real-Time product archive (image directory, version 2.0.0). https://impc.dlr.de/SWE/Total_Electron_Content/TEC_Near_Real-Time/DLR_GNSS_GCG_L4_VTEC-NTCM-SCM_NC_EUROPE/v2.0.0/

[8] E. Schmölter et al., "Improving the global Total Electron Content maps provided by the Ionosphere Monitoring and Prediction Center", *Journal of Space Weather and Space Climate*, vol. 15, 2025. https://www.swsc-journal.org/articles/swsc/full_html/2025/01/swsc250009/swsc250009.html

[9] China Satellite Navigation Office (CSNO), "BeiDou Visible Satellites Number" — Monitoring and Evaluation, document 2024-003, January 2024. http://en.beidou.gov.cn/SYSTEMS/Monitoringandevaluation/202401/t20240103_27615.html

[10] China Satellite Navigation Office — BeiDou portal, Systems section (Monitoring and Evaluation). http://en.beidou.gov.cn/SYSTEMS/System

[11] Information and Analytical Center for Time Coordinates and Navigation Support (IAC, TsNIIMash / Roscosmos), GLONASS Application Consumer Center — portal. https://glonass-iac.ru

[12] IAC, "Ionosphere" — service description. https://glonass-iac.ru/en/iono/about/

[13] IAC, "System documents" (GLONASS, SDCM). https://glonass-iac.ru/en/documents

[14] ESA Navipedia, "SDCM — System for Differential Corrections and Monitoring". https://gssc.esa.int/navipedia/index.php/SDCM

[15] I. Revnivykh, "GLONASS and SDCM Status and Development", presentation at the 14th meeting of the International Committee on GNSS (ICG-14), UNOOSA, 2019. https://www.unoosa.org/documents/pdf/icg/2019/icg14/03.pdf

[16] ESA NAVISP, "EL2-117 — GNSS Interference Monitoring from LEO", project page and Final Presentation. Project page: https://navisp.esa.int/project/117-gnss-interference-monitoring-from-leo — Final presentation (PDF): https://navisp.esa.int/uploads/files/documents/NAVISP%20EL2-117%20Presentation%20Final.pdf

[17] M. J. Murrian, L. Narula, P. A. Iannucci, S. Budzien, B. W. O'Hanlon, M. L. Psiaki, T. E. Humphreys, "First results from three years of GNSS interference monitoring from low Earth orbit", *NAVIGATION: Journal of the Institute of Navigation*, vol. 68, no. 4, pp. 673–683, 2021. https://doi.org/10.1002/navi.449 — preprint: arXiv:2009.04093

[18] DLR, "GNSS Performance Monitoring — Satellite Clocks". https://gnss-monitoring.dlr.de/satellite-clock

[19] N. Ashby, "Relativity in the Global Positioning System", *Living Reviews in Relativity*, vol. 6, art. 1, 2003. https://doi.org/10.12942/lrr-2003-1

[20] GPS World, "Inside the box: GPS and relativity", October 2023. https://www.gpsworld.com/inside-the-box-gps-and-relativity

[21] ESA Navipedia, "Relativistic Clock Correction". https://gssc.esa.int/navipedia/index.php/Relativistic_Clock_Correction

[22] galmon.eu — open GNSS monitoring: coverage map http://galmon.eu/geo/coverage.html ; portal https://galmon.eu ; source code https://github.com/berthubert/galmon ; B. Hubert, "The Galmon GNSS Monitoring Project", 2020, https://berthub.eu/articles/posts/galmon-project

[23] Leica Geosystems (Hexagon), "SmartNet NRTK" — service description: https://hxgnsmartnet.com/services/smartnet-nrtk — Russian instance: https://smartnetrtk.ru

[24] Topcon Positioning Systems, "Topnet Live Corrections" — service description: https://www.topconpositioning.com/us/en/solutions/technology/infrastructure-software-and-services/topnet-live-corrections — Russian instance: https://topnet-gsi.ru

[25] Industrial Geodetic Systems R&D, "HIVE — RTK and RINEX from reference stations". https://hive.geosystems.aero/landing?locale=en — operator: https://geosystems.aero

[26] PiDATA — GNSS base-station network: https://pidt.net — wiki (base-station operation, NTRIP caster RTCM3): https://wiki.pidt.net/books/pigopidata/page/pidata-base-station — GIS view: https://gis.pidt.net
