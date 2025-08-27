---
title: "Stalker 2 Save Sync Scripts"
date: 2025-08-27
draft: false
categories: ["Tools", "Gaming"]
tags: ["Stalker 2", "Steam", "Xbox", "Save Sync", "PowerShell"]
description: "Scripts to synchronize save game files between Steam and Xbox versions of Stalker 2 on Windows with automatic backups."
---

I ran into an issue with synchronizing save files between the Steam and Xbox
versions of _Stalker 2_ on Windows.  
To solve this, I created a set of PowerShell scripts to back up and transfer
saves between the two versions safely.  
You can find the scripts and more details on [GitHub](https://github.com/keefeere/stalker2-steam-xbox-sync).

These scripts help synchronize save files between the Steam and Xbox Game Pass
versions of _Stalker 2_ on the same Windows PC. They create timestamped backups
and manage a rotation of recent backups to prevent data loss.

## Script Descriptions

### XboxToSteam.ps1

- Backs up current Steam saves.
- Clears the Steam save folder.
- Copies the latest Xbox saves (`xgs\<id>\SaveGames`) into the Steam save folder.

### SteamToXbox.ps1

- Backs up current Xbox saves.
- Deletes only `container.*` files in the `wgs` folder to avoid cloud overwrite.
- Clears the `xgs\<id>\SaveGames` folder.
- Copies Steam saves into the Xbox `xgs\<id>\SaveGames` folder.

## Usage

Run the scripts in PowerShell. Use the `-KeepCount` parameter to control how
many backups to keep.

### Example: Xbox → Steam

```powershell
.\XboxToSteam.ps1 -KeepCount 5
```

> Close both Steam and Stalker 2 before running.

### Example: Steam → Xbox

```powershell
.\SteamToXbox.ps1 -KeepCount 5
```

> Run while the Xbox version is running, before quitting the game, so cloud sync
> registers your saves.

## Warnings & Limitations

- Cloud sync can overwrite changes if not careful.
- Always follow usage notes:
  - XboxToSteam: close both Steam and Xbox versions.
  - SteamToXbox: run with Xbox version open, before exit.
- Backups can consume significant disk space.
- Scripts do not automate launching/closing the game or disabling cloud sync.
- Use at your own risk; manual save management carries risk of corruption.

## Licensing Note

To play and sync saves across Steam, Windows UWP Xbox version, and Xbox console
version (Deluxe or Ultimate editions), you must purchase separate licenses for
each platform. Additionally, save files from Deluxe or Ultimate editions are not
compatible with the Standard edition of the game.

## GitHub Repository

Find the scripts and more details on [GitHub](https://github.com/keefeere/stalker2-steam-xbox-sync).
