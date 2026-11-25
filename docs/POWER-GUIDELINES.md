# Power Guidelines

## Main ideas

- main field bus: preferably 12 V;
- local DC/DC in modules;
- controlled rails per module;
- battery backup;
- solar input option;
- safe shutdown;
- current measurement;
- modem power domain separation.

## LTE notes

LTE modems can draw significant transient current. The power tree must be designed for bursts, not only average load.

## Survival mode

The node should support sleep, scheduled wake and low-power logging.
