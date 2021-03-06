**notes from https://github.com/maksverver/trisa-body-analyze**

# Trisa Body Analyze 4.0

Device name: 11257B*

Generic type: Transtek_WS1257B

In App: "1257B" maps to "GBF-1257-B" (note: single leading 1)

*If the host sets the broadcast id when pairing, then the device name becomes
"01257Baabbccdd" where <aabbccdd> is the hex-encoded broadcast id (uppercase).
Note that the device name prefix has changed from "11257B" to "01257B"!
```
% btgatt-client -d ${MAC} -t random

service - start: 0x0001, end: 0x0007, type: primary, uuid: 00001800-0000-1000-8000-00805f9b34fb
          charac - start: 0x0002, value: 0x0003, props: 0x0e, ext_props: 0x0000, uuid: 00002a00-0000-1000-8000-00805f9b34fb
          charac - start: 0x0004, value: 0x0005, props: 0x02, ext_props: 0x0000, uuid: 00002a01-0000-1000-8000-00805f9b34fb
          charac - start: 0x0006, value: 0x0007, props: 0x02, ext_props: 0x0000, uuid: 00002a04-0000-1000-8000-00805f9b34fb

service - start: 0x0008, end: 0x0008, type: primary, uuid: 00001801-0000-1000-8000-00805f9b34fb

service - start: 0x0009, end: 0x0018, type: primary, uuid: 0000180a-0000-1000-8000-00805f9b34fb
          charac - start: 0x000a, value: 0x000b, props: 0x02, ext_props: 0x0000, uuid: 00002a25-0000-1000-8000-00805f9b34fb
                  descr - handle: 0x000c, uuid: 00002904-0000-1000-8000-00805f9b34fb
          charac - start: 0x000d, value: 0x000e, props: 0x02, ext_props: 0x0000, uuid: 00002a27-0000-1000-8000-00805f9b34fb
                  descr - handle: 0x000f, uuid: 00002904-0000-1000-8000-00805f9b34fb
          charac - start: 0x0010, value: 0x0011, props: 0x02, ext_props: 0x0000, uuid: 00002a26-0000-1000-8000-00805f9b34fb
                  descr - handle: 0x0012, uuid: 00002904-0000-1000-8000-00805f9b34fb
          charac - start: 0x0013, value: 0x0014, props: 0x02, ext_props: 0x0000, uuid: 00002a29-0000-1000-8000-00805f9b34fb
                  descr - handle: 0x0015, uuid: 00002904-0000-1000-8000-00805f9b34fb
          charac - start: 0x0016, value: 0x0017, props: 0x02, ext_props: 0x0000, uuid: 00002a28-0000-1000-8000-00805f9b34fb
                  descr - handle: 0x0018, uuid: 00002904-0000-1000-8000-00805f9b34fb

service - start: 0x0019, end: 0xffff, type: primary, uuid: 00007802-0000-1000-8000-00805f9b34fb
          charac - start: 0x001a, value: 0x001b, props: 0x20, ext_props: 0x0000, uuid: 00008a21-0000-1000-8000-00805f9b34fb
                  descr - handle: 0x001c, uuid: 00002902-0000-1000-8000-00805f9b34fb
          charac - start: 0x001d, value: 0x001e, props: 0x20, ext_props: 0x0000, uuid: 00008a22-0000-1000-8000-00805f9b34fb
                  descr - handle: 0x001f, uuid: 00002902-0000-1000-8000-00805f9b34fb
          charac - start: 0x0020, value: 0x0021, props: 0x02, ext_props: 0x0000, uuid: 00008a20-0000-1000-8000-00805f9b34fb
          charac - start: 0x0022, value: 0x0023, props: 0x08, ext_props: 0x0000, uuid: 00008a81-0000-1000-8000-00805f9b34fb
          charac - start: 0x0024, value: 0x0025, props: 0x20, ext_props: 0x0000, uuid: 00008a82-0000-1000-8000-00805f9b34fb
                  descr - handle: 0x0026, uuid: 00002902-0000-1000-8000-00805f9b34fb
```
Properties: 1 broadcast, 2 read, 4, write-without-response, 8 write-with-response, 0x10 notify, 0x20 indicate

https://www.bluetooth.com/specifications/gatt/generic-attributes-overview

https://github.com/oliexdev/openScale

## Services & Characteristics:
```
  1800 Generic Access
    2a00 Device Name
    2a01 Appearance
    2a04 Peripheral Preferred Connection Parameters
  1801 Generic Attribute
  180a Device Information
    2a25 Serial Number String ("80BAAB3279E6")
    2a27 Hardware Revision String ("B2")
    2a26 Firmware Revision String ("2.4")
    2a29 Manufacturer Name String ("Transtek")
    2a28 Software Revision String ("A2")
  78b2 Weight Scale Service
    8a20 (readable) value 0x21 "Feature" returns 00 00
    8a21 (indicate) value 0x1b "Measurement"
    8a22 (indicate) value 0x1e "Append Measurement"
    8a81 (writable) value 0x23 "Download Information or Command"
    8a82 (indicate) value 0x25 "Upload Information or Command"
```
Looks similar to:
https://github.com/oliexdev/openScale/wiki/Medisana-BS444#bluetooth-4x-services--characteristics

