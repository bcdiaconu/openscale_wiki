The general procedure of reverse engineering a Bluetooth 4.x scale is as follow:

#### 1. Acquiring some Bluetooth traffic
1. Delete first on your smartphone any old `btsnoop_hci.log` 
2. Turn on the `Bluetooth HCI Snoop Log` developer option on the smartphone 
3. Weight yourself with the original app and note down the corresponding exact true date/time with all other information (e.g. weight, water percentage, bone mass and so on). 
4. Turn off the `Bluetooth HCI Snoop Log` developer option
5. Save the `btsnoop_hci.log` with a meaningful filename
6. Do step 1-5 at least three times again but with different weights (e.g. weight yourself while holding a crate of beer)

#### 2. Find out the Bluetooth services and characteristic
1. Install [BLE Scanner App](https://play.google.com/store/apps/details?id=com.macdom.ble.blescanner) by Bluepixel Technology LLP
2. Scan and connect to the scale with this app
3. Note down all UUID numbers of services and characteristic which are not `Device Information`, `Generic Access` and, `Generic Attribute`
4. Additionally note down for every characteristic UUID if it readable (R), writeable (W) and/or have an indication flag (I) 

#### 3. Analyse the Bluetooth protocol
1. Open your first `btsnoop_hci.log` with wireshark version > 1.10
2. Search for the true values in the log files. A good starting point is to search for the weight
    * Convert your [decimal weight into a hex value](http://www.binaryhexconverter.com/decimal-to-hex-converter) (ignore any comma. The value is divided by 100 or 10 afterwards) for example if the weight is `75,3 kg` then the hex value is `02F1` in big-endian or `F102` in little endian
    * Look for the weight value in little endian format which is send from the scale to the app (source should be `remote()` and destination `localhost()`)
3. If you have found a value string that contains the weight try to find in this string other values as well (e.g. water percentage and date/time)
    * Decoding the date/time is the most difficult part because the format is unknown. It could be a unix time stamp or something different. A good free tool to help you to identify the used time format is [DCode](http://www.digital-detective.co.uk/freetools/decode.asp) by digital detective
4. Next we have to find out which steps are needed for the scale configuration to trigger the scale to send us the values
    * Search in wireshark for the first data package from the scale which contains your weight value 
    * Now analyse previous data packages and see and note down what values was written to which characteristic UUID (source should be `localhost()` and destination `remote()`) 
    * Note also down which UUID notification flag or indication flag was set enabled