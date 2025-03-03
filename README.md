# Python Spidev

This project contains a python module for interfacing with SPI devices from user space via the spidev linux kernel driver.

This is a modified version of the code originally found [here](http://elk.informatik.fh-augsburg.de/da/da-49/trees/pyap7k/lang/py-spi)

All code is GPLv2 licensed unless explicitly stated otherwise.

## More Details on Use with the Omega2

**Full-duplex SPI transmissions are not supported.** This is due to the MediaTek MT7688 SoC used in the Omega2 family. **Half-duplex SPI transmissions ARE supported.** See the [xfer3 Half-Duplex Transmission section](#xfer3-half-duplex-spi-transaction) below. 

See more details here:

* https://onion.io/2bt-brand-new-os-release/#spiimprovement
* http://community.onion.io/topic/3179/spi-bus-in-python/17

## Installation on Omega2

Connect to the Omega's command line and run the following commands to install the **Python3** module:
```
opkg update
opkg install python3-light python3-spidev
```

### Legacy Python2 Support

**Skip this section if you're using Python3**

The spidev module is available for Python2 on Onion Firmware v0.3.x, the installation commands are slightly different:

```
opkg update
opkg install python-light python-spidev
```

Note this will only work if you're using Python2 on an Omega2 running Onion Firmware v0.3.x or v0.2.x


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

Optionally change the settings:

* `bits_per_word` - change number of bits per word, expects integer value between `8` and `16`
* `cshigh` - expects `True` or `False`
* `loop` - Set the "SPI_LOOP" flag to enable loopback mode, expects `True` or `False`
* `no_cs` - Set the "SPI_NO_CS" flag to disable use of the chip select (although the driver may still own the CS pin), expects `True` or `False`
* `lsbfirst` - expects `True` or `False`
* `max_speed_hz`- set maximum bus speed in hertz, expects long integer number
* `mode` - SPI mode as two bit pattern of clock polarity and phase [CPOL|CPHA], expects binary, min: `0b00` = 0, max: `0b11` = 3
* `threewire` - SI/SO signals shared, expects `True` or `False`

## Methods

### `open()` - Connect to SPI device

Connects to the specified SPI device, opening `/dev/spidev<bus>.<device>`
```
open(bus, device)
```

---

### `readbytes()` - Read bytes from SPI bus

Read n bytes from SPI device. Returns list of bytes read by SPI controller
```
readbytes(n)
```
---

### `writebytes` - Write bytes to SPI bus

Writes a list of values to SPI device.
```
writebytes(list of values)
```
---

### `xfer()` - Full-Duplex SPI transaction, releasing CS

Performs an SPI transaction. **Chip-select should be released and reactivated between blocks.**
Delay specifies the delay in usec between blocks. Returns list of bytes read by SPI controller.
```
xfer(list of values[, speed_hz, delay_usec, bits_per_word])
```
---

### `xfer2()` - Full-Duplex SPI transaction, holding CS

Performs an SPI transaction. **Chip-select should be held active between blocks.**
Returns list of bytes read by SPI controller.
```
xfer2(list of values[, speed_hz, delay_usec, bits_per_word])
```
---

### `xfer3` - Half-Duplex SPI transaction
Performs a half-duplex SPI transaction. **Chip-select should be held active between blocks.**
Returns list of bytes read by SPI controller. 
> ***Use this function when the intent is to write a number of bytes and then immediately read a number of bytes (register reads for example)***
```
xfer3(list of values to be written, number of bytes to read [, speed_hz, delay_usec, bits_per_word])
```

---

## `close()` - Disconnect from Device

Disconnects from the SPI device.
```
close()
```


## Differences between the `xfer` functions

| Function | Recommended for use on Omega2 | Chip-Select behaviour between blocks | SPI Transfer Type                                       | Writes Successfully | Reads Successfully |
|----------|-------------------------------|--------------------------------------|---------------------------------------------------------|---------------------|--------------------|
| `xfer`   | ❌                             | Released and reactivated             | Full-duplex - sending and receiving data simultaneously | ✅                   | ❌                  |
| `xfer2`  | ❌                             | Held active                          | Full-duplex - sending and receiving data simultaneously | ✅                   | ❌                  |
| `xfer3`  | ✅                             | Held active                          | Half-duplex - send **then** receive data                | ✅                   | ✅                  |
