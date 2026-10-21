# Sentinel Jam — Scientific and Technical Validation Report

**System under validation:** Sentinel Jam, GNSS / RFI interference monitor based on Copernicus Sentinel-2 GNSS receivers (standalone application `sentinel-jam-monitor.html`, version v11.10.0)

**Report type:** Scientific and technical validation report (partial validation)

**Author:** GiamMa-based researchers SDR R&amp;D IoT | @giammaiot2 — https://x.com/giammaiot2

**Document version:** 1.0 — 21 October 2026

**Availability:** the system under validation is open source and runs entirely client-side; the data used are open Copernicus products. All figures quoted in this report are reproducible from the regression scripts and datasets described in Appendix D.

---

## Executive summary

Sentinel Jam is a browser-based monitor that surveys GNSS radio-frequency interference (RFI) using the precision-orbit-determination (POD) GNSS receivers flown on the Copernicus Sentinel-2 satellites as worldwide, space-based detectors. It retrieves the daily GNSS RINEX observation products (AUX_GNSSRD) published as open data by the Copernicus Data Space Ecosystem (CDSE), analyses them epoch by epoch directly in the browser, and displays correlated carrier-to-noise-density-ratio (C/N0) degradations on an interactive world map along the satellites' ground tracks [1][4][5].

This report documents what the project set out to do, what data it operates on, what the code actually computes, how the dashboard presents results, and — most importantly — the extent to which the method and its implementation have been validated so far. Validation is performed by triangulation across independent evidence sources, because no ground-truth database of terrestrial GNSS jammers exists that a spaceborne screen could be compared against. Ten validation vectors are described and assessed: internal consistency of the data record, physical-plausibility checks of the parser, calibration of the detection thresholds on real data, positive controls against known jamming regions, an in-depth investigation of the "South Pacific anomaly" (a suspicious geophysical result that turned out to be a geolocation rendering error, itself a valuable validation outcome), cross-platform consistency between two different receivers observing the same events, cross-validation against an independent reimplementation of the pipeline, synthetic-injection testing, real receiver anomalies used as natural test vectors, and an empirical verification of the publication pipeline over 250 catalogue releases.

The overall outcome can be summarised in four statements. First, the detection method is a *screening* technique based on C/N0 telemetry: it reliably surfaces correlated, multi-satellite signal degradations, and its positive detections over known jamming areas (Black Sea, eastern Mediterranean, Baltic, and others) are consistent with independent public evidence [20][21][22]. Second, the implementation has been exercised against synthetic faults — injected spoofing ramps, receiver-clock steps, artificial coverage holes — and against real receiver anomalies (millisecond-scale clock steps observed on Sentinel-2C), with the expected behaviour in every case. Third, one significant geolocation error was found, root-caused and eliminated during development (the v11.7 orbital-phase correction); the investigation that led to it, triggered by the South Pacific anomaly, is documented here as the strongest single piece of evidence that anomaly triage on this system works as designed. Fourth, the validation remains *partial* by design: formal detection statistics (probability of detection, false-alarm rate) over large blind datasets, systematic correlation with aviation-based maps, and ionospheric-activity screening against geomagnetic and scintillation indices are identified as the priority extensions of this work (Section 10).

This document is written so that a reader with a general engineering background can understand the system end to end, and so that a reviewer with GNSS or RFI expertise can locate every quantitative claim and re-derive it. Section 2 sets the scientific context; Sections 3–6 describe hardware, data, algorithms and dashboard; Section 7 is the validation core; Section 8 records the development and regression history; Sections 9–10 discuss limitations, comparisons and future work; appendices provide reference tables and reproducibility notes.

---

## 1. Introduction

### 1.1 Purpose and scope of this report

The purpose of this report is to provide a structured, evidence-based account of the Sentinel Jam system, covering both its scientific rationale and the degree of confidence that can currently be placed in its outputs. It is deliberately written as a *validation* report rather than a purely descriptive manual: every functional statement about the code is paired, wherever possible, with the test, measurement or comparison that supports it. Where evidence is indirect or incomplete, this is stated explicitly rather than glossed over, because the honest delineation of what is and is not yet validated is one of the primary deliverables of the document.

The scope covers version v11.10.0 of the standalone application, the companion local server (`sentinel-server.py`), the offline dataset downloader (`download-datasets.py`), and the three auxiliary Sentinel-2 data products the system consumes (AUX_GNSSRD, AUX_POEORB, AUX_PROQUA). The report does not cover the earlier Next.js prototype lineage of the project in detail, although that lineage contributed several design decisions that are mentioned where relevant. All content refers to the published English documentation and to the source code as shipped in the repository.

### 1.2 What the project set out to do

The original objective of the project is to **identify radio-frequency anomalies — in particular terrestrial and airborne GNSS jamming — from satellite observations**, filling a coverage gap that ground-based monitoring cannot close. Terrestrial GNSS jammers are illegal, mobile, and difficult to instrument from the ground, and interference hotspots frequently sit in regions with no monitoring infrastructure at all. A satellite in low Earth orbit (LEO) that overflies an interfering region sees the event from above: while its line of sight crosses the affected area, several GNSS satellites tracked simultaneously show correlated drops of the received C/N0. This signature is physically meaningful, computationally cheap to extract from standard POD telemetry, and — as the literature on spaceborne interference observation has demonstrated — sufficient both to detect and, under favourable geometry, to geolocate interference sources [9][10].

From this general objective, the project defined a set of concrete requirements:

- **O1 — Global, continuous screening.** Exploit a GNSS receiver that operates continuously on a sun-synchronous platform with a repeating ground track, so that every region of the Earth is revisited systematically and comparisons between days are meaningful.
- **O2 — Open data only.** Rely exclusively on openly published products (Copernicus auxiliary data), so that the entire analysis is reproducible by anyone with a free account or with the provided offline datasets.
- **O3 — Multi-criteria detection.** Combine an absolute C/N0 threshold rule, a per-satellite relative-baseline rule, and a cross-constellation rule, so that receivers with different sensitivities and different constellations produce comparable detections.
- **O4 — Attribution and classification.** Separate GPS-only, Galileo-only and wideband (cross-constellation) events, report coverage gaps, and flag high-latitude events where ionospheric scintillation is the most likely natural cause.
- **O5 — Anti-spoofing and receiver-quality screening.** Derive, from observables alone (no positions, no Doppler), physical-consistency checks that can distinguish spoofing-like dynamics and receiver malfunctions from ordinary interference.
- **O6 — Usability and privacy.** Deliver the whole analysis in a single HTML file with an optional local helper, require credentials only once against the official Copernicus identity service, and keep all processing client-side.
- **O7 — Verifiability.** Ship reproducible regression tests and offline datasets so that every release can be re-exercised without network access.

### 1.3 Validation terminology and the meaning of "partial"

Because no authoritative, geolocated, time-stamped catalogue of every terrestrial GNSS jammer exists, a spaceborne screening tool cannot be validated in the classical instrument-validation sense against ground truth. The validation performed here is therefore **partial and triangulated**: it combines positive controls (regions where public reporting consistently documents jamming), negative expectations (quiet ocean and polar baselines), internal-consistency checks, independent-reimplementation cross-checks, synthetic fault injection, and exploitation of real receiver anomalies as natural test vectors. Throughout this report the following convention is used: an outcome is labelled **confirmed** when at least two independent evidence sources agree; **consistent** when a single strong evidence source supports it; **preliminary** when the evidence is limited to isolated cases; and **not yet validated** when only design arguments exist.

### 1.4 Document structure

Section 2 describes the scientific and technical context of the ecosystem the system belongs to. Section 3 characterises the hardware under analysis. Section 4 details the data products, their structure, metadata and access path. Section 5 describes the processing pipeline and every detection rule with its parameters. Section 6 describes the dashboard and the anomaly classification levels. Section 7 presents the validation methodology and results, including the South Pacific anomaly case study. Section 8 records the development history and the regression evidence. Section 9 discusses strengths, comparisons with independent services and limitations. Section 10 draws conclusions and defines future work. References and appendices close the document.

---

## 2. Scientific and technical context

### 2.1 GNSS observables and the C/N0 signature of interference

Every GNSS receiver used for orbit determination reports, per satellite and per epoch, a set of standard observables: pseudorange (code), carrier phase, and the carrier-to-noise-density ratio C/N0, expressed in dB-Hz. C/N0 measures the ratio of the received carrier power to the noise power spectral density in 1 Hz, and for a healthy spaceborne POD receiver typically sits in the 40–50 dB-Hz range; the median L1 C/N0 measured on Sentinel-2B daily files is 47.3 dB-Hz, a value used in this project as a physical reference for a healthy instrument (Section 7.3). The RINEX format in which these observables are exchanged is the de-facto standard of the geodetic community [6].

Terrestrial interference acts on this observable in a characteristic way. A broadband jammer raises the effective noise floor seen by the receiver front-end, which depresses the measured C/N0 of *every* satellite whose line of sight crosses the illuminated area — not of one satellite only. This correlated, multi-satellite depression, persisting over the seconds-to-minutes scale of an overflight, is the fundamental detection signature of Sentinel Jam. Single-satellite dips, by contrast, are dominated by normal geometry (a satellite setting near the antenna pattern horizon), multipath, or isolated cycle slips, and form the statistical noise background that the detection rules must reject. The distinction between correlated and uncorrelated degradation is therefore not an implementation convenience but the physical core of the method, a principle that echoes the wider literature on interference detection from low-cost and spaceborne receivers [9][10][16].

Jamming must also be distinguished from spoofing. A jammer denies service by raising the noise floor; a spoofer transmits counterfeit signals that imitate the real ones and can shift a receiver's position or time solution. For a POD-quality receiver, spoofing manifests as *non-physical* consistency between observables — for example, code and carrier rates that diverge in ways the satellite geometry cannot produce — rather than as a simple loss of signal quality [11][12]. This is why the project treats jamming detection (Section 5.5) and kinematic-consistency screening (Section 5.7) as two complementary layers rather than as one algorithm.

### 2.2 Natural confounders

A responsible detection system must account for the natural and technical effects that mimic the interference signature. Four families matter here, and all four appear in the results and in the user guidance.

**Ionospheric scintillation.** Rapid fluctuations of signal amplitude and phase, produced by small-scale electron-density irregularities, cause correlated multi-satellite C/N0 fades that are climatologically concentrated at high latitudes (above roughly 55 degrees, both hemispheres) and in the equatorial evening/night sector, where plasma bubbles form over the Atlantic and Pacific sectors [17][18]. Scintillation is the dominant natural false-positive source for any C/N0-based screen, and for this reason the dashboard tags high-latitude detections and the operating documentation instructs the user to weight them accordingly.

**Solar radio bursts.** Intense solar radio emission in the L-band can raise the noise floor hemisphere-wide and depress C/N0 on all sunlit satellites simultaneously; the December 2006 events demonstrated degradations of tens of dB-Hz on GPS receivers over the dayside of the Earth [19]. Because bursts are daylight-dependent and globally coherent, they produce a characteristic "whole-hemisphere, same-epoch" pattern that an operator can recognise in the timeline view.

**Multipath and low-elevation geometry.** Reflections from large structures, and satellites observed near the edge of the antenna pattern, reduce the C/N0 of individual satellites. These effects are largely excluded by the multi-satellite rules, but they remain visible in the per-satellite charts and are the reason the absolute rule requires at least one third of the tracked constellation to dip simultaneously.

**Receiver and platform events.** Clock steps, cycle slips, receiver reboots and hardware ageing all inject artefacts into the record. Far from being only a nuisance, these events are exploited in this project as natural test vectors: real millisecond-scale clock steps observed on Sentinel-2C are used to verify the kinematic-consistency checks (Section 7.9).

