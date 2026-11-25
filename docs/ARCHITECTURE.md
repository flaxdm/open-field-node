# Architecture

```text
Android smartphone
   │ USB-C / BLE / Wi-Fi
   ▼
Control and mission layer
   │
   ▼
Open Field Node
├── Replaceable compute core
├── Backplane
│   ├── power bus
│   ├── USB 3.0
│   ├── Gigabit Ethernet
│   ├── I2C / SPI / UART / GPIO
│   ├── module ID bus
│   └── controlled rails
├── MCU companion
├── GPS
├── Storage
├── Network modules
│   ├── Ethernet 1
│   ├── Ethernet 2
│   ├── Wi-Fi 1
│   ├── Wi-Fi 2
│   ├── LTE modem 0 + SIM 1
│   └── LTE modem 1 + SIM 2, optional
├── RF modules
│   ├── LoRa / Meshtastic
│   ├── SubGHz / 433 / 868
│   ├── DMR hotspot, optional
│   └── SDR USB host for RTL-SDR / HackRF
└── Status / service interfaces
```

## Software layers

- Android app;
- mission engine;
- module manager;
- drivers/services;
- backplane and modules.
