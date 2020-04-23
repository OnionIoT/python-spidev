# Python Spidev

This project contains a python module for interfacing with SPI devices from user space via the spidev linux kernel driver.

This is a modified version of the code originally found [here](http://elk.informatik.fh-augsburg.de/da/da-49/trees/pyap7k/lang/py-spi)

All code is GPLv2 licensed unless explicitly stated otherwise.

## More Details on Use with the Omega2

**Full-duplex SPI transmissions are not supported.** This is due to the MediaTek MT7688 SoC used in the Omega2 family. **Half-duplex SPI transmissions ARE supported.** See the [Half-Duplex Transmissions section](#half-duplex-transmissions) below. 

See more details here:

* https://onion.io/2bt-brand-new-os-release/#spiimprovement
* http://community.onion.io/topic/3179/spi-bus-in-python/17

## Installation on Omega2

> The spidev module is available for Python2 and Python3, the installation commands are slightly different.

Connect to the Omega's command line and run the following commands to install the **Python2** module:
```
opkg update
opkg install python-light python-spidev
```

---

To install the **Python3** module, run this set of commands
```
opkg update
opkg install python3-light python3-spidev
```

## Usage

```python
import spidev
spi = spidev.SpiDev()
spi.open(bus, device)
to_send = [0x01, 0x02, 0x03]
spi.xfer(to_send)
```

## Settings


```python
import spidev
spi = spidev.SpiDev()
spi.open(bus, device)

# Settings (for example)
spi.max_speed_hz = 5000
spi.mode = 0b01

...
```

* `bits_per_word`
* `cshigh`
* `loop` - Set the "SPI_LOOP" flag to enable loopback mode
* `no_cs` - Set the "SPI_NO_CS" flag to disable use of the chip select (although the driver may still own the CS pin)
* `lsbfirst`
* `max_speed_hz`
* `mode` - SPI mode as two bit pattern of clock polarity and phase [CPOL|CPHA], min: 0b00 = 0, max: 0b11 = 3
* `threewire` - SI/SO signals shared

## Methods

Connects to the specified SPI device, opening `/dev/spidev<bus>.<device>`
```
open(bus, device)
```
---
Read n bytes from SPI device. Returns list of bytes read by SPI controller
```
readbytes(n)
```
---
Writes a list of values to SPI device.
```
writebytes(list of values)
```
---
Performs an SPI transaction. **Chip-select should be released and reactivated between blocks.**
Delay specifies the delay in usec between blocks. Returns list of bytes read by SPI controller.
```
xfer(list of values[, speed_hz, delay_usec, bits_per_word])
```
---
Performs an SPI transaction. **Chip-select should be held active between blocks.**
Returns list of bytes read by SPI controller.
```
xfer2(list of values[, speed_hz, delay_usec, bits_per_word])
```
---

Disconnects from the SPI device.
```
close()
```
---

### Half-Duplex Transmissions
Performs a half-duplex SPI transaction. **Chip-select should be held active between blocks.**
Returns list of bytes read by SPI controller. 
> ***Use this function when the intent is to write a number of bytes and then immediately read a number of bytes (register reads for example)***
```
xfer3(list of values to be written, number of bytes to read [, speed_hz, delay_usec, bits_per_word])
```