### 2.3 The interference-monitoring ecosystem

Sentinel Jam is not an isolated instrument but one node in a growing ecosystem of GNSS-interference monitoring services, each with a different sensing principle. Aviation-based services exploit the ADS-B navigation-integrity information transmitted by civil aircraft: GPSJam [22] builds daily worldwide maps of GNSS accuracy degradation reported by aircraft; Stanford University operates two complementary dashboards, a GNSS interference detection service based on WAAS/NAS aviation data [23] and a research-grade ADS-B/NIC interference map maintained by the GPS Research Lab [24]; Flightradar24 publishes a GPS jamming layer derived from its global receiver network [25]; and Wingbits provides a live jamming/spoofing map [26]. Aviation services are excellent over traffic-dense regions (Europe, the Middle East, East Asia) but blind over oceans, polar routes and low-traffic airspace. Ground networks of low-cost front ends demonstrate dense local coverage where deployed [16]. Regulatory bodies track the operational impact: EASA has issued and repeatedly updated safety information bulletins on GNSS interference [20], and EASA/EUROCONTROL workshops consolidate the European picture [21].

Space-based observation complements this picture precisely where the others are blind: over oceans, conflict zones without aviation reporting, and remote areas. The scientific literature has demonstrated the principle with the FOTON software-defined receiver on the International Space Station, which characterised — and Doppler-geolocated to better than 1 km — a powerful interference source active in Syria from 2017 [9], following earlier spaceborne interference assessments [10]; recent reviews consolidate the state of the art of space-based GNSS interference monitoring [14] and of interference localisation in general [13]. Sentinel Jam brings this observation principle to a fully open, openly documented data chain: rather than raw IF samples or dedicated payloads, it uses the routine telemetry of POD receivers that fly *for another purpose*, which makes the approach inexpensive, continuous and globally uniform.

### 2.4 Why Sentinel-2 POD receivers are suitable detectors

Three properties make the Sentinel-2 GNSS chain an unusually good opportunistic sensor. First, **continuity and uniformity**: the POD receivers track GNSS continuously — every 10 seconds, 8,640 epochs per day per platform — along a sun-synchronous orbit with a 10-day ground-track repeat cycle [5], producing a daily, globally uniform record with stable geometry. Second, **instrument quality**: geodetic-grade receivers with calibrated antennas produce C/N0 of stable statistical character, so that a per-satellite daily median is a meaningful baseline against which drops of 6–8 dB are unambiguous (Section 5.5). Third, **orbit calmness requirements**: the very success of precise orbit determination depends on an undisturbed RF environment; POD processing needs clean carrier phase on many satellites simultaneously, which means any terrestrial interference strong enough to bother POD is, almost by definition, an event worth reporting.

The 10-day repeat cycle has a specific validation value that is exploited in Section 7.2: the same geographic track is re-observed every 10 days, so persistent jamming sources should produce recurring detections on the same geometry, giving an internal persistence test that requires no external data.

### 2.5 Positioning within the author's toolchain

Sentinel Jam is part of a coherent set of open-source radio-monitoring tools by the same group: GNSS-RO Multi-Portal aggregates GNSS radio-occultation anomaly and interference indicators from meteorological and space-situational-awareness feeds [27]; ADS-B Sky Tracker Pro performs aviation-side GNSS jamming and SSR analysis [28]. The three tools observe the same underlying phenomenon from three vantage points — occultation receivers, POD receivers, and aviation transponders — and cross-comparison between them is an intended, though not yet systematic, validation axis (Section 10).

---

## 3. Instrumentation: platforms and hardware under analysis

### 3.1 The Sentinel-2 mission and its orbit

Sentinel-2 is the Copernicus high-resolution optical imaging mission, operated by ESA within the Copernicus Space Component [5]. The imaging payload is the MultiSpectral Instrument (MSI); the GNSS receivers analysed by this project are part of the navigation subsystem that supports precise orbit determination (POD), processed by the Copernicus POD Service (CPOD) [3]. The satellites fly a sun-synchronous orbit at approximately 786 km altitude, inclination 98.568°, with a 10-day repeat cycle of the ground track (5 days when both units of a pair observe) and a local solar time of the descending node near 10:30. The orbital plane parameters are not incidental details: the inclination constant (98.568°) and the per-platform phase offset along the orbit are exactly the two quantities the application's nominal ground-track model uses to place detections on the map, and Section 7.6 documents how an error in one of them (the S2C phase) produced the South Pacific anomaly and was corrected.

Three units are relevant to this project. Sentinel-2A (launched 23 June 2015, COSPAR 2015-028A, NORAD 40697) operated until the failure of its GNSS receiver, after which no AUX products have been published since 21 January 2025. Sentinel-2B (launched 7 March 2017, COSPAR 2017-013A, NORAD 42063) is fully operational on a GPS-only receiver of the 2017 generation. Sentinel-2C (launched 5 September 2024, COSPAR 2024-156A, NORAD 60773) replaced Sentinel-2A in the same orbital slot (180 degrees of phase from Sentinel-2B) and carries the new-generation multi-constellation receiver. The platform inventory is summarised in Table 1.

### 3.2 The GNSS POD receivers

The receivers flown for POD on Sentinel-2 are geodetic-grade spaceborne GNSS units of the PODRIX family produced by RUAG Space, now Beyond Gravity [7]. The PODRIX product line was designed as a precision POD receiver for LEO platforms and, in its later generations, tracks both GPS and Galileo; peer-reviewed descriptions of its deployment on Copernicus successors confirm the redundant multi-GNSS (GPS + Galileo) architecture [8]. Within Sentinel-2 the two generations differ in a way that directly matters to this project:

- **Sentinel-2B (2017 generation, GPS-only).** A RUAG GPS receiver tracking L1 and L2 with typically about 8 satellites visible at any epoch. RINEX observation types per GPS satellite cover code, phase and signal strength on both frequencies (12 observation types, C/L/S families). Its narrower constellation view makes the absolute-threshold rule (Section 5.5) fire more easily, since one third of 8 satellites is fewer than one third of 17; the relative-baseline rule was introduced precisely to keep the two platforms comparable.
- **Sentinel-2C (2024 generation, PODRIX L801, GPS + Galileo).** The multi-constellation receiver tracks typically about 17 satellites (GPS L1/L2 plus Galileo E1/E5a; the Galileo observation set on real files is C1C/L1C/S1C and C5Q/L5Q/S5Q). Its wider view enables the cross-constellation wideband test: a terrestrial broadband jammer illuminates every constellation at once, and simultaneous degradations across GPS *and* Galileo cannot be faked by geometry, multipath, or single-system effects.
- **Sentinel-2A (2015 generation).** The original PODRIX unit, GPS-only, whose failure on 21 January 2025 terminated the platform's AUX record. Historical S2A data remain available and are part of the offline 2018 and 2022 datasets.

### 3.3 Platform inventory

**Table 1.** Platform and receiver inventory as used by Sentinel Jam. Launch, COSPAR and NORAD identifiers are the values embedded in the application's spacecraft panel; receiver characterisation comes from the project's own parsing of AUX_GNSSRD files (constellations, observation types, tracked-satellite statistics) and from the receiver documentation [7][8].

| Platform | Launch date | COSPAR / NORAD | GNSS receiver | Constellations tracked | Typical SV count | Observation content | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Sentinel-2A | 2015-06-23 | 2015-028A / 40697 | PODRIX (RUAG, GPS-only) | GPS L1/L2 | ~8 | code, phase, C/N0, LLI | GNSS receiver failed; no AUX products since 2025-01-21 |
| Sentinel-2B | 2017-03-07 | 2017-013A / 42063 | RUAG GPS receiver, 2017 generation | GPS L1/L2 | ~8 (median L1 C/N0 47.3 dB-Hz) | 12 obs types per SV (C/L/S), no clock field used pre-v11.10 | operational |
| Sentinel-2C | 2024-09-05 | 2024-156A / 60773 | PODRIX L801, 2024 generation | GPS L1/L2 + Galileo E1/E5a | ~17 | GPS: 12 obs types (C/L/S); Galileo: C1C L1C S1C C5Q L5Q S5Q; epoch receiver clock parsed since v11.10 | operational, inherited S2A slot (180° from S2B) |

The orbital geometry is common to the three platforms: inclination 98.568°, sun-synchronous, 10-day ground-track repeat. Sentinel-2A and Sentinel-2C occupy the same slot (nominal ground-track phase 0 in the application model); Sentinel-2B occupies the opposite slot (phase 180°). Every daily analysis window therefore contains up to two files, one per active satellite.

### 3.4 Hardware constraints and their consequences for detection

**Table 2.** Hardware-derived constraints and how the method compensates for them. Each row pairs a property of the flown receivers with the design response in the algorithms (Section 5) and the residual limitation recorded in Section 9.

| Hardware property | Consequence for detection | Design response |
| --- | --- | --- |
| S2B tracks ~8 GPS SV only | Absolute rule fires more easily; single-constellation view cannot separate wideband from GPS-local effects | Relative-baseline rule normalises per platform; cross-constellation test defined only for multi-GNSS receivers (S2C) |
| S2C antenna/processing less jamming-exposed (deeper dips required to lose POD quality) | Absolute C/N0 rule under-reports on S2C relative to S2B | v11.6 relative baseline: per-SV daily median, 6–8 dB dips counted, making S2C contributions visible |
| AUX_GNSSRD contains no positions, velocities or Doppler | No absolute speed test possible; spoofing must be inferred from observable rates | v11.10 kinematic consistency: three metrics independent of the receiver clock (M1 phase-vs-code rate, M2 LOS-acceleration spread, M3 clock steps) |
| 10 s sampling | Events shorter than ~10–20 s can be missed or smoothed | Documented limitation; gap detector measures what is *missing* at multiples of 10 s |
| Receiver clock steps (observed: −10 ms scale on S2C) | Raw clock discontinuities would corrupt rate estimates | Clock-independent metric design; M3 surfaces clock steps explicitly as alerts instead of hiding them |
| Single antenna, no attitude use in detection | Detections are attributed to the sub-satellite point, not the jammer location | Positions explicitly documented as approximate along-track screening results |

Two platform facts deserve emphasis for result interpretation. First, the two operational receivers have different noise and tracking behaviour: the median L1 C/N0 of a healthy S2B file is 47.3 dB-Hz, and any parsed S2C file claiming several satellites near 13 dB-Hz is physically implausible — an inconsistency that this project used, with success, to catch a parser defect (Section 7.3). Second, the receiver generation difference means that identical ground truth can appear differently on the two platforms: a narrowband GPS L1/L2 jammer is visible on both, while an E5a-only or Galileo-only disturbance is visible only on S2C. The per-constellation layers of the dashboard exist to make exactly this distinction legible.

---

## 4. Data description: what the project has

### 4.1 AUX_GNSSRD — the daily GNSS observation record

AUX_GNSSRD is the Copernicus POD auxiliary product containing the daily RINEX observation file of each Sentinel-2 platform [3][4]. It is the sole input to the interference detection and is published as open data through the Copernicus Data Space Ecosystem [1]. The salient characteristics, all verified on real products during this project, are the following.

**Format and content.** Each product delivers a RINEX 3.03 observation file [6] containing, per epoch, the tracked satellites and their observables: pseudorange (C), carrier phase (L), signal strength C/N0 (S), and loss-of-lock indicators (LLI). The files contain no positions, no velocities, no Doppler and no navigation records — a fact that was verified directly by scanning real files (zero matches for ephemeris/clock records) and that dictated the design of the anti-spoofing layer (Section 5.7). RINEX 2 files from the historical record are also handled, using the SSI signal-strength flag in that format.