Information below was extracted by disassembling the Trisa Vital Bluetooth app
(discontinued) and snooping on Bluetooth traffic sent by the MedM Health app.


## GENERAL PROTOCOL

To pair:
 - Put the device in pair mode (hold button for several seconds).
 - Connect via Bluetooth.
 - Register for notifications from characteristics 8a21, 8a22, 8a82.
 - Receive from 8a82: `a0AABBCCDD' (where AABBCCDD is a 32-bit password)
   Save the password somewhere.
 - Send to 8a81: `21AABBCCDD` (0x21: set broadcast id to AABBCCDD)
 - Send to 8a81: `22` (0x22: disconnect)
 - Disconnect from device to complete pairing!

To read measurements:
 - Connect via Bluetooth.
 - Register for notifications from characteristics 8a21, 8a22, 8a82.
 - Receive from 8a82: `a1AABBCCDD' (where AABBCCDD is a 32-bit challenge)
 - Send to 8a81: `20AABBCCDD' (where AABBCCDD are the bytes obtained by XOR'ing
   the challenge with the password, obtained by pairing).
 - Send to 8a81: `02AABBCCDD' (where AABBCCDD is a 32-bit little endian integer,
   the number of seconds passed since 2010-01-01 00:00:00 UTC).
 - Now the device will send past measurements and new measurements as they come
   in, from characteristic 8a21 instead of 8a82.
   Receive from 8a81: 20-byte weight measurument data (see below)
 - When done, send `22` (0x22: disconnect) and disconnect.


## WEIGHT MEASUREMENT DATA

Response look like:
```
 df   28:1e:00:fe 04:0a:70:10 00:00:00:ff  60:14:00:ff  00   09  00
 9f   d0:02:00:fe bf:19:70:10 00:00:00:ff  00:00:00:ff  00   09  00
 9f   5a:1e:00:fe ff:19:70:10 00:00:00:ff  03:14:00:ff  00   09  00
info  weight_kg   timestamp   resistance1  resistance2  uid stat ??
```
Info byte contains display unit (bits 5-6)
First bit contains display units, and indicates which other fields are present.
```
bit 0: is timestamp present?
bit 1: is resistance1 present?
bit 2: is resistance2 present?
bit 3: is user number present?
bit 4: is measurement status present?
bit 5-6: unit 0x00 = Kg, 0x20 = Lb, 0x40 = St, 0x60 = Kg (too!)
bit 7: unknown
```
Timestamp: 32-bit little-endian integer (offset since 2010-01-01 00:00:00).
Weight and resistances are encoded as base-10 floating point numbers, comprised
of a 24-bit little-endian mantissa and an 8-bit signed exponent. Value is
mantissa * pow(10, exponent).

Resistance1 seems to be always 0.

Measurement status contains three enum fields:
```
  bit 0:
    0: MEASUREMENT_STATUS_WEIGHT_STATUS_WEIGHT_UNSTABLE
    1: MEASUREMENT_STATUS_WEIGHT_STATUS_WEIGHT_STABLE
  bit 1-3:
    0x0: MEASUREMENT_STATUS_IMPEDANCE_STATUS_IDLE
    0x2: MEASUREMENT_STATUS_IMPEDANCE_STATUS_PROCESSING
    0x4: MEASUREMENT_STATUS_IMPEDANCE_STATUS_SHOES
    0x6: MEASUREMENT_STATUS_IMPEDANCE_STATUS_BAREFOOT
    0x8: MEASUREMENT_STATUS_IMPEDANCE_STATUS_FINISH
    0xa: MEASUREMENT_STATUS_IMPEDANCE_STATUS_ERROR
  bit 4:
    hasAppendMeasurement (meaning unknown; seems to be always false)
```

Final byte is always 0.

Another example:
```
  9f:ba:1d:00:fe:32:2b:71:10:00:00:00:ff:8d:14:00:ff:00:09:00
```
From the app:
```
  timestamp = Fri Sep 28 17:34 CEST 2018
  weight = 76.1 kg
  BMI 24.8, Fat: 18.3% Muscles: 43.2% Water: 60.6% Bones: 3.4 kg
```
Another example:
```
  9f:b0:1d:00:fe:dc:2f:81:10:00:00:00:ff:0a:15:00:ff:00:09:00
```
```
  WeightScaleMeasurementData(
    weight_kg=76.0,
    display_unit=<WeightUnit.KILOGRAM: 0>,
    timestamp=1539205852,
    resistance1=0.0,
    resistance2=538.6,
    user_number=0,
    measurement_weight_status=<MeasurementWeightStatus.STABLE: 1>,
    measurement_impedance_status=<MeasurementImpedanceStatus.FINISH: 4>,
    has_append_measurement_data=False)
```

  From the MedM health app (user profile: male, 186 cm, 36 years old)
```
    BMI 21.9, Fat: 14.6%, Muscles: 43.4%, Water: 64.4%, Bones: 3.4 kg
    Basal Metabolic Rate: 2776 kcal
```