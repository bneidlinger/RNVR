# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **RangerEye** project - a comprehensive guide for building an in-vehicle Network Video Recorder (NVR) system using a Raspberry Pi 5 and ESP32-CAM modules. The project is specifically designed for a 2024 Ford Ranger but can be adapted for other vehicles.

## Key Architecture Components

The RangerEye system consists of:

### Hardware Architecture
- **Raspberry Pi 5** (8GB preferred) as the main processing unit and NVR server
- **ESP32-CAM modules** (OV2640) for video capture (designed for 2-4 cameras)
- **Storage**: microSD for OS + NVMe SSD or USB SSD for video storage
- **Network**: Pi operates as a WiFi Access Point (AP mode) for offline operation
- **Power**: 5V power distribution from portable battery or vehicle power

### Software Stack
- **Raspberry Pi OS** (64-bit) as base operating system
- **Docker Compose** for containerized NVR deployment
- **NVR Options**: Shinobi (recommended), MotionEye, or Frigate
- **Network Services**: hostapd (AP), dnsmasq (DHCP), optional UFW firewall

### Network Topology
- Pi creates `RangerEye-AP` WiFi network (10.42.0.1/24)
- ESP32-CAM modules connect as clients (10.42.0.21-24)
- Fully offline/air-gapped operation by design
- Admin access via web UI on LAN

## Development Environment

### Storage Structure
```
/mnt/rangereye/          # Main storage mount point
├── video/              # NVR video recordings
├── db/                 # Database files
└── logs/               # System logs
```

### Common Commands

#### Docker Operations
```bash
# Start NVR services
cd ~/nvr/shinobi && docker compose up -d

# View container logs
docker logs shinobi

# Check running containers
docker ps
```

#### System Maintenance
```bash
# Check WiFi AP status
systemctl status hostapd dnsmasq

# Monitor storage usage
df -h /mnt/rangereye

# Check system resources
htop
```

#### Hardware Diagnostics
```bash
# List storage devices
sudo lsblk -o NAME,MODEL,SIZE,FSTYPE,MOUNTPOINT

# Check thermal status
vcgencmd measure_temp

# View system messages
dmesg | tail -20
```

### Configuration Files
- `/etc/hostapd/hostapd.conf` - WiFi AP configuration
- `/etc/dnsmasq.d/rangereye.conf` - DHCP settings
- `/etc/fstab` - Storage mount configuration
- `~/nvr/*/compose.yml` - Docker Compose configurations

## Documentation Structure

The `docs/` directory contains comprehensive HTML build guides:
- `buildguide1.html` - Complete system build guide
- `buildguide2.html` - Alternative/updated build instructions

These guides contain detailed step-by-step instructions for:
- Hardware assembly and wiring
- ESP32-CAM firmware flashing
- Raspberry Pi OS setup and hardening
- Network configuration (AP mode)
- NVR software installation and configuration
- Power and thermal management
- Troubleshooting and maintenance

## Key Design Principles

1. **Offline Operation**: System designed to work completely air-gapped
2. **Serviceability**: Modular components with clear labeling
3. **Reliability**: Motion-based recording with automatic pruning
4. **Vehicle Integration**: Clean wiring with proper fusing and mounting
5. **Expandability**: Designed to scale from 2 to 4 cameras

## Stream URLs

ESP32-CAM modules typically expose:
- MJPEG stream: `http://<IP>:81/stream`
- Snapshot: `http://<IP>/capture`
- ESPHome RTSP: `rtsp://<IP>:8554/stream` (if enabled)

## Retention and Storage

Storage calculations (approximate):
- 720p @ 10fps ≈ 500KB/s per camera
- Motion-only recording reduces usage by 60-90%
- Recommended pruning: keep last N days or use max 80% of storage