**Sampling and regularity.** The nominal sampling is 10 seconds, giving 8,640 epochs per day per platform. Real files checked during the v11.9.8 diagnostics were complete and uniform: 8,640/8,640 epochs at 10,000 s spacing, with timestamp jitter of the order of ±0.24 ms on S2C — the reason the gap detector applies a 500 ms tolerance (Section 5.6). This regularity is itself a data-quality asset: any deviation from the 10-second lattice is either a genuine outage or an event worth reporting, and nothing else.

**Typical sizes.** The compressed daily products are small: a Sentinel-2B daily file observed on CDSE measured 2,932,183 bytes, and a Sentinel-2C daily file approximately 5.3 MB, consistent with the larger per-epoch content of a two-constellation receiver.

**Naming convention and metadata.** Product names follow the AUX naming scheme and embed three distinct pieces of time information: the POD creation timestamp, and the validity window introduced by the `_V` marker. A real example, used throughout development: `S2B_OPER_AUX_GNSSRD_POD__20261017T120328_V20261010T235946_20261011T235936` (a 2.9 MB .TGZ). The creation timestamp (17 October 2026, 12:03:28 UTC) belongs to the publication pipeline; the validity window (10 October 23:59:46 → 11 October 23:59:36 UTC) belongs to the data. The application parses the `_V` fields and displays the validity window parsed from the product names in the local-files section and in per-file results, because conflating creation date with data date was identified as an interpretation trap for users comparing runs (v11.9.6). Appendix C tabulates the naming fields.

**Observation content per constellation.** Table 3 summarises the observation types actually found in real files and their role in the analysis.

**Table 3.** AUX_GNSSRD observation content and its role in Sentinel Jam. Observation types are the RINEX 3.03 per-signal codes [6]; the role column states where each type is used in the pipeline (Section 5).

| Constellation / signal | Observation types in real files | Role in Sentinel Jam |
| --- | --- | --- |
| GPS L1 C/A | C1C, L1C, S1C (+ D1C present in files, not used) | Primary C/N0 channel for absolute rule (S1C), relative baseline, cross-constellation test; C1C/L1C feed the kinematic metrics M1/M2 |
| GPS L2 | C2W/L2W-family, S2W | Secondary C/N0 channel for the absolute rule (L2 threshold 20 dB-Hz) |
| Galileo E1 | C1C, L1C, S1C | C/N0 for relative baseline and cross-constellation test on S2C |
| Galileo E5a | C5Q, L5Q, S5Q | Additional phase/code pair for kinematic metrics; E5a C/N0 available for analysis |
| Epoch record | receiver clock offset (last field of the `>` epoch line, optional) | Parsed since v11.10 for metric M3 (clock-step detection) |
| Loss-of-lock indicators | LLI flags on L1/L2 | Counted as degradation attributes; single-SV LLI does not trigger by design |

### 4.2 AUX_POEORB and AUX_PROQUA — the supporting products

Two further product families are browsed and retrieved alongside AUX_GNSSRD, although the core detection runs on observations alone. **AUX_POEORB** contains the precise orbit ephemerides of the platform: they anchor the geodetic reference of the ground-track model used for display, and they document the orbit geometry (sun-synchronous, 10-day repeat) that underlies the persistence reasoning. **AUX_PROQUA** contains the platform precise attitude quaternions, used by POD processing to relate the antenna frame to the orbital frame. Within the current version of Sentinel Jam these two families serve mainly as catalogue context and completeness indicators in the spacecraft panels (the dashboard shows, per platform, how many products of each family exist in the selected window), while the detection mathematics is deliberately confined to the observation file. This separation keeps the critical path minimal: a valid AUX_GNSSRD product is sufficient for a full analysis, including the kinematic layer.

### 4.3 Access path: catalogue, authentication and publication latency

The catalogue is exposed by CDSE through a public OData API [2]: querying the product lists requires no authentication, and the application browses the three auxiliary families (AUX_GNSSRD, AUX_POEORB, AUX_PROQUA) of the Sentinel-2 collection with open-data filters. Downloading the binary product (`$value`) requires authentication: an anonymous request returns HTTP 401 (error DAT-ZIP-604), verified repeatedly during development. The application therefore asks the user for CDSE credentials once, exchanges them directly against the official Copernicus identity service (Keycloak) for an OAuth2 token, keeps only the token in the browser's local storage, and discards nothing to third parties; the optional local Python proxy relays each download carrying the browser-provided token per request and stores no credentials. The password never leaves the official identity endpoint.

**Publication latency, measured.** A systematic empirical check was performed during v11.9.1 development on the 250 most recent AUX_GNSSRD releases across all Sentinel missions: every file is published 5 days 12 hours (±1 hour) after the end of its validity window, as a daily batch around 12:00 UTC. Two consequences follow. Operationally, "the last days are missing" is the normal pipeline state, not an anomaly, and the dashboard shows a standing data-availability notice to prevent misreading. Scientifically, the fixed schedule makes the archive timestamped in a perfectly predictable way, which simplifies any third-party replication of this project's comparisons.

### 4.4 Offline datasets for reproducible validation

To make the analysis reproducible without any account, the companion script `download-datasets.py` fetches three ready-made six-day windows into separate folders, each with a `manifest.txt` listing product names and validity windows (Table 4). The three windows were chosen to span materially different interference epochs and different platform generations, so that a reviewer can exercise both the detection rules and the era-dependent reading of results offline.

**Table 4.** Offline validation datasets provided by `download-datasets.py`. Each folder contains the AUX_GNSSRD products of the six-day validity window and a manifest with product names and validity intervals.

| Dataset folder | Validity window (UTC) | Platforms in the record | Historical context |
| --- | --- | --- | --- |
| `dataset-2018-10` | 2018-10-10 → 2018-10-15 | S2A + S2B (both GPS-only, 2015/2017 generation) | Baseline era before the large-scale conflict jamming of the 2020s; eastern-Mediterranean interference known from the ISS literature [9] |
| `dataset-2022-10` | 2022-10-10 → 2022-10-15 | S2A + S2B | Full-scale war in Ukraine era; Black Sea jamming widely reported [20][21][22] |
| `dataset-2026-10` | 2026-10-10 → 2026-10-15 | S2B + S2C (first multi-GNSS era) | Current era: Baltic and eastern-Europe interference reported continuously [21][22]; enables wideband and per-constellation analysis on S2C |

The script supports `--check` (catalogue verification without download), optional authentication (it obtains the OAuth2 token exactly as the main application does), and selective download (`python download-datasets.py 2022` for one dataset only). Downloaded folders can be analysed in the application's local-files mode with results bit-identical to an account run over the same files — an equivalence that is asserted by the shared code path and by the regression suite (Section 8.3).

### 4.5 Data-quality characteristics observed

Across all real files examined during development, three quality characteristics stand out and are worth recording as the empirical portrait of the input data. First, **completeness**: the daily files are gapless in nominal conditions (8,640/8,640 epochs), which makes the coverage-gap detector a genuinely selective instrument — a gap is an event, not a routine occurrence. Second, **stability of the C/N0 background**: per-satellite daily medians are stable enough that a 6–8 dB simultaneous multi-satellite dip stands far outside the ordinary variability, which is what makes the relative-baseline rule robust at a sensitivity as tight as 6 dB. Third, **predictable platform behaviour**: the only real anomalies found inside otherwise nominal files were two millisecond-scale clock-step events on S2C (Section 7.9) and known-cause tracking changes; no unexplained data corruption was ever observed in the AUX_GNSSRD chain, which speaks for the quality of the upstream Copernicus POD pipeline [3].

---

## 5. Software description: what the code actually does

### 5.1 Architecture and run modes

The application is a single self-contained HTML file (`sentinel-jam-monitor.html`) implementing catalogue browsing, downloading, RINEX parsing, detection, aggregation and visualisation in vanilla JavaScript, with no build step and no server-side component. A companion Python 3 script (`sentinel-server.py`) provides a purely optional local server on port 8123 that serves the application and relays RINEX downloads with the per-request CDSE token (header `X-CDSE-Token`); the launcher scripts (`start-rinex.bat` / `start-rinex.sh`) start it with one click. The interface is localised in five languages (English, Italian, German, French, Spanish) and every setting persists in the browser.

Three run modes share the same analysis core:

1. **Standalone HTML** (double-click the file): catalogue browsing, map, spacecraft panels and timeline work immediately; RINEX downloads from `file://` are limited by cross-origin rules, so full analysis uses the other modes.
2. **Python package** (recommended): the local server relays authenticated downloads to CDSE; the token is issued by the official identity service once and travels only between the browser and Copernicus.
3. **Local files without login**: up to 120 AUX_GNSSRD products already on disk (.TGZ from CDSE, or bare .RNX/.RNX.gz) are analysed with results bit-identical to an account run over the same files; this mode, together with the offline datasets of Section 4.4, makes the whole validation chain reproducible offline.

Any change to a detection parameter re-analyses the results already on screen without re-downloading anything, because the parsed observables are retained per file.

### 5.2 The processing pipeline

The pipeline executed per product can be described in ten stages, all implemented client-side:

1. **Catalogue query.** OData requests to CDSE [2] with collection, product-family and open-data filters; results are rendered in the catalogue list with per-family badges and byte sizes.
2. **Authenticated download.** Fetch of the product via the `$value` method; anonymous access fails with HTTP 401 as expected.
3. **Container unwrapping.** The product arrives as a ZIP; inside, the payload may be a .TGZ (tar.gz), a bare gzipped RINEX, or nested .RNX.gz. Native `DecompressionStream` provides gunzip and raw-deflate; a minimal 512-byte-ustar tar reader extracts members; a minimal read-only ZIP extractor (STORED and DEFLATE) unwraps the outer container. No third-party archive library is used.
4. **Robust candidate scoring.** Every candidate blob is scored (`rxRinexScore`): 6 = observation RINEX (carries C/N0), 3 = other RINEX (e.g. navigation), 1 = large non-RINEX blob; candidates are tried with the parser in order, and on failure the error reports name, size and head of every piece found — a deliberate diagnostics-first design that proved essential during format troubleshooting.
5. **RINEX parsing (RX-CORE).** Header (version, observation types per system, leap seconds) and epoch records are parsed into a per-epoch, per-satellite array of observables; the parser conservatively extracts what the detectors need (C/N0 on L1/L2, LLI, SSI for legacy files, and since v11.10 the pseudorange C1C, phase L1C and epoch receiver clock).
6. **Per-epoch quality evaluation** (Section 5.5): absolute, relative and cross-constellation criteria, loss-of-lock counters.
7. **Coverage-gap detection** (Section 5.6).
8. **Segmentation** (Section 5.8): flagged epochs grouped into time segments.
9. **Kinematic screening** (Section 5.7): M1/M2/M3 metrics computed from code, phase and clock.
10. **Aggregation and rendering.** Segments attributed to constellations, placed at the nominal sub-satellite position, binned into hexagons, and rendered with the timeline and per-file lists; counters and metadata lines update live.

### 5.3 Detection criteria

An epoch is flagged as degraded when any of the criteria of Table 5 holds; all thresholds are user-adjustable and persisted in the browser (`localStorage`, key `sj-det`, with input validation on reload). The criteria are complementary by construction: the absolute rule catches full-saturation jamming; the relative rule catches partial degradations that stay above the absolute floor; the cross-constellation rule certifies wideband character; the gap rule reports denial severe enough to silence the record.

**Table 5.** Detection criteria and default parameters (v11.10.0). Defaults in bold. Every parameter is exposed in the RINEX panel, shared between account and local-files runs, and re-applied live without re-downloading.

