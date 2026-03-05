---
title: Running an HP LaserJet P1005 Print Server on Synology NAS with Docker
date: 2026-03-05
tags: ["synology", "docker", "printing", "homelab"]
---

## Problem

The HP LaserJet P1005 is a classic “host-based” printer.
It does not contain permanent firmware. Each time the printer powers on, firmware must be uploaded before it can print.
On Linux systems this is typically handled by the `foo2zjs` driver. The firmware is downloaded using the `getweb` tool and
then sent to the printer device (usually /dev/usb/lp0). ￼
However, modern Synology DSM versions removed the built-in print server, so sharing an old USB printer like the P1005
requires a workaround.

### Where did the Synology print server go?

Older versions of DSM included a built-in print server. You could connect a USB printer directly to the NAS and
share it over the network through **Control Panel → External Devices → Printer**. Internally DSM used CUPS and
supported features such as **AirPrint** and **Google Cloud Print**.

A single NAS could typically share up to two USB printers with Windows, macOS, Linux and mobile devices on the local network.

Starting with **DSM 7**, Synology began reducing support for many external USB peripherals and quietly deprecated several
legacy features, including the built-in USB printer server. The management interface for printers is no longer available
on many newer systems and models.

As a result:

- many USB printers can no longer be added through DSM
- printer drivers are no longer maintained
- the feature may disappear completely in future DSM versions

Because of this, running a small container‑based print server has become the most reliable way to share a legacy USB
printer from a Synology NAS.

References:

- <https://global.download.synology.com/download/Document/Software/UserGuide/Firmware/DSM/4.3/enu/Syno_UsersGuide_NAServer_enu.pdf>
- <https://global.download.synology.com/download/Document/Software/UserGuide/Firmware/DSM/4.2/enu/Syno_UsersGuide_NAServer_enu.pdf>
- <https://macandegg.com/2021/06/synology-dsm-7-0-ends-support-for-usb-devices/>

### This article shows how to

1. Download and convert the firmware
2. Store it on the NAS
3. Run a minimal print server using Docker (Container Manager)
4. Share the printer over TCP port 9100

### How the P1005 Works

Unlike many printers, the P1005 requires firmware to be uploaded every time it powers on.
Typical firmware loading command:

```bash
cat /usr/share/foo2xqx/firmware/sihpP1005.dl > /dev/usb/lp0
```

When the firmware loads successfully, the printer LED usually flashes for a few seconds. ￼
Once firmware is loaded, the printer behaves like a simple RAW socket printer.

## Step 1 — Obtain the Firmware

Run a temporary Debian container:

```bash
docker run --rm -it debian:bookworm-slim bash
```

Inside the container:

```bash
apt update
apt install -y printer-driver-foo2zjs wget
getweb P1005
```

This downloads the original HP firmware and converts it automatically.
The resulting file appears at:

`/lib/firmware/hp/sihpP1005.dl`

Copy it to your NAS, for example:

`/volume1/DockerVolumes/PrintServer/sihpP1005.dl`

You only need to do this once.

## Step 2 — Create the Docker Compose Stack

Create `compose.yaml`:

```yaml
version: "3.8"

services:
  p1005-fw:
    image: alpine:3.19
    container_name: p1005-fw
    privileged: true
    restart: unless-stopped
    devices:
      - /dev/usb/lp0:/dev/usb/lp0
    volumes:
      - /volume1/DockerVolumes/PrintServer/sihpP1005.dl:/firmware/sihpP1005.dl:ro
    command: >
      sh -c "
      sleep 3 &&
      cat /firmware/sihpP1005.dl > /dev/usb/lp0 &&
      sleep infinity
      "

  p910nd:
    image: justinhimself/p910nd
    container_name: p910nd
    privileged: true
    restart: unless-stopped
    ports:
      - "9100:9100"
    devices:
      - /dev/usb/lp0:/dev/usb/lp0
    command: ["-b", "-d", "-f", "/dev/usb/lp0"]
    depends_on:
      - p1005-fw
```

This stack runs two containers:

- **p1005-fw** — uploads firmware to the printer
- **p910nd** — exposes the printer via TCP port 9100

## Step 3 — Deploy in Synology Container Manager

1. Open Container Manager
2. Go to Project
3. Create a new project
4. Paste the YAML
5. Deploy

Once started, check logs:

`docker logs p1005-fw`

Expected output:

```bash
upload firmware
done
```

At this moment the printer should briefly blink and spin.

## Step 4 — Add the Printer on Clients

Configure the printer as a RAW socket printer:

```bash
socket://NAS_IP:9100
```

Most operating systems support this directly.

Examples:

- macOS: System Settings → Printers → IP → HP JetDirect
- Linux: AppSocket / JetDirect
- Windows: Standard TCP/IP Port

## Notes

Firmware must be uploaded after every printer power cycle. If your NAS is connected to a UPS, but the printer is not, you
may need to restart the `p1005-fw` container after a power outage.

If the printer loses power, firmware must be uploaded again before printing works. Since the container runs
continuously, restarting `p1005-fw` will reload the firmware.

### USB device path

On most systems the printer appears as:

`/dev/usb/lp0`

If the path changes (for example `/dev/usb/lp1`), update the compose file accordingly.

## Result

This setup restores network printing for the HP LaserJet P1005 without installing a full CUPS server
and works on Synology NAS with Docker.

The solution is lightweight, reproducible, and survives NAS reboots.
