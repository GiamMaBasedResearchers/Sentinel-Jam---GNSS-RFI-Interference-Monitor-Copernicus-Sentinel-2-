================================================================
 SENTINEL JAM — GNSS / RFI Monitor (Copernicus Sentinel-2)
 FILES IN THIS FOLDER
================================================================

  sentinel-jam-monitor.html ............ standalone monitor app
                                         (double-click to open; the version
                                         is shown in the app top bar)
  sentinel-server.py ................... local server + RINEX download proxy
  start-rinex.bat ...................... start with Python (Windows)
  start-rinex.sh ....................... start with Python (Linux/Mac)
  download-datasets.py ................. offline dataset downloader
                                         (ready-made 2018 / 2022 / 2026
                                         six-day windows)
  sentinel-jam-monitor-python.zip ...... ALL-IN-ONE PACKAGE (Python):
                                         app + server + launchers +
                                         download-datasets.py +
                                         README-PYTHON.txt
  README-PYTHON.txt .................... package documentation
                                         (requirements, usage,
                                         troubleshooting)
  sentinel-rfi-monitor-nextjs.zip ...... full version (Next.js,
                                         requires Node.js 20+)
  github/ .............................. GitHub publication documents
                                         (README.md, repo description)

QUICK START (RECOMMENDED, PYTHON)
  1. Extract sentinel-jam-monitor-python.zip into a NEW empty folder
  2. Windows: double-click start-rinex.bat  |  Mac/Linux: bash start-rinex.sh
  3. Browser opens on http://localhost:8123
  4. To analyze real RINEX content: click the CDSE LOGIN chip (top bar),
     enter your free Copernicus Data Space credentials, then ANALYZE RINEX.

  The standalone HTML file also opens with a double click and works
  without Python: catalogue, map and panels run immediately, and local
  .TGZ / .RNX files can be analyzed without any login.

FULL DOCUMENTATION
  - README-PYTHON.txt  (Python package: requirements, usage, errors)
  - github/README.md   (repository manual: methodology, how to use,
                        references)