| Criterion | Rule | Default parameters | Rationale |
| --- | --- | --- | --- |
| C1 — Absolute C/N0 floor | Epoch flagged when at least **1/3** of tracked SVs (minimum 3) are simultaneously below the L1/L2 floors; two simultaneous low SVs also count when they represent ≥25% of tracked | L1 **30** dB-Hz, L2 **20** dB-Hz (editable, valid ranges 15–45 / 5–35) | Real interference saturates the whole front-end; a single low-elevation SV is normal geometry |
| C2 — Relative baseline | Per-SV daily median (≥20 samples); epoch flagged when ≥**4** SV and ≥**50%** of tracked drop ≥ **8 dB** below their own median at the same epoch | Dip **8 dB** (Balanced); presets 7 dB (Sensitive), 6 dB (Very sensitive) | Platform-independent degradation measure; makes the less jamming-exposed S2C receiver contribute |
| C3 — Cross-constellation (wideband) | ≥2 systems, each with ≥2 SV and ≥25% of its tracked SVs dipping ≥ the relative threshold simultaneously | Qualifying systems recorded per epoch (`G`, `E`, `G+E`) | A ground jammer illuminates every constellation at once; geometry, multipath and single-system effects cannot fake this signature |
| C4 — Coverage gaps | Interval between epochs > nominal step and ≥ threshold | **30 s** (≥2 missing epochs); options 1 min (≥5), 2 min (≥11) | Reports denial severe enough to interrupt the record; nominal 10 s file seam never counts (500 ms tolerance absorbs jitter) |

Additional per-epoch accounting that feeds the timeline and per-file statistics: mean L1 C/N0, minimum C/N0 of the run, count of degradations (low C/N0 or LLI), and a simultaneous multi-SV loss-of-lock counter that only counts dropouts of ≥3 SV representing ≥1/3 of previously tracked SVs (constellation turnover from rising/setting satellites is excluded by design).

### 5.4 Constellation channel filtering

For multi-GNSS platforms the whole pipeline can be restricted to a single constellation channel — GPS only or Galileo only — by filtering satellites at the epoch level *before* medians and quality evaluation (v11.9). This has a precise validation purpose: it allows a 1:1 comparison between a GPS-only receiver (S2B) and the GPS half of a dual-constellation receiver (S2C) on the same day, isolating receiver effects from constellation effects. In per-constellation detection mode, an epoch also counts when any single system independently qualifies (≥2 SV and ≥25% of its own tracked SVs below their median), which reveals single-system interference that the combined view would dilute; the v11.9.8 diagnostics demonstrated on real data that such Galileo-only events exist and are real (Section 7.4).

### 5.5 Kinematic consistency checks (anti-spoofing and receiver quality)

Because AUX_GNSSRD contains no positions, velocities or Doppler, the anti-spoofing layer (v11.10) is explicitly **not** an absolute-speed test — a LEO platform flies at ~7.5 km/s with accelerations of the order of m/s², nothing is constant. It is a **physical-consistency** check over the observable rates, built on three metrics that are all independent of the receiver clock, so that receiver-clock errors cannot fake or mask them. Table 6 lists the metrics and thresholds; "Normal" is the default level, "High sensitivity" tightens every threshold, and "Off" disables the layer.

**Table 6.** Kinematic consistency metrics and thresholds (v11.10.0). M1 requires ≥3 satellites above threshold in the same interval to trigger; M2 requires ≥4 satellites for the spread statistic; gross single-SV divergences are recorded as attributes but never trigger alone (routine cycle slips must not flood the map). Nominal statistics on real files are quoted from the feasibility study on S2B and S2C daily data (Section 7.9).

| Metric | Computed as | Normal threshold | High-sensitivity threshold | Nominal behaviour on real files |
| --- | --- | --- | --- | --- |
| M1 — Phase-vs-code rate mismatch (anti-spoofing) | Absolute difference between code-derived and phase-derived range rate per SV per interval | ≥3 SV above **0.5 m/s** | ≥3 SV above **0.2 m/s** | p95 of 0.035–0.062 m/s; spoofing ramps and counterfeit dynamics violate it massively |
| M2 — LOS-acceleration spread | Deviation of each SV's line-of-sight acceleration from the cross-satellite median | ≥3 SV beyond **25 m/s²** | ≥3 SV beyond **15 m/s²** | p99 spread 8–10 m/s², maximum observed 10.01 m/s² |
| M3 — Receiver-clock step | Absolute clock offset change between consecutive epochs | ≥ **5 ms** | ≥ **2 ms** | Millisecond-scale steps observed on S2C (−10 ms events); nominal intervals silent |
| (attribute) Gross single-SV divergence | Same statistic as M1 on one SV | recorded above 10 m/s | recorded above 5 m/s | Real cycle slips (values up to 882 km/s observed on G11) stay attributes-only |

Flagged intervals are grouped into segments carrying the worst-case metric values (`mis`, `acc`, `clk`, `gross`) and the number of epochs, and are rendered on a dedicated map layer with pink markers, popups detailing M1/M2/M3, and counters in the per-file list and analysis metadata. A segment without any clock step and without gross divergence is a pure M1/M2 anti-spoofing candidate; a clock-step-only segment is receiver quality information rather than an interference indication — the dashboard exposes this distinction to the operator.

### 5.6 Segmentation, geolocation and hexagonal aggregation

Flagged epochs are grouped into time segments of at least **3 consecutive epochs** (30 s) with a tolerance of ≤2 non-flagged epochs inside a run; each segment inherits the constellation attribution (`G`, `E`, `G+E`, or empty for unattributed/GPS-only files) and, in per-constellation mode, the qualifying-system tag. Coverage gaps are carried alongside segments in the same results structure with their missing-epoch count.

Geolocation is deliberately simple and honest: the segment's time interval is mapped to the **nominal sub-satellite position** of the platform — a parametric sun-synchronous ground-track model (inclination 98.568°, per-platform phase 0° or 180°, 10-day repeat) accurate at the degree level, not a jammer geolocation. The application documents that events are placed at the approximate overpass position; the South Pacific anomaly investigation (Section 7.6) is the empirical demonstration of both the model's role and its former defect. Segments are then binned into a custom hexagonal tiling in geographic coordinates with selectable cell size: Very fine ≈75 km, Fine ≈150 km, Normal ≈300 km (default), Coarse ≈600 km, plus an auto mode that walks the ladder to keep the drawn cells under a budget (8,000 target, 12,000 hard cap, affected cells kept first). Each cell aggregates the passes over it, with a 60 s maximum intra-pass time gap separating distinct passes.

### 5.7 Core function inventory

**Table 7.** Core functions of the RX-CORE and analysis layer (extracted from the source of v11.10.0). The table is intentionally at function granularity to support code review.

| Function / object | Role |
| --- | --- |
| `parseRinexText` (RX-CORE) | Full RINEX 3.03/2 observation parser: header, obs types per system, epochs, per-SV C/N0, LLI, SSI, and since v11.10 pseudorange, phase and epoch clock |
| `rxQuality` | Per-epoch evaluation of C1–C3 criteria, relative medians, constellation channel filter, per-system qualification tags, loss-of-lock counters, gap detection |
| `rxDetectSegs` | Grouping of flagged epochs into segments (≥3 epochs, ≤2 tolerance), per-constellation mode, system tagging; wideband pass + non-overlapping single-system pass |
| `rxKin` | Kinematic consistency metrics M1/M2/M3, threshold sets Normal/High, segment construction |
| `rxHead`, `rxRinexScore` | Blob head inspection and RINEX candidate scoring (6/3/1) |
| `gunzip`, `untarMin`, `unzipMin` | Native gzip decompression, minimal ustar reader, minimal ZIP extractor (STORED/DEFLATE) |
| `rxCandsFromBlob` | Recursive container unwrapping (gz → tar → members, or gz → single file, or bare blob) with diagnostics on failure |
| `rxNameValidity` | Parsing of the `_V<start>_<stop>` validity window from product names |
| `sjDet` (configuration), `sjDetSave`, `sjDetApply` | Persisted detection configuration (mode, channel, thresholds, gap, kinematic level) with validation on load and live re-analysis on change |
| hex-aggregation functions (`SJ_HEX_*` constants and builders) | Hex binning, pass separation, auto-fit ladder, cell trimming and heatmap/anomaly modes |
| `rxDetMeta` | Analysis metadata line builder (active thresholds, sensitivity, gap level, kinematic level) shown with results |

### 5.8 Determinism and bit-identity guarantees

Two engineering guarantees matter for validation. First, **shared code path**: account runs, local-file runs and threshold re-analyses call the same parsing and detection functions on the same bytes, which is what makes the claimed bit-identity between run modes testable rather than rhetorical. Second, **deterministic re-analysis**: parameter changes re-run detection over stored observables without re-downloading, so any difference in output is attributable to the parameter alone. Both properties are exercised by the regression suite (Section 8.3), which compares counters and segment structures across releases on fixed real files.

---

## 6. Dashboard capabilities and anomaly classification levels

### 6.1 Map modes and hexagonal display

The interactive map is the primary result surface. It operates in two modes over the same hexagonal aggregation: **Anomalies** (event collection — one cell coloured by the worst observed event class, chronological popups per event, a minimum-events filter, and a top-N trim with an explicit notice when the display is trimmed) and **Heatmap** (statistical view — the share of affected passes per cell on a fixed eight-colour ramp, with a quantile filter). Hexagon size is selectable between Very fine (~75 km), Fine (~150 km), Normal (~300 km, default), Coarse (~600 km) and an auto mode that fits the ladder to the window; fill opacity is user-configurable (default 0.62). Four base maps are available (Dark, Streets, Satellite, Terrain), retrieved from public Esri and OpenTopoMap tile services.

### 6.2 Layers

Results are distributed over toggleable layers with live counters: detections per constellation (GPS-only, Galileo-only, wideband), coverage gaps, speed/kinematic alerts (pink markers with pulse), indicative RFI zones (Section 6.5), reference ground tracks, and live spacecraft positions. The constellation separation is not cosmetic: it encodes the physical distinction between single-system disturbances (visible only to GPS or only to Galileo) and wideband sources (visible to both simultaneously), and it is the display counterpart of the C3 criterion.

### 6.3 Timeline and per-file results

The timeline chart draws one point per 10-second epoch across all analysed products — mean number of tracked satellites, mean C/N0, and degraded flags — with toggleable lines and an expandable explorer featuring a measurement crosshair. The per-file results list carries detection counters per product (GPS, Galileo, wideband, gaps, kinematic alerts), a chronological ordering (oldest → newest, v11.9.6), the validity window parsed from the product name and from RINEX content, and clickable entries that fly the map to the corresponding object. A run-statistics line reports epochs analysed, minimum epoch C/N0 and unreadable files, and the analysis metadata line records the active detection settings (sensitivity preset, mode, constellation channel, absolute thresholds, gap level, kinematic level) so that every screenshot is self-describing.

### 6.4 Persistence and shareability

Every parameter persists in the browser, and the analysis window is reproducible through shareable URL parameters (`?from=YYYY-MM-DD&to=YYYY-MM-DD`, validated and ignored if malformed) that pre-fill the RINEX panel — a deliberate support for result exchange and for the reproduction of validation comparisons by third parties.

### 6.5 Anomaly classification scheme

The dashboard does not reduce results to a single scalar severity; instead it classifies every reported object along four orthogonal axes — *what degraded* (constellation attribution), *how the record was disturbed* (degradation vs gap vs kinematic), *where it happened* (geographic context tags), and *under which parameter regime* (levels embedded in the metadata line). Table 8 consolidates the classification scheme as implemented in v11.10.0.

**Table 8.** Anomaly classification levels implemented in the dashboard. The axes are orthogonal: a single segment carries one class per axis simultaneously.

