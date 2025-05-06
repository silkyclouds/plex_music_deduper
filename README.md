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
10. [License](#license)  

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
# If Plex runs in Docker, map each container path to where it lives on the host:
PATH_MAP = {
    "/music/compilations": "/mnt/music/Compilations",
    "/music/matched":      "/mnt/music/Music_matched",
    "/music/unmatched":    "/mnt/music/Music_dump",
    "/music/dupes":        "/mnt/music/Music_dupes/Plex_dupes",
    "/music/other":        "/mnt/music/OtherSource",
}
# If Plex runs on the same host, you can use identity mapping:
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
   
   

