## Find the connection

```
ls /dev/ttyUSB* /dev/ttyS*
```

- /dev/ttyUSB0: Common if you are using a USB-to-RJ45 (blue) console cable.
- /dev/ttyS0: Common if you are using a native DB9 serial port.

## Connect using Screen

```
screen /dev/ttyUSB0 9600
```

`screen [device] [baud_rate]`

> the device may need to be resetted, if the console port is disabled.