| Axis | Levels | Meaning and display |
| --- | --- | --- |
| Constellation attribution | GPS only / Galileo only / Wideband (G+E) / Unattributed | Which systems degraded simultaneously; wideband is the high-confidence interference signature; per-constellation events go to their own layers |
| Record disturbance | Degradation segment / Coverage gap / Kinematic alert | Multi-SV C/N0 drops; missing epochs (30 s/1 min/2 min threshold); M1/M2/M3 physical-consistency flags |
| Sensitivity regime | Balanced 8 dB / Sensitive 7 dB / Very sensitive 6 dB; Absolute thresholds L1 30 / L2 20 dB-Hz; Gap 30 s / 1 min / 2 min; Kinematic Normal / High / Off | Active parameter set recorded in the metadata line of every result |
| Geographic context | Inside indicative RFI zone / High-latitude tag / Open-ocean / Named popup (place, time, duration, dip depth) | Supports the responsible-reading guidance: in-zone events consistent with jamming; high-latitude and open-ocean events usually natural |

Two display-level filters modulate what the operator sees without changing the underlying classification: the minimum-events filter in Anomalies mode (cells with fewer events are hidden on demand) and the heatmap quantile filter (top quantile of affected-pass share). Both filters are explicit and reversible, so the classification itself is never silently altered.

### 6.6 Indicative RFI zones

The application embeds nine indicative RFI zones derived from public reporting, drawn as translucent circles on a dedicated layer. They are reference geography for interpretation, not detections: their radius and opacity encode confidence of the *zone*, not of any event. Table 9 lists them as implemented.

**Table 9.** Indicative RFI zones embedded in v11.10.0 (centre latitude/longitude, radius, layer opacity).

| Zone | Centre (lat, lon) | Radius | Opacity |
| --- | --- | --- | --- |
| Black Sea / southern Ukraine | 45.8, 33.8 | 340 km | 0.55 |
| Israel / Gaza | 31.5, 34.6 | 200 km | 0.55 |
| Lebanon | 33.9, 35.8 | 160 km | 0.45 |
| Red Sea / Yemen | 14.8, 43.4 | 300 km | 0.50 |
| Baltic / Kaliningrad | 55.1, 21.2 | 240 km | 0.40 |
| Strait of Hormuz | 26.6, 56.3 | 200 km | 0.35 |
| Sudan | 15.6, 32.6 | 200 km | 0.30 |
| Korean Peninsula | 38.3, 127.2 | 160 km | 0.25 |
| Strait of Taiwan | 24.3, 119.4 | 160 km | 0.20 |

---

## 7. Validation methodology and results

### 7.1 Strategy

The validation strategy follows from the impossibility of classical ground-truth validation (Section 1.3): **triangulate**. Ten independent vectors were executed across the development of the system, each answering a different question: is the data record what it claims to be (V1); does the parser extract physically possible values (V2); do the thresholds separate signal from geometry noise on real data (V3); do known jamming regions produce detections (V4); does a suspicious geophysical result survive scrutiny (V5); do two different receivers observing the same day agree (V6); does an independent reimplementation of the pipeline reproduce the application's output (V7); do synthetic faults trigger the designed responses and nothing else (V8); do real receiver anomalies surface as designed (V9); and is the publication pipeline understood well enough to rule out false "missing data" alarms (V10). Sections 7.2–7.11 report each vector with its evidence and verdict; Table 11 (Section 7.12) consolidates the matrix.

### 7.2 V1 — Internal consistency of the data record

**Question.** Are the AUX_GNSSRD files complete, regular and self-describing, as the open-data documentation claims?

**Method.** Direct examination of real daily files (S2B validity 10→11 and 11→12 October 2026; S2C validity 11→12 October 2026): epoch counts, inter-epoch intervals, observation-type inventory, receiver-clock field presence; plus the 250-release publication-schedule study (Section 7.10).

**Result.** All three files were complete and uniform: 8,640/8,640 epochs at 10,000 s spacing, timestamp jitter within ±0.24 ms (S2C), no navigation records, expected observation sets per constellation, and a fixed publication cadence (5 d 12 h ± 1 h after validity end, daily batch ~12:00 UTC). The nominal inter-file seam of 10 s was confirmed to sit below every gap threshold.

**Verdict.** **Confirmed** — the record is complete, regular and self-describing; gap detections on this archive are meaningful events by construction.

### 7.3 V2 — Physical plausibility of the parsed observables

**Question.** Does the parser extract values that a healthy spaceborne receiver could actually produce?

**Method.** A defect was caught not by a test failing but by a physical argument: cross-checking user-reported on-screen C/N0 values against receiver physics. A user screenshot of an S2C timeline showed the mean C/N0 stuck near 13 dB-Hz with no detections in 24 h; a mathematical deduction over the detection rule (with 15–20 tracked SV per epoch, flagging requires 4–7 simultaneous dips; zero flags in 24 h while *all* parsed values sit at ~13 dB-Hz implies `lowSv = cnt`, which forces `cnt ≤ 4`) proved that the parser was extracting valid C/N0 for at most ~4 SV out of 15–20 tracked, and that those few values were ~13 dB-Hz — physically impossible for a healthy receiver whose true median on comparable geometry is 47.3 dB-Hz (S2B).

**Result.** The root cause was a parser defect specific to that platform's file flavour; it was fixed, after which S2C files parsed the full ~17 SV with C/N0 in the expected 40s dB-Hz range.

**Verdict.** **Confirmed as a caught defect** — the physical-plausibility discipline converted a silent data-corruption bug into a diagnosed, fixed and regression-guarded failure. This vector also produced the project's reference health figure (median L1 C/N0 ≈ 47.3 dB-Hz on S2B), reused in later thresholds.

### 7.4 V3 — Calibration of detection thresholds on real data

**Question.** Do the multi-SV rules separate interference-like correlated drops from ordinary single-satellite geometry noise?

**Method.** Histogram-then-geography calibration on a real S2C daily file (~15–20 tracked SV per epoch). The original v11.0-era rule (≥1/3 of the constellation below threshold) was applied to the file: 1,407 epochs had exactly one low SV (geometry noise), while the strict 1/3 rule collapsed 123 candidate epochs into 9 segments — cutting real events. The rule was therefore extended: an epoch flags when ≥2 SV are simultaneously low *and* they represent ≥25% of tracked SVs.

**Result.** On the calibration file the extended rule flagged 430 epochs (5.0% of the day) grouped into 23 segments, whose sub-satellite positions fell over a geographically credible set: the Belarus/Baltic corridor, Chechnya, Kazakhstan, Pakistan and the Volga region — a mix of known tension areas and plausible regional interference geography, while the 1,407 single-SV epochs remained excluded. The `lowCn0` rule shipped as `(lowSv/svs ≥ 1/3) || (lowSv ≥ 2 && lowSv·4 ≥ svs)`, and the v11.6 relative baseline (Section 5.3) later complemented it with a per-satellite median criterion so that the less exposed S2C receiver also contributes.

**Verdict.** **Confirmed** — the multi-SV requirement rejects geometry noise (1,407 single-SV epochs) while preserving geographically credible events (23 segments); thresholds were calibrated on data, not on intuition.

### 7.5 V4 — Positive controls: known jamming regions

**Question.** Do detections cluster over regions where public reporting consistently documents GNSS interference?

