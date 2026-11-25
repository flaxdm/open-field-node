# Module Standard Draft

## Module classes

- Core;
- Network;
- RF;
- SDR Host;
- Power;
- Storage;
- Status;
- Expansion.

## Mechanical requirements

- card-style module with guide rails;
- 4 mounting points;
- keyed installation;
- front panel for connectors;
- external SMA or appropriate RF connectors for RF modules;
- visible slot marking;
- thermal clearance;
- 3D-printable enclosure support.

## Electrical requirements

Backplane should provide:

- 12 V main field bus;
- controlled power rails;
- reverse polarity protection;
- current limiting;
- USB 3.0;
- Gigabit Ethernet;
- I2C for ID and telemetry;
- UART service;
- GPIO for buttons/status.

Each module must provide descriptor data and safe power behavior.
