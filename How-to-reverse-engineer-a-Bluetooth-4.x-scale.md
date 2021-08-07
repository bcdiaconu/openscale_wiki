The general procedure of reverse engineering a Bluetooth 4.x scale is as follow:

#### 1. Acquiring some Bluetooth traffic
1. Enable developer options if not already done so
1. Delete first on your smartphone any old `btsnoop_hci.log`
1. Activate the `Bluetooth HCI Snoop Log` developer option on the smartphone 
1. Disable and enable bluetooth to start logging
1. Weight yourself with the original app and note down the corresponding exact true date/time with all other information (e.g. weight, water percentage, bone mass and so on). Also note your user information like sex (male/female), body height, activity level, and age.
1. Do step 1-5 at least three times again but with different weights (e.g. weight yourself while holding a crate of beer)
1. Deactivate the `Bluetooth HCI Snoop Log` developer option
1. Save the `btsnoop_hci.log` with a meaningful filename
    1. Enable `USB debugging` developer option on the smartphone
    1. Issue `adb.exe bugreport .\debugdata` on the PC
    1. From `debugdata.zip` file created at previous step, get the `btsnoop_hci.log`; the file can be found in archive inside directory `FS\data\misc\bluetooth\logs\`

#### 2. Find out the Bluetooth services and characteristic
1. Install the openScale [development version](https://github.com/oliexdev/openScale/releases/tag/travis-dev-build).
1. Go to settings, about and enable debug log.
1. Then go to settings, Bluetooth and search for your scale.
1. Once found, click on it and openScale will then fetch information about all services and characteristics.
1. Return to settings, about and disable logging. Attach the log together with the btsnoop logs in a GitHub issue.
1. If this for some reason doesn't work you can also try the [BLE Scanner App](https://play.google.com/store/apps/details?id=com.macdom.ble.blescanner) by Bluepixel Technology LLP.

#### 3. Analyse the Bluetooth protocol
1. Open your first `btsnoop_hci.log` with wireshark version > 1.10
1. Search for the true values in the log files. A good starting point is to search for the weight
    * Convert your [decimal weight into a hex value](http://www.binaryhexconverter.com/decimal-to-hex-converter) (ignore any comma. The value is divided by 100 or 10 afterwards) for example if the weight is `75,3 kg` then the hex value is `02F1` in big-endian or `F102` in little endian
    * Look for the weight value in little endian format which is send from the scale to the app (source should be `remote()` and destination `localhost()`)
1. If you have found a value string that contains the weight try to find in this string other values as well (e.g. water percentage and date/time)
    * Decoding the date/time is the most difficult part because the format is unknown. It could be a unix time stamp or something different. A good free tool to help you to identify the used time format is [DCode](http://www.digital-detective.co.uk/freetools/decode.asp) by digital detective
1. Next we have to find out which steps are needed for the scale configuration to trigger the scale to send us the values
    * Search in wireshark for the first data package from the scale which contains your weight value 
    * Now analyse previous data packages and see and note down what values was written to which characteristic UUID (source should be `localhost()` and destination `remote()`) 
    * Note also down which UUID notification flag or indication flag was set enabled

#### 4. Links

[Verifying and Debugging Bluetooth](https://source.android.com/devices/bluetooth/verifying_debugging#debugging-with-bug-reports)
