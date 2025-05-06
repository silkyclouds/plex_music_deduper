# Plex Music Deduper

> **Deduplicate** your Plex music library by choosing a single “super-master” edition per album, merging missing tracks, moving losing editions aside, and triggering a targeted Plex rescan & emptyTrash.

---

## 🔍 Table of Contents

1. [Overview](#overview)  
2. [Features](#features)  
3. [Prerequisites](#prerequisites)  
4. [Configuration](#configuration)  
5. [Installation](#installation)  
6. [Usage](#usage)  
7. [How It Works](#how-it-works)  
8. [Rollback & Safety](#rollback--safety)  
9. [Disclaimer](#disclaimer)  

---

## 📖 Overview

When your Plex music library grows, it’s easy to end up with multiple editions of the same album scattered across your filesystem. **Plex Music Deduper** automates:

- **Selecting** the highest-quality, most complete edition  
- **Merging** any unique bonus tracks into that edition  
- **Moving** all losing editions into a configurable “dupes” folder (with rollback support)  
- **Triggering** Plex to rescan just the affected artist/album path and empty its trash so your UI updates immediately  

End result: exactly one version of each album in Plex, with no lost tracks.

---

## 🚀 Features

- **Audio fingerprinting & scoring**  
  - Format priority: FLAC/APE/ALAC (3) > WAV/M4A (2) > MP3/OGG (1)  
  - Bit-depth, sample rate, duration factored in  
- **Automatic track merging** into chosen super-master  
- **Safe move** of duplicates into a “dupes” directory  
- **Rollback log** to undo moves  
- **Targeted Plex rescan** (only artist/album folder) + emptyTrash  
- **Dry-run**, **verbose**, and **debug** modes  

---

## ⚙️ Prerequisites

- **Python 3.8+**  
- **Chromaprint CLI** (`fpcalc`)  
- **MediaInfo CLI** & Python package `pymediainfo`  
- Python packages: `requests`, `tqdm`  
- A running **Plex Media Server**

---

## 🔧 Configuration

Edit the top of `plex_music_deduper.py` to match your setup:

```python
# Path to Plex’s SQLite library DB (read-only)
PLEX_DB_FILE = "/path/to/Library/Application Support/Plex Media Server/Plug-in Support/Databases/com.plexapp.plugins.library.db"

# Your Plex HTTP host & token
PLEX_HOST  = "http://YOUR_PLEX_IP:32400"
PLEX_TOKEN = "YOUR_PLEX_TOKEN"

# Numeric ID of your Plex “Music” library section
SECTION_ID = 1

# Map Plex container prefixes → real host paths
# **If Plex runs in Docker**, map each container path to where it lives on the host:
PATH_MAP = {
    "/music/lib/location/in/container": "/music/lib/location/on/host/",
}

# **If Plex runs on the same host**, you can use identity mapping:
# PATH_MAP = {"/": "/"}

# Where to drop losing editions
DUPE_ROOT = Path(PATH_MAP["/music/dupes"])
```

## 💾 Installation

1. **Clone the repo**  
   ```bash
   git clone https://github.com/yourusername/plex_music_deduper.git
   cd plex_music_deduper
   ```
   
2. **Install Python dependencies**
   ```bash
   pip install -r requirements.txt
   ```
3. **Install system tools**
   ```bash
   sudo apt install chromaprint mediainfo
   ```

## ▶️ Usage

- **Dry-run** for a single artist (no changes applied)  
  ```bash
  ./plex_music_deduper.py --artist "Artist Name" --dry-run
  ```
- **Full run** on all artists
  ```bash
  ./plex_music_deduper.py
  ```
- **Debug mode** (ultra-verbose)
  ```bash
  ./plex_music_deduper.py --debug
  ```
- **Rollback** last run’s file moves
  ```bash
  ./plex_music_deduper.py --rollback
  ```

## 🔍 How It Works

1. **Discover editions**  
   - Queries Plex SQLite DB for each album’s `ratingKey`  
   - Joins `metadata_items` → `media_items` → `media_parts`  
   - Collects all file‐paths (`mp.file`) and maps them to real host directories via `PATH_MAP`

2. **Normalize & group**  
   - Strips format suffixes from album titles (`normalize_title`)  
   - Groups all Plex “editions” sharing the same normalized title

3. **Collect file lists**  
   - Recursively scans each edition directory for audio files (`*.flac, .mp3, .wav…`)  
   - Builds a combined list of files to fingerprint

4. **Fingerprint & score**  
   - Runs `fpcalc` (Chromaprint) on each track (first 30 s) and caches results  
   - Extracts format, bit-depth, sample rate, duration via MediaInfo  
   - Scores each edition:  
     1. Format weight (FLAC/APE/ALAC = 3 → WAV/M4A = 2 → MP3/OGG = 1)  
     2. Bit depth  
     3. Sample rate  
     4. Total unique fingerprints  

5. **Choose super-master**  
   - Picks edition with highest **average format score** and most **unique fingerprints**  

6. **Merge missing tracks**  
   - Copies any fingerprint-unique tracks from losing editions into the super-master folder  
   - Avoids overwriting (appends `_alt1`, `_alt2`, … if needed)  

7. **Move losing editions**  
   - Relocates all non-master edition folders to `DUPE_ROOT` (e.g. `…/Music_dupes/Plex_dupes`)  
   - Records each move (`src|dst`) in `ROLLBACK_LOG`  

8. **Plex sync**  
   - Sends Plex API requests to:  
     1. Refresh artist metadata  
     2. Trigger a **folder scan** on the specific `/music/matched/.../Artist/Album` path  
     3. Empty the library section’s trash  

9. **Summary & rollback**  
   - Prints per-album stats (`Init` tracks, `Add` merged, `Moved` editions, `Gain` MB freed)  
   - **Rollback** mode reads `ROLLBACK_LOG` and moves directories back to original locations
   
## ⚠️ Disclaimer

- **Use at your own risk.** This tool reflects *my* personal workflow: prioritizing a single high-quality “super-master” and relocating losing editions.  
- **Dry-run first!** Always test with `--dry-run` on a small artist or album to verify behavior before touching your entire library.  
- **Backup recommended:** Even with rollback support, create a backup of your music files and database before running.  
- **Not one-size-fits-all:** Review and adjust scoring criteria, folder mappings, and merge logic to suit your own requirements.  
- **No warranties:** This script is provided “as is.” I’m not responsible for any data loss, misplacement, or other issues that may occur.  
