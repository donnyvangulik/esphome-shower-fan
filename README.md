
# Douche Ventilator (ESPHome) — humidity‑triggered shower fan

Humidity spikes the fan when someone showers, and turns it off again when humidity returns to baseline, even on naturally humid days. Built for **ESPHome** on an **ESP32 (WT32‑ETH01)** with a **DFRobot SHT31 weatherproof sensor**.

> **Sensor:** DFRobot SHT31 Weatherproof Temperature & Humidity — TinyTronics: https://www.tinytronics.nl/nl/sensoren/lucht/vochtigheid/dfrobot-sht31-weerbestendige-temperatuur-en-luchtvochtigheidssensor  
> **Fan (example):** 125 mm inline duct fan, 220–240 VAC — Ventilatieland: https://www.ventilatieland.nl/nl_NL/p/buisventilator-125mm-mixed-flow-etamaster-em-125-e2-02/8074/

---

## Features

- Double moving‑average algorithm (short vs long) to detect a **humidity spike** (shower) rather than absolute humidity.
- **Hysteresis** + **run‑on** (15 min) so it doesn’t flap.
- **Manual override** switch (auto resets in 1 h).
- **Max‑runtime failsafe** (90 min) so it can’t run forever.
- Designed around: ESP32 WT32‑ETH01, SHT31 (I²C), 230 VAC inline fan via relay/SSR.

---

## Hardware

- **Controller:** ESP32 WT32‑ETH01 (any ESP32 will work with pin changes).
- **Sensor:** DFRobot SHT31 Weatherproof (I²C, addr 0x44).
- **Fan (recommended):** 230 VAC inline duct fan. Example used: Etamaster EM‑125 E2‑02 (link above).
- **Switching:** Use a **zero‑cross AC SSR** (e.g. SSR‑10DA/25DA) **or** a mains‑rated relay (≥250 VAC, ≥2× fan current). Input side can be driven from a 3.3 V GPIO.
- **Power:** Safe, isolated 230 VAC→5 V/3.3 V PSU for the ESP32.

> ⚠️ **Mains safety**: If you’re not comfortable working with 230 VAC, get a qualified electrician. Use proper enclosures, fuses, strain‑relief, and protective earth.


### Wiring (WT32‑ETH01 ↔ SHT31)

| WT32‑ETH01 | SHT31 |
|---|---|
| 3V3 | VCC |
| GND | GND |
| GPIO32 | SCL |
| GPIO33 | SDA |

Keep leads short. For long runs, use shielded cable and keep I²C pull‑ups to 3.3 V.

### Wiring (ESP32 ↔ Fan via SSR / Relay)

- **GPIO4** → SSR/relay input (+). SSR input (3–32 VDC) can be driven directly by GPIO4.  
- SSR/relay output **in series** with the **fan’s live** conductor. Neutral goes directly to fan. Earth to fan body (if present).
- Add an inline fuse sized for the fan’s current.

---

## Software (ESPHome)

Files are in `esphome/`. Copy `secrets.yaml.example` to `secrets.yaml` and fill your values, then install using ESPHome.

### Entities exposed

- `switch.douche_ventilatie_benjamin` — the fan
- `switch.douche_fan_override` — manual override
- `sensor.douche_benjamin_luchtvochtigheid` — humidity
- `sensor.humidity_delta` — short‑term vs long‑term delta (internal by default)

---

## Fan choice: 230 VAC vs “220 V DC”

The example fan is a **230 VAC** inline duct fan, which is standard for bathrooms in the EU and recommended for simplicity. If you truly require **~220 V DC** (rare/industrial), you’ll need:
- A DC‑rated fan and DC supply at that voltage, **plus**
- A **DC‑to‑DC SSR** (e.g., Crydom D1D40) or a DC contactor,
- Proper snubbing and over‑current protection.

This project is built and tested for **230 VAC** fans. Use DC only if you know exactly what you are doing.

---

## Tuning

In `douche-ventilator.yaml`, tweak these if needed:

- `DELTA_ON` — humidity spike above baseline to start (default 5.0)
- `ABS_ON` — hard start if absolute humidity is very high (default 82%)
- `DELTA_OFF` — stop when delta falls near baseline (default 1.6)
- Run‑on delay after release (default 15min)
- Max runtime (default 90min)

---

## Repository layout

```
douche-ventilator/
├─ esphome/
│  ├─ douche-ventilator.yaml
│  └─ secrets.yaml.example
├─ home-assistant/
│  └─ lovelace-example.md
├─ LICENSE
└─ README.md
```

## Getting started

1. Copy `esphome/secrets.yaml.example` → `esphome/secrets.yaml` and edit values.
2. Open **ESPHome**, add `esphome/douche-ventilator.yaml`, and install to your device.
3. Wire the SHT31 and fan as above. Power up. The device exposes an AP for first-time Wi‑Fi if needed.
4. Add the device to **Home Assistant** via ESPHome integration.
5. Shower test: watch `Humidity Delta` rise; the fan should start, then stop after the spike subsides and the run‑on completes.

---

## License

MIT — see `LICENSE`.
