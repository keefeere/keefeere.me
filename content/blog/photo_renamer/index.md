---
title: "Made Photo Renaming tool with Bash and Exiftool"
date: 2025-07-25
tags: ["bash", "photo", "exif", "backup", "synology", "file-management"]
summary: "Organize your chaotic photo backups into a clean, chronological archive with a smart, cross-platform Bash script."
repo: "https://github.com/keefeere/photo_renamer"
---

# Photo Renamer

Just made Photo Renamer script

**photo_renamer.sh** is a cross-platform shell script for organizing messy
photo/video archives into a clean, chronological folder structure based on
metadata (EXIF and fallback timestamps). Designed to handle multi-device backups
with duplicates, inconsistent naming, and unsorted content.

👉 You can find the full script and instructions on GitHub: [photo_renamer.sh](https://github.com/keefeere/photo_renamer)

## ✨ Features

- ✅ EXIF-based date extraction (`DateTimeOriginal`, `CreateDate`, etc.)
- ✅ Fallback to file creation/modification time
- ✅ Smart renaming to format: `YYYYMMDD-HHMMSS.ext`
- ✅ Optional prefix (`IMG_`, `VID_`) toggle
- ✅ Duplicate-safe: adds suffixes only when needed
- ✅ Handles name collisions by appending suffixes only when needed (e.g., `IMG_20230701-143000_001.jpg`)
- ✅ Idempotent: supports repeated runs with `_Renamed` exclusion
- ✅ MPO & RAW support (e.g. `.cr2`, `.mpo`)
- ✅ CLI preview before execution
- ✅ Synology-compatible & lightweight (no DB or indexing)

## 📦 Output structure

```bash
/_Renamed/
└── 2023/
└── 07/
├── IMG_20230701-143000.jpg
├── VID_20230701-150000.mp4
└── …
```

## 🧪 Example before and after

Before:

```bash
📁 someone_phone/backup_07/DCIM/
  ├── IMG_20210809_100101.jpg
  ├── 20200902_090143(0).jpg
  ├── VID_20220326_113304.mp4
  └── IMG-20210902-WA0010.jpg
```

After:

```bash
📁 _Renamed/
  ├── 2020/09/IMG_20200902-090143.jpg
  ├── 2021/08/IMG_20210809-100101.jpg
  ├── 2021/09/IMG_20210902-000000.jpg
  └── 2022/03/VID_20220326-093327.mp4
```

## 🔧 Usage

```bash
sh photo_renamer.sh
```

- Requires exiftool
- Tested on macOS, Linux, Synology DSM (with exiftool installed)
- Interactive confirmation before renaming
- Excludes already renamed folders (_Renamed, @eaDir, @Recycle, etc.)

⚙️ Configuration

Edit constants at the top of the script:

```bash
ADD_PREFIX=0       # 1 to enable IMG_/VID_ prefix, 0 to disable
TARGET_ROOT="./_Renamed"
```

📌 Why this script?

This tool was born out of frustration with 15+ years of backups from multiples
phones, cloud dumps, messily named folders and files like
`IMG_20210809_100101.jpg` or `[17.04.2008] - 010.mp4`. It aims to:

- unify naming
- eliminate duplicates
- allow easy merge of backups
- support future deduplication and cloud import
- handles file name collisions gracefully by preserving content and avoiding overwrites
