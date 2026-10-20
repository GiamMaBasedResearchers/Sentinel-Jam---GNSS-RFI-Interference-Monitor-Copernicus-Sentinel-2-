================================================================
 SENTINEL JAM — STANDALONE MONITOR (Python package, all-in-one)
================================================================

CONTENTS (keep all files in the SAME folder!)
  sentinel-jam-monitor.html ... the monitor app (v11.10.0)
  sentinel-server.py .......... local server + RINEX download proxy
  download-datasets.py ........ offline dataset downloader (see below)
  start-rinex.bat ............. START ON WINDOWS (double-click)
  start-rinex.sh .............. start on Mac/Linux (bash start-rinex.sh)

REQUIREMENTS
  Only Python 3 (https://www.python.org/downloads/).
  On Windows, tick "Add python.exe to PATH" during installation.
  Node.js is NOT required for this package.

HOW TO START (Windows)
  1. Extract the zip into a NEW folder (e.g. on the Desktop)
  2. Double-click start-rinex.bat
  3. The browser opens automatically on http://localhost:8123
  4. Keep the black window open (Ctrl+C to stop)

HOW TO START (Mac/Linux)
  1. Extract the zip
  2. Terminal in that folder:  bash start-rinex.sh

AFTER START — TO DOWNLOAD AND ANALYZE RINEX FILES
  1. In the top bar click the LOGIN / CDSE chip
  2. Enter your Copernicus Data Space username and password
     (free account at https://dataspace.copernicus.eu)
  3. Then click ANALYZE RINEX: the download happens THROUGH THE
     PYTHON SERVER (server side), no browser CORS/401/400 errors.

OFFLINE DATASETS (download-datasets.py)
  1. Put download-datasets.py in any folder and run it FROM THERE:
       python download-datasets.py            (all three datasets)
       python download-datasets.py 2022       (one dataset: 2018|2022|2026)
       python download-datasets.py --check    (list only, no login needed)
  2. Enter your CDSE account once (or set CDSE_USER / CDSE_PASS env vars).
     Free account: https://dataspace.copernicus.eu
  3. Each dataset lands in its OWN folder next to where you ran the
     script (dataset-2018-10 / dataset-2022-10 / dataset-2026-10),
     each with a manifest.txt (products + validity windows):
       dataset-2018-10   validity 2018-10-10..15  (S2A + S2B era)
       dataset-2022-10   validity 2022-10-10..15  (S2A + S2B era)
       dataset-2026-10   validity 2026-10-10..15  (S2B + S2C era)
  4. In the app: RINEX content analysis -> Local RINEX files ->
     PICK FILES (all files of one dataset folder) -> ANALYZE LOCAL FILES.
  Note: recent windows (e.g. Oct 2026) have the last ~5-6 days still
  unpublished (normal pipeline delay): re-run the script later, files
  already on disk are skipped.

VERSION
  App version in this package: v11.10.0 (shown in the top bar of the app).

HOUSEKEEPING (IMPORTANT)
  - Keep all files of THIS package in the same folder and use only this
    copy. If you still keep an older copy of the app somewhere else,
    delete it: the version badge in the top bar must match the version
    listed under VERSION above.

TYPICAL ERRORS
  - "Download rejected (401)" -> you did not log in to CDSE
    (or the session expired: log in again with the top chip)
  - "python.exe: can't open file ... sentinel-server.py" -> the .py
    file was not in the same folder as the .bat: use THIS package,
    it contains everything.
  - "No AUX_GNSSRD files in the selected date range" -> daily files
    are published with ~5-6 days delay (precise orbits ~3 weeks): move
    the range a few days back.
  - Port busy -> start on another port:
    python sentinel-server.py 8124  (then open http://localhost:8124)

NOTES
  - Internet is required: data comes from the Copernicus OData APIs
    (AUX_GNSSRD, AUX_POEORB, AUX_PROQUA).
  - If the firewall asks for permission for Python: allow it.
  - Full version with H3 RFI grid, database reports and TLE tracks:
    sentinel-rfi-monitor-nextjs.zip (requires Node.js 20+).
