<p align="center">
<a href="../raw/master/docs/scales/miscale.jpg" target="_blank">
<img src='../raw/master/docs/scales/miscale.jpg' width='300px' alt='missing' /> </a> <br>
<sub>Xiaomi Mi Scale (front)</sub>
</p>

## Xiaomi Mi Scale Reverse Engineering

Some of my relative bought a [Xiaomi Mi Scale](http://www.mi.com/en/scale/). After a while I had the idea to buy one for myself and give it a try to reverse engineer the Bluetooth 4.x protocol (also known as Smart Bluetooth). First of all I searched on the internet for information. Luckily I found some information about the [Bluetooth protocol](https://wiki.gentoo.moe/wiki/Mi-scale) by perillamint and there was already an Android implementation [OpenXiaomiScale](https://github.com/Mnkai/OpenXiaomiScale) by Mnkai.
A short summarized by the above information was:

* The connection was *not* encrypted at all
* The Mi Scale broadcasts the weight in the Bluetooth advertisement frame and
* You can enable a notification flag if you want to receive the weight data on every weight change
* The first byte was a config control byte (which include a stabilized flag, a weight removed flag, and the scale unit) 
* If the scale was set to lbs or jin you have to divide the received weight with 100 otherwise with 200
* The scale weight was received in little endian format

I could use these great information pieces to start my implementation and test the first connection to the Mi Scale. I enable the notification flag on the Mi Scale and now I received the weight data on every weight change.

The raw hex weight data looks like `0x62 0xAC 0x49 0xE0 0x07 0x0C 0x14 0x0D 0x1C 0x04` on every change. I quickly discovered that the weight date and time was also transmitted with the data, so in the end the byte decoding was: 

| byte | function                                                                                                                         |
|------|---------------------------------------------------------------------------------------------------------------------------------|
| 0    | status byte: <br> - Bit 0: lbs unit<br> - Bit 1-3: unknown<br> - Bit 4: jin unit<br> - Bit 5: stabilized<br> - Bit 6: unknown<br> - Bit 7: weight removed |
| 1-2  | weight (little endian)                                                                                                          |
| 3-4  | year (little endian)                                                                                                            |
| 5    | month                                                                                                                           |
| 6    | day                                                                                                                             |
| 7    | hour                                                                                                                            |
| 8    | minute                                                                                                                          |
| 9    | second                                                                                                                          |

But be aware that the weight date and time is only valid if the stabilized flag is true and the weight removed flag is false!

Unfortunately I could only receive the weight data if a user was on the scale and a connection was established. I didn't have access to the RAM of the Mi Scale were the history weight data was stored. Now the complicated part of the reverse engineering started. I had to capture and to analyse the Bluetooth transmission between the Mi Scale and the original [Mi Fit App](https://play.google.com/store/apps/details?id=com.xiaomi.hm.health).

For capturing the transmission I turned on the `Bluetooth HCI Snoop Log` under the developer options on my smartphone and started the Mi Fit App. Make sure you turn off your Bluetooth before your turn on the log and if you want to turn it off do it in reverse order. For the analysing step I opened the saved log with [wireshark](https://www.wireshark.org/). Please note that you should use a version > 1.10. The recent version supported Bluetooth much better!

<p align="center">
<a href="../raw/master/docs/misc/mi_scale_wireshark_init.png" target="_blank">
<img src='../raw/master/docs/misc/mi_scale_wireshark_init.png' width='600px' alt='missing' /> </a> <br>
<sub>Analysing initialization process with wireshark</sub>
</p>

You will find a lot of unnecessary transmissions but to find the important operations I searched for the hex value `0xE0 0x07` (year 2016 in little endian format) because I knew that every weight data contains the date and time. I found a lot of data package that was sending from the Mi Scale to the App that looks like the above weight data. The interesting part was that before the packages were send the App sends a write command to the scale with the value `0x02`. That must be the command to get the history data. 

But I didn't know on which Bluetooth Service and Characteristic it was send. So I had to find out the available Bluetooth Services and Characteristic on the Mi Scale. For this I downloaded the [BLE Scanner App](https://play.google.com/store/apps/details?id=com.macdom.ble.blescanner) by Bluepixel Technology LLP. With the help of this App I could easily discover the services and characteristics and even sending/receiving data package to/from the Mi Scale. 

| Service                                                 | Characteristics                                                                                                                                              | 
|---------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Device Information** <br>  0000180a-0000-1000-8000-00805f9b34fb | **Serial Number String** <br> 00002a25-0000-1000-8000-00805f9b34fb <br> **Software Revision String** <br> 00002a28-0000-1000-8000-00805f9b34fb <br> **System ID** <br> 00002a23-0000-1000-8000-00805f9b34fb <br> **PNP ID** <br> 00002a50-0000-1000-8000-00805f9b34fb                                           |         
| **Generic Access** <br> 00001800-0000-1000-8000-00805f9b34fb     | **Device Name** <br> 00002a00-0000-1000-8000-00805f9b34fb <br> **Appearance** <br> 00002a01-0000-1000-8000-00805f9b34fb <br> **Peripheral Privacy Flag** <br> 00002a02-0000-1000-8000-00805f9b34fb <br> **Reconnection Address** <br> 00002a03-0000-1000-8000-00805f9b34fb                                                                      |         
| **Generic Attribute** <br> 00001801-0000-1000-8000-00805f9b34fb  | **Service Changed** <br> 00002a05-0000-1000-8000-00805f9b34fb                                                                                                                                         |         
| **Weight Scale** <br> 0000181d-0000-1000-8000-00805f9b34fb       | **Current Time** <br> 00002a2b-0000-1000-8000-00805f9b34fb <br> **Weight Scale Feature** <br> 00002a9e-0000-1000-8000-00805f9b34fb <br> **Weight Measurement** <br> 00002a9d-0000-1000-8000-00805f9b34fb <br> **History Weight Measurement [Custom Characteristics]** <br> 00002a2f-0000-3512-2118-0009af100700                                 |         
| **Custom Service** <br> 00001530-0000-3512-2118-0009af100700     | **Custom Characteristics** <br> 00001531-0000-3512-2118-0009af100700 <br> **Custom Characteristic** <br> 00001532-0000-3512-2118-0009af100700 <br> **Peripheral Preferred Connection Parameters** <br> 00002a04-0000-1000-8000-00805f9b34fb <br> **Custom Characteristic** <br> 00001542-0000-3512-2118-0009af100700 <br> **Custom Characteristic** <br> 00001543-0000-3512-2118-0009af100700 |         

You can find further descriptions of the standard services and characteristics at the official [Bluetooth specification](https://www.bluetooth.com/specifications/gatt/services) website.

The weight scale service was of course the interesting part of it. we already know how the weight measurement characteristic works from the above information but there was also a custom characteristic (`0x2a2f`) available at which we can enable an indication flag and write some values. So I turned on the notification flag and wrote the value `0x02` to this characteristic. Now I received the complete history weight data of the Mi Scale. But I got two weight data at once so I had to split them up.

To get only the recently saved history weight data, send following command process to custom characteristic  (`0x2a2f`):
1. Set notification on for weight measurement history 
2. Write value `0x01 0xFF 0xFF 0xFF 0xFF` on characteristic (You will receive `0x01 0x?? 0xFF 0xFF 0xFF 0xFF` via notification. ?? is the amount of new history data)
3. Set notification off for weight measurement history 
4. Set notification on for weight measurement history 
5. Write value `0x02` to invoke receiving the recently saved history data

You will be notify with a stop command `0x03` then you have to do the following clean up procedure.

1. Send back stop command by writing value `0x03` on characteristic
2. Write value `0x04 0xFF 0xFF 0xFF 0xFF` on characteristic to acknowledge that you received the recently saved history data

I thought I was finished but if you take out the scale batteries and but them back you have to initialized the Mi Scale otherwise the date and time is wrong and the weight measurement history is not saved. To do this I had to capture and analyse the initialization process between the Mi Fit App and the Mi Fit Scale again. After some headache I found out that you can enable the history weight measurement with the following magic bytes `0x01 0x96 0x8a 0xbd 0x62` but before you have to enable the history weight measurement indication flag. To set the current date and time on the Mi Scale you have to send a write command to the Current Time Characteristics (`0x2a2b`) with the following byte order:

| byte | value                |
|------|----------------------|
| 0-1  | year (little endian) |
| 2    | month                |
| 3    | day                  |
| 4    | hour                 |
| 5    | minute               |
| 6    | second               |
| 7    | 0x03                 |
| 8    | 0x00                 |
| 9    | 0x00                 |

Somehow you have to restart the connection to write the current date and time. I don't know why. The last think that makes me wonder is that you can't delete the history weight measurement on the Mi Scale at all or I didn't find it out yet. If you know how you can delete them, please let me know.