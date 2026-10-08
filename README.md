# 🌫️ Smart Smoke Detection & Air Filtration System

An embedded air-quality monitoring and filtration system based on the **STM32F103C8T6** microcontroller.

The system uses **MQ-2** and **MQ-135** sensors to detect smoke and changes in air quality. Based on sensor readings, it automatically controls a two-stage fan system while providing real-time information through an **SSD1306 OLED display**.

## 🚀 Project Overview

The system is designed around a simple closed-loop air-cleaning concept:

```text
       Air / Smoke
            │
            ▼
       ┌─────────┐
       │  MQ-2   │
       │ Sensor  │
       └────┬────┘
            │
            ▼
       ┌─────────┐
       │  FAN 1  │
       │Filtering│
       └────┬────┘
            │
            ▼
       ┌─────────┐
       │ Filter  │
       └────┬────┘
            │
            ▼
       ┌─────────┐
       │  MQ-135 │
       │  Sensor │
       └────┬────┘
            │
            ▼
       ┌─────────┐
       │  FAN 2  │
       │ Exhaust │
       └─────────┘
```

The controller evaluates both sensor signals and determines whether the air is considered clean or contaminated.

## 🧠 Control Logic

### 🔴 Poor / Contaminated Air

If either sensor reaches the configured threshold:

```text
MQ-2 ≥ Threshold
        OR
MQ-135 ≥ Threshold
```

The system:

- FAN 1 → ON
- FAN 2 → OFF
- Red LED → ON
- Green LED → OFF
- Buzzer → ON
- OLED → `SMOKE / POOR`

### 🟢 Clean Air

When both sensors fall below the threshold:

```text
MQ-2 < Threshold
        AND
MQ-135 < Threshold
```

The system:

- FAN 1 → OFF
- FAN 2 → ON
- Green LED → ON
- Red LED → OFF
- Buzzer → OFF
- OLED → `AIR: CLEAN`

## 🔧 Hardware

| Component | Purpose |
|---|---|
| STM32F103C8T6 Blue Pill | Main controller |
| MQ-2 Gas Sensor | Smoke detection |
| MQ-135 Gas Sensor | Air-quality monitoring |
| 0.96" SSD1306 OLED | Real-time display |
| 5 V Relay Module | Fan switching |
| Fan 1 | Air filtration |
| Fan 2 | Clean-air exhaust |
| Red LED | Poor-air indication |
| Green LED | Clean-air indication |
| 5 V Buzzer | Audible warning |
| ST-LINK V2 | Programming/debugging |
| External Power Supply | Fans/relay/load power |

## 📌 STM32 Pin Configuration

| Function | STM32 Pin |
|---|---|
| MQ-2 Analog Output | PA0 / ADC1_IN0 |
| MQ-135 Analog Output | PA1 / ADC1_IN1 |
| Fan 1 Relay | PB0 |
| Fan 2 Relay | PA8 |
| OLED SCL | PB10 / I2C2_SCL |
| OLED SDA | PB11 / I2C2_SDA |
| Green LED | PB12 |
| Red LED | PB13 |
| Buzzer | PB14 |
| SWDIO | PA13 |
| SWCLK | PA14 |

## 📊 Current Detection Threshold

The current prototype uses:

```c
#define MQ2_THRESHOLD      1100
#define MQ135_THRESHOLD    1100
```

The threshold can be calibrated according to the sensor environment and actual measured ADC values.

## 🖥️ OLED Information

The OLED provides real-time information including:

```text
MQ2:  850
MQ135: 790

AIR: CLEAN

F1:OFF F2:ON
```

or during smoke detection:

```text
MQ2:  1400
MQ135: 1250

SMOKE / POOR

F1:ON F2:OFF
```

## ⚙️ Firmware Architecture

The firmware is developed using:

- STM32CubeMX
- STM32CubeIDE
- STM32 HAL
- Embedded C
- ADC
- I2C
- GPIO
- SSD1306 OLED driver

### Main firmware tasks

1. Initialize STM32 peripherals.
2. Initialize ADC channels.
3. Read MQ-2 sensor.
4. Read MQ-135 sensor.
5. Compare sensor values against thresholds.
6. Determine air-quality state.
7. Control filtration and exhaust fans.
8. Control LEDs and buzzer.
9. Update OLED.
10. Repeat continuously.

## 🔌 Power Architecture

The microcontroller and display can be powered through the STM32 board's USB power input after firmware programming.

The fans and other higher-current loads should use an **appropriate external power supply** through the relay/driver circuitry.

```text
                  ┌──────────────┐
USB 5V ──────────►│    STM32     │
                  └──────┬───────┘
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
           OLED        Sensors     Control
                                     │
                                     ▼
                                  Relays
                                     │
                              External Supply
                                     │
                              ┌──────┴──────┐
                              ▼             ▼
                            FAN 1         FAN 2
```

## ⚠️ Hardware Considerations

### MQ Sensor Analog Voltage

MQ sensors are commonly operated from 5 V and their analog output must be checked before connecting directly to a 3.3 V STM32 ADC input.

If the analog output can exceed **3.3 V**, a suitable voltage divider or signal-conditioning circuit should be used.

### Fans

Fans must **not** be driven directly from STM32 GPIO pins.

Use:

- Relay module, or
- MOSFET/transistor driver

with an appropriate external power supply.

### Buzzer

The prototype uses a 5 V two-pin buzzer. A transistor/MOSFET driver is recommended when the buzzer current exceeds the safe GPIO drive capability.

## 🔄 System State Machine

```text
              ┌─────────────────┐
              │   Read Sensors  │
              └────────┬────────┘
                       │
                       ▼
             ┌────────────────────┐
             │ Smoke / Poor Air?  │
             └───────┬───────┬────┘
                     │ YES   │ NO
                     ▼       ▼
              ┌──────────┐ ┌──────────┐
              │  FAN 1   │ │  FAN 1   │
              │   ON     │ │   OFF    │
              └──────────┘ └──────────┘
              ┌──────────┐ ┌──────────┐
              │  FAN 2   │ │  FAN 2   │
              │   OFF    │ │   ON     │
              └──────────┘ └──────────┘
                     │       │
                     ▼       ▼
                 RED LED   GREEN LED
                   ON         ON
                     │       │
                     ▼       ▼
                  BUZZER     SAFE
                    ON       STATE
```

## 🔮 Future Improvements

- Hysteresis-based threshold control
- Sensor calibration routine
- Temperature/humidity compensation
- PM2.5 particulate sensor integration
- Automatic filter-life monitoring
- IoT/cloud monitoring
- Mobile dashboard
- Data logging
- Fan PWM speed control
- Advanced air-quality classification
- Closed-loop filtration optimization

## 🎯 Applications

- Indoor air-quality monitoring
- Smoke detection
- Small-scale air filtration
- Smart ventilation
- Laboratory prototypes
- Industrial monitoring concepts
- Embedded control research

## 🧪 Engineering Concepts Demonstrated

This project combines:

**Embedded Systems + Sensors + ADC + Signal Conditioning + Control Logic + Automation + Human-Machine Interface + Actuation**

---

## 📜 License

This project is intended for educational, research, and portfolio purposes.