**Method.** Full-day analysis of S2B validity 12 October 2026 (positions at the platform's own phase, considered reliable) with composition of every segment by place, time and plausibility; supplemented by the v11.3 geography calibration (Section 7.4) and by the standing agreement of in-zone detections with the nine indicative RFI zones (Table 9), which themselves mirror public reporting [20][21][22].

**Result.** The S2B day decomposed into two cleanly interpretable families, tabulated in Table 10: a small set of segments consistent with known RFI geography (Guam area; NW Russia/Moscow corridor, whose position falls 977 km from the Baltic RFI zone; Caspian/Iran; southern Urals), and a set of segments consistent with natural ionospheric scintillation (three arctic episodes, one antarctic episode, and four equatorial plasma-bubble episodes over the Pacific and Brazil in the post-sunset sector). The equatorial episodes occur exactly in the climatological bubble sectors and hours described in the scintillation literature [17][18].

**Table 10.** Composition of S2B detections, validity day 12 October 2026. Times are UTC; classification follows the reading guidance of the documentation (Section 2.2). This single day illustrates the intended output of the system: a mixture that separates by geography and local time into interference-consistent and natural-consistent families.

| Local epoch (UTC) | Position class | Interpretation | Family |
| --- | --- | --- | --- |
| 13:42 | Guam area | Regional interference environment (military/EA activity) | RFI-consistent |
| 20:17 | Caspian / Iran | Known interference region | RFI-consistent |
| 20:09 | Southern Urals | Regional interference / contestable area | RFI-consistent |
| 21:48 | NW Russia / Moscow corridor | 977 km from Baltic RFI zone; consistent corridor | RFI-consistent |
| 18:20, 20:06, 20:08 | Arctic latitudes | High-latitude scintillation climatology | Natural-consistent |
| 02:30 | Antarctic sector | High-latitude scintillation climatology | Natural-consistent |
| 02:02, 07:01, 08:45, 10:23 | Equatorial Pacific / Brazil | Post-sunset equatorial plasma bubbles [17][18] | Natural-consistent |

**Verdict.** **Confirmed (positive controls hit; natural families separate cleanly)** — detections over documented jamming regions appear, and the natural confounders produce recognisable, separately-clusterable families rather than contaminating the RFI-consistent ones.

### 7.6 V5 — The South Pacific anomaly: anatomy of a caught geolocation error

**Question (as raised by the operator).** S2C detections appeared clustered over the Southern Ocean / South Pacific — an area with no known interference and no plausible natural concentration — while the same days' S2B map showed events elsewhere. Was this a real, unreported phenomenon, or an artefact?

**Method.** Hypothesis testing by cross-satellite consistency. If the underlying events were real, the two platforms observing on the same days should show *relative* segments on S2C aligned — in relative terms — with *absolute* segments on S2B, whatever the nominal-position model says. The pipeline was replicated in an independent Python script (`geo_v116.py`) and 93 epoch-level cross-coupled pairs from same-day events were fitted against the ground-track phase parameter.

**Result.** The best-fit phase converged to ≈0 (340°, within estimation tolerance) — the Sentinel-2A slot — while the application rendered S2C with 0.55π. The median residual with the correct phase was ≈2,400 km, i.e. the typical *adjacent-track separation* of the 10-day-repeat constellation (segments on co-observed zones — South Pacific −21°/−23°, Urals, NW Russia, Black Sea — fell 180–590 km from their S2B counterparts at segment mid-points); with the old phase the residual was ≈13,800 km, i.e. ≈1/3 of an orbit. Root cause: Sentinel-2C had inherited Sentinel-2A's orbital slot after commissioning, but the nominal ground-track model still carried a placeholder phase; every S2C detection was therefore displaced by up to ~14,000 km, plotting events over the Southern Ocean while they occurred elsewhere. The correction shipped in v11.7 and was validated directly with the application's own sub-point function (`verify_v117_positions.js`).

**Verdict.** **Confirmed as a caught defect — and the most instructive validation event of the project.** The South Pacific anomaly was the map, not the ionosphere. Three durable lessons are recorded: (i) an unexplained geophysical cluster is a legitimate defect alarm and was treated as such; (ii) cross-platform relative agreement is a powerful, ground-truth-free validation instrument; (iii) nominal-position models must be re-audited whenever a platform changes slot. The corrected model now agrees across platforms within adjacent-track separation.

### 7.7 V6 — Cross-platform consistency

**Question.** Do two different receivers observing the same events produce consistent detections?

**Method.** Same-day cross-matching of S2B (GPS-only, 2017 generation) and S2C (GPS+Galileo, 2024 generation) segments after the v11.7 correction, at epoch level (93 cross-coupled pairs) and at segment-mid level over co-observed zones.

**Result.** Epoch-level residuals collapse from ~13,800 km (wrong phase) to ~2,400 km median (correct phase, adjacent-track separation), and co-observed zone segments agree within 180–590 km; the two families used throughout V4 (RFI-consistent and natural-consistent) reproduce on both platforms. The receiver-generation difference remains visible exactly as designed: the absolute rule fires more readily on S2B (fewer SV), while the relative-baseline rule keeps the platforms comparable.

**Verdict.** **Consistent** — cross-platform agreement within geometry bounds after correction; the comparison is now a standing acceptance check for future releases.

### 7.8 V7 — Independent reimplementation cross-check

**Question.** Does an independent implementation of the documented pipeline reproduce the application's results?

**Method.** The parsing and detection chain was reimplemented in Python (`geo_v116.py`: parser + `rxQuality` equivalent + segmentation + sub-point) and executed on the same real files; a companion read-only script (`verify_l1_map_effect.js`) extracts the RX-CORE from the shipped HTML and compares, for every L1/L2 setting, exactly the data the map draws (segments by kind/system/interval and gaps).

**Result.** The Python replication reproduced the segment inventory and positions of the application on the calibration files; the map-effect verifier confirmed that every rendered object traces back to a computed segment/gap of the shared core. The v11.7 phase fit itself is a by-product of this vector.

**Verdict.** **Confirmed** — pipeline outputs are reproducible outside the application's own code path.

### 7.9 V8/V9 — Synthetic fault injection and real receiver anomalies

**Question.** Do the detectors respond to injected faults with the designed behaviour — and to nothing else?

**Method and results (synthetic, V8).**

- *Coverage gaps.* A 12-epoch (2 min) hole injected into a real file yields exactly one gap with `miss = 12`; real gapless files yield zero gaps at every threshold level (30 s/1 min/2 min); a 20 s hole never counts, 30 s counts only at the 30 s level, and the 10 s daily seam never counts — all asserted by `test_v1199_gap.js`.
- *Spoofing ramp.* A +2 m/s phase ramp on 3 SV for 12 epochs: M1 flags 12/12 intervals at the Normal threshold with **zero false positives** on the pristine remainder; a subtler 0.3 m/s ramp is silent at Normal and flagged at High sensitivity (exactly the designed sensitivity ladder).
- *Clock step.* An injected +10 ms receiver-clock step yields one M3 segment; segments without clock steps are M1/M2 candidates only.
- *M2 dynamics.* A +300 m/s LOS acceleration anomaly on 3 SV yields isolated M2 segments (2 segments in the regression fixture); single-SV gross divergences (values as extreme as real cycle slips) never trigger alone.

**Method and results (natural test vectors, V9).** Two real receiver-clock events exist in the S2C file of 11→12 October 2026: −9.999 ms steps at ≈04:14 UTC and ≈19:54 UTC, each accompanied by a phase-vs-code divergence of ~2.99×10⁶ m on E16/E33/G13/E26 — consistent with a receiver clock-resynchronisation. The kinematic layer detects *exactly* these two segments on the real file (04:14:12 and 19:53:42 UTC segment starts, `mis = 17/16`, clock+gross attributes), while the S2B file of the same window stays pristine except for a single borderline 0.53 m/s interval below every threshold. The nominal statistics measured across both platforms (LOS rates ±8.4 km/s; LOS acceleration p95 ≈ 12–13 m/s²; phase-vs-code mismatch p95 0.035–0.062 m/s; cross-satellite acceleration spread p99 8.4–10 m/s²) are the empirical basis of the thresholds in Table 6.

**Verdict.** **Confirmed** — designed responses to every injected fault; no flooding from cycle slips; real anomalies surface precisely; thresholds anchored to measured nominal statistics.

### 7.10 V10 — Publication-pipeline verification

**Question.** Is "missing recent days" a pipeline artefact or an anomaly?

**Method.** Empirical audit of the 250 most recent AUX_GNSSRD releases (all Sentinel missions) at v11.9.1 time.

**Result.** Fixed schedule: publication 5 d 12 h ± 1 h after the end of the validity window, daily batch around 12:00 UTC. The catalogue anomaly module was removed as a consequence and replaced by a standing availability notice.

**Verdict.** **Confirmed** — the publication process is deterministic and documented; user-facing false alarms are prevented by design.

### 7.11 Representative end-to-end run

For orientation, a representative end-to-end run on real data (one day, S2B + S2C, sensitivity 6 dB from persisted settings) yields the counters: **GPS 97, Galileo 24, wideband 8, gaps 0, kinematic alerts 2** — the kinematic alerts being exactly the two real clock-step events of Section 7.9, and the wideband core (8 segments GPS+Galileo) coexisting with the per-constellation attributions after v11.9.8. These are the same counters asserted unchanged across the last three releases by the regression suite, making them de-facto golden values for the shipped version.

### 7.12 Validation matrix

**Table 11.** Consolidated validation matrix. Verdicts follow the terminology of Section 1.3.

| # | Vector | Question | Evidence | Verdict |
| --- | --- | --- | --- | --- |
| V1 | Data-record consistency | Is the input complete and regular? | 3 real files: 8,640/8,640 epochs; ±0.24 ms jitter; obs-type inventory; 250-release schedule study | Confirmed |
| V2 | Physical plausibility | Can the parser emit impossible values? | 13 dB-Hz / ≤4 SV deduction on S2C → parser defect caught, fixed, regression-guarded | Confirmed (defect caught) |
| V3 | Threshold calibration | Do rules separate signal from geometry noise? | 1,407 single-SV epochs rejected; 430 epochs → 23 segments over credible geography | Confirmed |
| V4 | Positive controls | Do known jamming regions hit? | S2B 12 Oct 2026 composition (Table 10); RFI-zone agreement; era datasets 2018/2022/2026 | Confirmed |
| V5 | South Pacific anomaly | Was the unexplained cluster real? | 93 cross-coupled pairs; phase fit ≈0 vs 0.55π; residual 2,400 vs 13,800 km; fix v11.7 | Confirmed (defect caught) |
| V6 | Cross-platform consistency | Do S2B and S2C agree? | Post-v11.7 same-day cross-match within adjacent-track separation (180–590 km on co-observed zones) | Consistent |
| V7 | Reimplementation | Does an independent pipeline reproduce outputs? | `geo_v116.py` replication; map-effect verifier | Confirmed |
| V8 | Synthetic injection | Designed responses and nothing else? | Gap/spoof/clock/M2 injections (Section 7.9) with zero false positives at Normal | Confirmed |
| V9 | Natural test vectors | Do real receiver anomalies surface? | 2 real −10 ms S2C clock steps detected exactly; S2B pristine | Confirmed |
| V10 | Publication pipeline | Are missing recent days normal? | 250-release audit: 5 d 12 h ± 1 h batch | Confirmed |

### 7.13 Threats to validity and honesty about the remaining gap

The matrix above must be read against its own limits. **Selection bias**: real-file validations concentrate on the October 2026 window (and the three offline datasets); behaviour in other seasons, solar-cycle phases and ionospheric regimes is argued from climatology, not yet measured systematically. **No formal detection statistics**: the probability of detection and the false-alarm rate per operating point (8/7/6 dB presets) have not been measured over large blind sets; the counters quoted here are case evidence, not ROC curves. **External correlations are episodic**: comparisons with GPSJam and the Stanford/FR24/Wingbits maps have been made informally and agree on the major regions, but no systematic day-by-day correlation study has been executed and archived yet. **Geolocation is nominal**: the sub-satellite placement is a screening position, and the V5/V6 evidence bounds the *model* error, not the jammer position error. **Single-platform per pass**: most epochs see events from one satellite only, so corroboration rests on the 10-day repeat rather than on simultaneous multi-platform observation. These limitations are carried into Section 9 and into the future-work list, where each has a concrete remedy.

---

## 8. Development history and regression evidence

### 8.1 Development model

The system evolved through numbered patch releases (v11.x lineage) applied as exact-match replacement scripts over a backed-up prior version, each release accompanied by: a syntax check of the embedded JavaScript, a regression run over fixed real files, an end-to-end browser test on a local server, and (when the package changed) a repackaging verification comparing MD5 checksums of disk and zip copies and enforcing CRLF line endings in the Windows launcher. The version markers visible in the source comments reconstruct the lineage; no version history is exposed in the user interface, which is intentional — the published artefacts describe only the current behaviour, while this section preserves the development narrative for validation purposes.

### 8.2 Version milestones

**Table 12.** Milestones of the v11 lineage with their validation evidence, reconstructed from in-source comments and the development log. Intermediate maintenance versions (v11.1, v11.2, v11.4, v11.5) carried file-handling, interface and localisation refinements now superseded; they are summarised in the final row.

| Version | Milestone | Validation evidence produced |
| --- | --- | --- |
| v11.0 | Robust product extraction (ZIP → TGZ/gz/RINEX), original epoch detection rule (≥1/3 constellation below absolute thresholds) | First detections on real Copernicus files; candidate-scoring diagnostics introduced |
| v11.3 | Threshold recalibration on real data | 1,407 single-SV epochs rejected vs 430 flagged epochs → 23 segments over credible geography (Belarus/Baltic corridor, Chechnya, Kazakhstan, Pakistan, Volga); regression `test_v113_real` (Section 7.4) |
| v11.3–v11.5 era | Platform parser fix after the 13 dB-Hz physical-plausibility deduction | Parser defect caught by receiver physics; S2C files then parse ~17 SV at healthy C/N0 (Section 7.3) |
| v11.6 | Relative-baseline criterion (per-SV daily median, 4+ SV, ≥50%, 8/7/6 dB presets) | Makes the less jamming-exposed S2C receiver contribute; platform-comparable detections |
| v11.7 | Orbital-phase correction (S2C inherits S2A slot; phase 0.55π → 0) | 93 cross-coupled pairs; residual ≈2,400 km vs ≈13,800 km; South Pacific anomaly eliminated (Section 7.6) |
| v11.8 | Cross-constellation wideband test + relative-dip selector | Wideband certification (≥2 systems, ≥2 SV, ≥25% each) on multi-GNSS files |
| v11.9 | Detection configuration object (`sj-det`): mode wideband/per-constellation, constellation channel G/E/both, editable absolute thresholds, per-epoch system tags, separate map layers | Channel filter enables 1:1 S2B ↔ S2C(GPS) comparison; persisted, validated on reload |
| v11.9.1 | Removal of the catalogue publication-gap module | 250-release audit: fixed 5 d 12 h ± 1 h batch; standing availability notice instead of false anomalies (Section 7.10) |
| v11.9.4 | Manual local-files mode (no login) | Offline analysis path for the validation datasets; replaces demo fixtures with real products |
| v11.9.5 | Shared detection parameters for manual and account runs; anti-blocking yields | Uniformity of results across run modes; identical counters on same bytes |
| v11.9.6 | Validity-window parsing from product names (`_V` fields), chronological result ordering, per-file data-date line | Creation-date vs data-date disambiguation; reproducible ordering |
| v11.9.7 | Days-based file selector (max 60 days ≈ 120 files); shareable `?from&to` URL prefill | Reproducible windows for third-party replication |
| v11.9.8 | Per-constellation attribution fix (wideband mode no longer hides single-system events; GPS-only files tag correctly); latent gap-click index fix | Diagnostics on 3 real files: 42/51 E-only runs far from wideband segments (Galileo-only events real); synthetic 12-epoch hole → exactly 1 gap, miss=12; golden counters GPS 97 / Gal 24 / Wide 8 / Gaps 0 |
| v11.9.9 | User-selectable coverage-gap threshold (30 s / 1 min / 2 min) | Gap semantics table tested (20 s never; 30 s only at 30; 60 s at 30+60; 130 s at all); real gapless files unchanged |
| v11.10.0 | Kinematic consistency layer (M1 anti-spoofing, M2 dynamics, M3 clock steps) with Normal/High/Off, dedicated layer, metadata counters | Feasibility statistics on real S2B/S2C; injection suite all-PASS; exactly 2 real S2C clock-step segments detected; zero impact on existing counters (Section 7.9) |

### 8.3 Regression suite

Every release since v11.8 is gated by a fixed regression battery executed against real files committed for that purpose, plus a browser-level end-to-end test. The battery is designed so that a change in any detector either preserves the golden counters or deliberately changes them with an updated assertion — silent drift is impossible by construction.

**Table 13.** Regression test suite as of v11.10.0. All entries PASS on the shipped release; scopes are stated to allow independent re-execution (Appendix D).

| Test | Scope | Key assertions |
| --- | --- | --- |
| `test_v113_real` | v11.3 calibration file | 430 flagged epochs; 23 segments; single-SV noise excluded |
| `test_v118_real` | Relative baseline + wideband on real files | Segment inventories per sensitivity preset; wideband qualification rules |
| `test_v119_real` | RX-CORE under channel filter | RX-CORE bit-identical output; channel filtering isolates G/E subsystems |
| `test_v1198_issue12` | Per-constellation attribution | S2B segments all GPS-tagged; S2C G=14 / E=24 / X=8 ordering; per-mode 49/24 E-only; wideband core 8 G+E intact |
| `test_v1199_gap` | Gap detector semantics | 20 s never; 30 s only at 30 (miss=2); 60 s at 30+60 (miss=5); 130 s at all (miss=12); real gapless files → 0 |
| `test_v1110_kin` | Kinematic layer | Parser fields (rng/ph/clk); nominal → 0 alerts; spoof ramp → 1 segment (mis ≥3); clock step +10 ms → 1 segment; 4-SV gross → flagged; single-SV gross → 0 segments; M2 +300 m/s ×3 SV → 2 isolated segments; Off → empty; 0.3 m/s silent at Normal, flagged at High; real S2C exactly 2 segments (04:14:12, 19:53:42 UTC), S2B 0 |
| `e2e_v1110.sh` | Browser end-to-end on local server | Version badge; zero console errors; counters GPS=97, Galileo=24, Wideband=8, Gaps=0, KIN=2; pink KIN rows and metadata line; Off/High persistence; five-language chips; footer integrity; screenshots |
| packaging checks | Distribution integrity | MD5 equality disk vs zip for every member; Windows launcher CRLF-only; `py_compile` on shipped Python scripts |

### 8.4 End-to-end verification practice

The end-to-end practice deserves a brief methodological note because it complements unit-level regression. Each E2E run boots the packaged application on a local port, loads real AUX_GNSSRD products from disk (no network), drives the actual UI (panel selectors, layer toggles, list clicks), and asserts both the DOM state and the rendered counters. Because the analysis core is the same code path as the account mode, an E2E pass plus the regression battery plus MD5 integrity of the inputs jointly guarantee that the shipped artefact reproduces the validated behaviour on the validation data. This chain — real bytes, shared code path, asserted counters — is the practical meaning of the "bit-identical" claims in Section 5.8.

---

## 9. Discussion

### 9.1 What is validated, and to what degree

Taking the ten vectors together, the current state of validation supports four claims with high confidence. (1) *Data integrity*: the AUX_GNSSRD chain behaves as documented, and the application's handling of it (containers, naming, latency) is verified empirically. (2) *Detector behaviour on real data*: thresholds separate correlated events from geometry noise; known jamming regions produce detections; natural confounders form separable families. (3) *Implementation correctness under fault*: injections and real receiver anomalies produce exactly the designed responses, with the two historical defects (parser, orbital phase) caught by the validation process itself and now regression-guarded. (4) *Reproducibility*: an independent reimplementation reproduces the outputs, and the offline datasets plus the regression suite allow any third party to re-execute the evidence chain without credentials.

What is *not* yet established is statistical performance in the instrument-validation sense: operating-characteristic curves per parameter preset, systematic day-by-day correlation against aviation-based services, and quantified false-alarm rates under adverse ionospheric conditions. These remain open, and Section 10 defines them as the next milestones; until then, the system should be read as a high-quality *screening* instrument whose individual detections require corroboration — a framing the user documentation makes explicit.

### 9.2 Comparison with independent services

**Table 14.** Sentinel Jam in the context of independent GNSS-interference monitoring services. The comparison is functional, not competitive: the sensing principles are complementary (Section 2.3), and cross-service agreement is one of the system's intended validation axes.

| Service | Sensing principle | Coverage strength | Latency | Notes for correlation with Sentinel Jam |
| --- | --- | --- | --- | --- |
| Sentinel Jam (this work) | C/N0 telemetry of POD GNSS receivers on LEO (Sentinel-2) | Global, including oceans, poles, conflict zones without aviation | ~5–6 days (publication pipeline) | Sub-satellite screening positions; per-constellation and wideband attribution; open data end-to-end |
| GPSJam [22] | GNSS accuracy degradation reported by aircraft via ADS-B | Traffic-dense regions | Daily maps | Best correlation reference over Europe/Middle East/East Asia; blind over oceans |
| Stanford WAAS-NAS dashboard [23] | Aviation GNSS integrity data (WAAS/NAS) | North-American and correlated air routes | Near-real-time | Research-grade; complements on civil-aviation integrity |
| Stanford GNSS Interference Map [24] | ADS-B/NIC-based RFI monitoring (GPS Research Lab) | Global traffic routes | Near-real-time | Academic counterpart; useful for event-level cross-matching |
| Flightradar24 GPS jamming [25] | ADS-B navigation-integrity category from global receiver network | Traffic-dense regions | Near-real-time | Broad public visibility; matches Baltic/Middle East patterns |
| Wingbits map [26] | Live ADS-B-derived jamming/spoofing | Traffic-dense regions | Live | Live layer for ongoing events |
| sat-jam.com [29] | Comparable GNSS interference monitoring from satellite GNSS observations | Global | Service-dependent | Independent comparable service used for cross-check only; not part of this project |
| Ground networks of low-cost front ends [16] | Terrestrial multi-receiver detection/localisation | Deployed areas only | Continuous | Ground truth where available; validates geometry assumptions |

The expected correlation structure follows from the physics: agreement over traffic-dense jamming regions (aviation services and Sentinel Jam both see the source), Sentinel-Jam-only detections over oceans and non-transponded areas (unique coverage), and aviation-only reports where an event is too weak, too local, or too short to cross the multi-SV rules at 10 s sampling. Both asymmetries are informative rather than contradictory, and Section 10 proposes the systematic correlation campaign needed to quantify them.

### 9.3 Known limitations and residual risks

**Table 15.** Known limitations of the system with their impact and current mitigation. The list consolidates the documented limitations and those surfaced by the validation work.

| Limitation | Impact | Mitigation in v11.10.0 |
| --- | --- | --- |
| Screening method: no jammer geolocation, no emitted-power estimate | Positions are sub-satellite approximations of the overpass, not source locations | Explicit documentation; V5/V6 bound the model error; nominal model at degree level |
| 10 s sampling | Short events can be missed or smoothed; dip depth partially averaged | Documented; gap detector quantifies what is missing at 10 s multiples |
| ~5–6 day publication latency | Not real-time; recent days appear late by design | Standing availability notice; measured fixed schedule (V10) |
| S2A receiver failed (2025-01-21) | Historical platform only; record ends there | Spacecraft panel reflects status; datasets label platform eras |
| Platform sensitivity asymmetry (S2B vs S2C) | Absolute rule fires more easily on the GPS-only receiver | Relative-baseline criterion for cross-platform comparability; channel filter for 1:1 tests |
| Ionospheric confounders (high latitudes, equatorial post-sunset) | Natural multi-SV fades mimic interference | High-latitude tagging; natural-family separability demonstrated (V4); reading guidance |
| Solar radio bursts | Hemisphere-wide C/N0 depression can create broad detections | Recognisable same-epoch hemispheric pattern in the timeline; guidance |
| Nominal ground-track positions (±1–2°) | Display-level positional uncertainty | Corrected phases (V5); optional improvement via AUX_POEORB planned (Section 10) |
| No formal ROC statistics yet | Sensitivity/specificity not quantified per preset | Case evidence only; systematic campaign defined as future work |
| Legacy RINEX 2 SSI signal-strength flags | Coarser quality indicator on the historical record | SSI mode supported with reduced threshold fidelity; documented |

---

## 10. Conclusions and future work

### 10.1 Conclusions

The project set out to identify radio anomalies from satellite observations using only open data, and the validation work performed so far supports the conclusion that this objective is met within the claimed envelope: Sentinel Jam turns the routine C/N0 telemetry of Sentinel-2 POD receivers into a global, reproducible, multi-criteria GNSS interference screen with per-constellation attribution, coverage-gap reporting and kinematic anti-spoofing screening, delivered as a single client-side artefact with offline reproducibility. The validation is *partial but substantive*: ten vectors — from physical-plausibility reasoning to cross-satellite phase fitting, from synthetic injection to real clock-step events — converge on the same picture, and the two defects that the process uncovered (parser physicality, orbital phase) were caught precisely because the validation discipline treats unexpected geophysical results as alarms rather than curiosities. The system's detections over known jamming regions are consistent with public evidence, its natural-confounder families separate cleanly, and its golden counters are regression-guarded.

### 10.2 Future work

Six extensions are identified, in priority order. (1) **Formal detection statistics**: execute the three offline-era datasets plus additional blind windows per sensitivity preset to estimate probability of detection and false-alarm rate, producing the first ROC family for this sensor class from open data. (2) **Systematic external correlation**: a day-by-day, region-bucketed correlation campaign against GPSJam and the Stanford/Flightradar24/Wingbits maps, with the correlation tables published alongside the code; the shareable URL parameters already make windows reproducible. (3) **Ionospheric screening**: integrate Kp/F10.7 and scintillation indices to tag detections under disturbed conditions automatically, sharpening the natural/interference separation that V4 demonstrated qualitatively. (4) **Exact positions from AUX_POEORB**: replace the nominal ground-track model for display (and for cross-platform matching) with precise orbit interpolation, reducing the remaining ±1–2° placement uncertainty. (5) **Fleet extension**: evaluate equivalent open POD GNSS records from other Copernicus platforms to widen the multi-platform simultaneity that V6 showed to be the strongest internal validation instrument. (6) **Provenance export**: per-run machine-readable result export (segments, gaps, kinematic alerts, parameters) so that third parties can ingest validation comparisons directly.

---

## References

[1] Copernicus Data Space Ecosystem — portal and free registration. https://dataspace.copernicus.eu

[2] Copernicus Data Space Ecosystem — OData API documentation. https://documentation.dataspace.copernicus.eu/APIs/OData.html

[3] SentiWiki, Copernicus — Precise Orbit Determination (Copernicus POD Service, CPOD). https://sentiwiki.copernicus.eu/web/precise-orbit-determination

[4] Copernicus POD Product Handbook (definitions of the AUX_GNSSRD, AUX_POEORB and AUX_PROQUA products). https://sentinels.copernicus.eu/documents/d/sentinel/copernicus-pod-product-handbook

[5] ESA, Sentinel Online — Sentinel-2 mission. https://sentinels.copernicus.eu/web/sentinel/missions/sentinel-2

[6] IGS/RTCM RINEX Working Group, "RINEX — The Receiver Independent Exchange Format, Version 3.03", International GNSS Service, 2015. https://files.igs.org/pub/formats/rinex/rinex303.pdf

[7] Beyond Gravity (RUAG Space), "PODRIX GNSS Receiver — spaceborne satellite receiver", product datasheet. https://www.beyondgravity.com/sites/default/files/media_document/2024-04/BG_PODRIX_GNSS_Receiver%20spaceborne_Satellite_RX_V1.0.pdf

[8] A. Conrad et al., "Sentinel-6 Michael Freilich precise orbit determination", *Journal of Geodesy*, vol. 98, 2024. https://doi.org/10.1007/s00190-024-01842-5

[9] M. J. Murrian, L. Narula, P. A. Iannucci, S. Budzien, B. W. O'Hanlon, M. L. Psiaki, T. E. Humphreys, "First results from three years of GNSS interference monitoring from low Earth orbit", *NAVIGATION: Journal of the Institute of Navigation*, vol. 68, no. 4, pp. 673–683, 2021. https://doi.org/10.1002/navi.449 — preprint: arXiv:2009.04093

[10] O. Isoz, D. M. Akos, T. Lindgren, A.-C. Sun, C.-C. Jan, "Assessment of GPS L1 GNSS interference events utilizing a spaceborne GPS receiver", *NAVIGATION: Journal of the Institute of Navigation*, vol. 61, no. 2, pp. 147–161, 2014.

[11] M. L. Psiaki, T. E. Humphreys, "GNSS Spoofing and Detection", *Proceedings of the IEEE*, vol. 104, no. 6, pp. 1258–1270, 2016. https://doi.org/10.1109/JPROC.2016.2528658

[12] T. E. Humphreys, B. M. Ledvina, M. L. Psiaki, B. W. O'Hanlon, P. M. Kintner Jr., "Assessing the spoofing threat of GPS", *Proceedings of ION GNSS 2008*, Savannah, GA, 2008.

[13] A. G. Dempster, E. Cetin, "Interference localization for satellite navigation systems", *Proceedings of the IEEE*, vol. 104, no. 6, pp. 1318–1326, 2016. https://doi.org/10.1109/JPROC.2016.2530814

[14] L. Cao, "A Review of Space-Based GNSS Interference Monitoring", *Electronics*, vol. 15, no. 19, art. 4357, 2026. https://www.mdpi.com/2079-9292/15/19/4357

[15] G. Patil et al., "Detecting and localizing space-based RFI", *Proc. ION International Technical Meeting (ITM)*, 2024. https://web.stanford.edu/group/scpnt/gpslab/pubs/papers/Patil_ION_ITM_2024_Detecting_Space_RFI.pdf

[16] J. Lindström, D. M. Akos, O. Isoz, M. Junered, "GNSS interference detection and localization using a network of low-cost front-end modules", *Proc. ION GNSS 2007*, Fort Worth, TX, 2007. https://www.diva-portal.org/smash/get/diva2:1011883/FULLTEXT01.pdf

[17] P. M. Kintner Jr., T. E. Humphreys, J. Hinks, "GNSS and ionospheric scintillation — How to survive the next solar maximum", *Inside GNSS*, vol. 4, no. 4, pp. 22–33, 2009.

[18] J. Vilà-Valls et al., "Survey on signal processing for GNSS under ionospheric scintillation: detection, robust estimation and mitigation", *NAVIGATION: Journal of the Institute of Navigation*, vol. 67, no. 3, 2020. https://navi.ion.org/content/67/3/511

[19] A. P. Cerruti, P. M. Kintner Jr., D. E. Gary, A. J. Mannucci, R. F. Meyer, P. Doherty, A. J. Coster, "Effect of intense December 2006 solar radio bursts on GPS receivers", *Space Weather*, vol. 6, S10D07, 2008. https://doi.org/10.1029/2007SW000375

[20] EASA, "EASA updates Safety Information Bulletin on GNSS interference". https://www.easa.europa.eu/en/newsroom-and-events/news/easa-updates-safety-information-bulletin-gnss-interference

[21] EASA / EUROCONTROL, GNSS RFI workshop material (slides). https://www.eurocontrol.int/system/files/2026-07/gnss-rfi-workshop_slides_final.pdf

[22] GPSJam — daily GNSS interference maps derived from ADS-B data (J. Wiseman). https://gpsjam.org

[23] Stanford University, GNSS Interference Detection using ADS-B (WAAS/NAS monitoring dashboard). https://waas-nas.stanford.edu/

[24] Stanford University GPS Research Lab, GNSS Interference Map (ADS-B/NIC-based RFI monitoring and research). https://rfi.stanford.edu/

[25] Flightradar24, GPS jamming map derived from ADS-B navigation-integrity data. https://www.flightradar24.com/data/gps-jamming

[26] Wingbits, live GPS jamming and spoofing map. https://wingbits.com/gps-jamming?layers=both

[27] GNSS-RO Multi-Portal — GNSS Radio Occultation &amp; Scatterometer, real-time multi-source GNSS-RO anomaly and interference dashboard (same author). https://github.com/GiamMaBasedResearchers/GNSS-RO-Multi-Portal--GNSS-Radio-Occultation-Scatterometer

[28] ADS-B Sky Tracker Pro — aviation tracking with GNSS jamming detection and SSR analysis (same author). https://github.com/GiamMaBasedResearchers/ADS-B-Sky-Tracker-Pro

[29] sat-jam.com — independent comparable GNSS interference monitoring service (cross-check only, not part of this project). https://sat-jam.com

[30] Author profile and updates — GiamMa-based researchers SDR R&amp;D IoT. https://x.com/giammaiot2

---

## Appendix A. Acronyms and abbreviations

| Acronym | Meaning |
| --- | --- |
| ADS-B | Automatic Dependent Surveillance–Broadcast |
| AUX | Auxiliary (Copernicus product family prefix) |
| C/N0 | Carrier-to-noise density ratio (dB-Hz) |
| CDSE | Copernicus Data Space Ecosystem |
| COSPAR / NORAD | International satellite catalogue identifiers |
| CPOD | Copernicus Precise Orbit Determination Service |
| E2E | End-to-end (test) |
| GNSS | Global Navigation Satellite System |
| LEO | Low Earth Orbit |
| LLI | Loss-of-Lock Indicator (RINEX) |
| LOS | Line of sight |
| MSI | MultiSpectral Instrument (Sentinel-2 payload) |
| OData | Open Data Protocol (CDSE catalogue API) |
| OAuth2 | Open standard for access delegation (token authentication) |
| POD | Precise Orbit Determination |
| PODRIX | POD receiver product family (RUAG Space / Beyond Gravity) |
| RFI | Radio-Frequency Interference |
| RINEX | Receiver Independent Exchange Format |
| ROC | Receiver Operating Characteristic |
| SSI | Signal Strength Indicator (RINEX 2) |
| SV | Space Vehicle (satellite) |
| TGZ | gzipped tar archive |
| UTC | Coordinated Universal Time |

## Appendix B. Default parameter reference (v11.10.0)

**Table 16.** Complete default-parameter reference. Ranges and valid-value sets are enforced on persisted reload.

| Parameter | Default | Valid range / options |
| --- | --- | --- |
| Relative-baseline dip threshold | 8 dB | 8 / 7 / 6 dB (Balanced / Sensitive / Very sensitive) |
| Relative-baseline qualifying set | ≥4 SV and ≥50% of tracked | fixed; per-SV median requires ≥20 samples |
| Absolute thresholds | L1 30 dB-Hz; L2 20 dB-Hz | L1 15–45; L2 5–35 (editable) |
| Absolute-rule qualifying fraction | ≥1/3 of tracked SVs (min 3); or ≥2 SV when ≥25% of tracked | fixed |
| Wideband qualification | ≥2 systems, ≥2 SV each, ≥25% of own tracked | fixed |
| Detection mode | Wideband cross-GNSS | wide / per-constellation |
| Constellation channel | GPS + Galileo | both / G / E |
| Coverage-gap threshold | 30 s | 30 s / 1 min / 2 min (500 ms jitter tolerance) |
| Segment rule | ≥3 consecutive flagged epochs, ≤2 tolerated | fixed |
| Simultaneous loss-of-lock counter | ≥3 SV vanishing, ≥1/3 of previously tracked | fixed |
| Kinematic level | Normal | off / normal / high |
| Kinematic thresholds | M1 0.5 m/s; M2 25 m/s²; M3 5 ms; gross attribute 10 m/s | High: 0.2 / 15 / 2 / 5 |
| Hex size | Normal (~300 km) | 75 / 150 / 300 / 600 km / auto (target 8,000 cells, cap 12,000) |
| Hex minimum samples per cell | 12 (rescaled with cell size) | auto with size |
| Intra-pass time gap | 60 s | fixed |
| Hex opacity | 0.62 | 0.1–0.95 |
| Basemaps | 4 (Dark, Streets, Satellite, Terrain) | — |
| Local-files capacity | ≤120 files per run | — |
| Analysis window | up to 60 days per run (≈2 files/day/platform) | — |
| Interface languages | EN, IT, DE, FR, ES | — |
| Publication cadence (context) | 5 d 12 h ± 1 h after validity end, batch ~12:00 UTC | measured over 250 releases |

## Appendix C. Product naming convention and metadata

**Table 17.** AUX_GNSSRD naming fields, illustrated on a real product used during validation: `S2B_OPER_AUX_GNSSRD_POD__20261017T120328_V20261010T235946_20261011T235936` (delivered as a 2,932,183-byte .TGZ).

| Field | Example value | Meaning |
| --- | --- | --- |
| Platform | `S2B` | Sentinel-2 unit whose receiver produced the record |
| Mission class | `OPER` | Operational processing |
| Product type | `AUX_GNSSRD` | Daily GNSS RINEX observation record |
| Processing agent | `POD` | Generated by the Copernicus POD chain |
| Creation timestamp | `20261017T120328` | Publication-side timestamp (12:03:28 UTC on 17 Oct 2026) |
| Validity start | `V20261010T235946` | First epoch covered by the file (23:59:46 UTC on 10 Oct 2026) |
| Validity end | `20261011T235936` | Last epoch covered (23:59:36 UTC on 11 Oct 2026) |

The application parses the `_V<start>_<stop>` pair and displays the validity window wherever results are listed, because the creation date is normally 5–6 days after the data date and conflating the two is the most common interpretation error when comparing runs. The same convention applies to AUX_POEORB and AUX_PROQUA within the catalogue panels.

## Appendix D. Reproducibility notes

**Datasets.** `python download-datasets.py` fetches the three six-day windows (2018-10-10→15, 2022-10-10→15, 2026-10-10→15) into `dataset-<year>-10/` folders with `manifest.txt` each; `--check` verifies catalogue availability without downloading. A free CDSE account is required for download; catalogue checks are anonymous.

**Re-executing the analysis.** The local-files mode of the application accepts the downloaded .TGZ products directly (up to 120 files per run) and produces results bit-identical to an account run over the same files, because both paths execute the same parser and detectors on the same bytes. All detection parameters are persisted and visible in the analysis metadata line, and shareable URLs (`?from=...&to=...`) reproduce any window exactly.

**Regression scripts.** The validation evidence of Section 7 is re-executable with the regression battery of Table 13 (`test_v113_real`, `test_v118_real`, `test_v119_real`, `test_v1198_issue12`, `test_v1199_gap`, `test_v1110_kin`, plus the `e2e_v1110.sh` end-to-end run and the packaging integrity checks). The independent Python replication (`geo_v116.py`), the kinematic feasibility study (`eval_kinematic.py`) and the position verifier (`verify_v117_positions.js`) document the cross-implementation vectors V7–V9.

**Environment requirements.** A modern browser with native `DecompressionStream` support (for gzip and raw-deflate) is required for the container unwrapping; the optional Python helper needs Python 3 only. No other runtime dependency exists, which keeps the entire evidence chain auditable by inspection.
