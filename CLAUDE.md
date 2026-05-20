# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

```bash
# Validate a config without building
esphome config <device>.yaml

# Compile firmware
esphome compile <device>.yaml

# Flash over-the-air
esphome run <device>.yaml

# Stream logs from a running device
esphome logs <device>.yaml
```

Devices: `consumption.yaml`, `coverremote.yaml`, `piscine.yaml`

Credentials live in `secrets.yaml` (gitignored). A `secrets.yaml.example` does not exist — refer to the substitution keys referenced across configs to know what to provide.

## Architecture

### Package System

All devices share common packages under `commons/`:

| File | Purpose |
|------|---------|
| `commons/device.yaml` | ESPHome identity via `system_name`, `friendly_name`, `device_description` substitutions |
| `commons/wifi.yaml` | 3 WiFi networks, static IP (`${ip}` substitution), OTA, captive portal |
| `commons/api.yaml` | Home Assistant encrypted API |
| `commons/logger.yaml` | DEBUG log level |
| `commons/restart.yaml` | HA restart button |

Common packages are included with `!include` and pass substitution variables via `vars:`.

### Devices

**`consumption.yaml`** — ESP8266 (D1 Mini) energy meter. Reads a JSY-MK-194T via Modbus (2s poll). Applies a sign flip via lambda when power is flowing back to the grid. Many sensors are commented out for future use. IP: `192.168.1.101`.

**`coverremote.yaml`** — ESP32-S3-Box cover controller for 6 motorized shutters. Uses a `cover_queue` script to serialize button presses (350ms press / 500ms release). Two remote types are defined in `coverremote/`: `simple_remote.yaml` (up/stop/down) and `multi_channel_remote.yaml` (stop+command pattern). GPIO switches use open-drain mode (`coverremote/switch.yaml`). IP: `192.168.1.100`.

**`piscine.yaml`** — ESP32-S3 (IDF framework) pool controller. Interlocked switches select pump speed (1/2/3) and gate the treatment relay: treatment activates when any speed is on, and is blocked when pump is off. Sensors: DS18B20 water temp (1-wire pin 16), DHT22 ambient (pin 15). Logs pump/treatment state on boot via Home Assistant notification. IP: `192.168.1.103`.

### Key Patterns

- **Substitutions**: Device-specific values (IP, pins, names) are passed as `vars:` in package includes.
- **YAML anchors**: Used in `piscine.yaml` for the pump interlock switch template (`&pump_interlock` / `*pump_interlock`).
- **C++ lambdas**: Used for conditional logic (power sign flip, queue processing, state checks at boot).
- **French UI strings**: Cover and pool entity names/labels are in French — keep new additions consistent.
