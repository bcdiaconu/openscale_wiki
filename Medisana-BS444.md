# Medisana BS444 scale

<p align="center">
<a href="https://github.com/oliexdev/openScale/raw/master/doc/mi_scale/medisana_bs444.png" target="_blank">
<img src='https://github.com/oliexdev/openScale/raw/master/doc/mi_scale/medisana_bs444.png' width='300px' alt='missing' /> </a> <br>
<sub>Medisana BS444 (front)</sub>
</p>

#### General device information
* Device name is a random number
* HW MAC Address start with: `0xE454EB`
* Hardware revision string: `0x4232`
* Firmware revision string: `B.8`
* Manufactor name string: `Transtek`
* Software revision string: `BLE_BF`

#### Bluetooth 4.x services & characteristics
Only non standard Bluetooth services & characteristics are listed:
* Weight service UUID `000078b2-0000-1000-8000-00805f9b34fb`
   * Weight measurement characteristic UUID `00008a21-0000-1000-8000-00805f9b34fb` (indication, read-only)
   * Feature measurement characteristic UUID `00008a22-0000-1000-8000-00805f9b34fb` (indication, read-only)
   * Custom3 characteristic UUID `00008a20-0000-1000-8000-00805f9b34fb` (read-only)
   * CMD characteristic UUID `00008a81-0000-1000-8000-00805f9b34fb` (write-only)
   * Custom5 characteristic UUID `00008a82-0000-1000-8000-00805f9b34fb` (indication, read-only)

#### Analysed Bluetooth protocol
Two data packages for each measurements are send from the scale to the app which contains our information.

First package is send from UUID `0x8a21` as follow:

`1d:80:25:00:fe:80:0f:f4:0d:6a:17:00:ff:01:09:00:00:00:00`

* byte 0: unknown
* byte 1-2: weight in little endian / 100
* byte 3-4: unknown
* byte 5-8: unix date/time stamp in little endian starting from year 2010
* byte 9-18: unknown

Second package is send from UUID `0x8a22` as follow:

`6f:80:0f:f4:0d:01:21:0c:57:f1:e3:f1:5b:f1:23:f0:00:00:00`

* byte 0: unknown
* byte 1-4: unix date/time stamp in little endian starting from year 2010
* byte 5: unknown
* byte 6-7: kCal in little endian
* byte 8-9: fat percentage in little endian / 10 and first byte is masked with 0xF0
* byte 10-11: water percentage in little endian / 10 and first byte is masked with 0xF0
* byte 12-13: muscle percentage in little endian / 10 and first byte is masked with 0xF0
* byte 14-15: bone mass in little endian / 10 and first byte is masked with 0xF0
* byte 16-18: reserved bytes

To configure the scale do the following:
1. Set indication on for feature characteristic
2. Set indication on for weight measurement characteristic
3. Set indication on for custom5 measurement characteristic
4. Write magic number `02:7b:7b:f6:0d` on CMD measurement characteristics to trigger the scale