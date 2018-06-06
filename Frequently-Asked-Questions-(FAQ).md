* [My scale xyz is not supported why?](https://github.com/oliexdev/openScale/wiki/Frequently-Asked-Questions-(FAQ)#my-scale-xyz-is-not-supported-why)
* [Why are the measurements are not synchronise to a cloud (e.g. Google fit / Garmin Connect / Dropbox and so on)?](https://github.com/oliexdev/openScale/wiki/Frequently-Asked-Questions-(FAQ)#why-are-the-measurements-are-not-synchronise-to-a-cloud-eg-google-fit--garmin-connect--dropbox-and-so-on)
* [Why is my language xyz is missing or incomplete?](https://github.com/oliexdev/openScale/wiki/Frequently-Asked-Questions-(FAQ)#why-is-my-language-xyz-is-missing-or-incomplete)
* [How do I get on my SBF70 scale the body metrics (water, muscle, fat percentage)?](https://github.com/oliexdev/openScale/wiki/Frequently-Asked-Questions-(FAQ)#how-do-i-get-on-my-sbf70-scale-the-body-metrics-water-muscle-fat-percentage)
* [Which scale can you recommend?](https://github.com/oliexdev/openScale/wiki/Frequently-Asked-Questions-%28FAQ%29#which-scale-can-you-recommend)
* [Why do openScale needs permission to the coarse location?](https://github.com/oliexdev/openScale/wiki/Frequently-Asked-Questions-%28FAQ%29#why-do-openscale-needs-permission-to-the-coarse-location)

## My scale xyz is not supported why?

I don't own every scale. But openScale is open source and lives from the open source community if you want to help you could try to reverse engineer your scale by analysing the Bluetooth 4.x protocol between your scale and the original vendor app, see [Bluetooth 4.x Reverse Engineer tutorial](https://github.com/oliexdev/openScale/wiki/How-to-reverse-engineer-a-Bluetooth-4.x-scale) for further information. If you understand the protocol then it is not complicated to implement it into openScale.

## Why are the measurements are not synchronise to a cloud (e.g. Google fit / Garmin Connect / Dropbox and so on)?

Personally I don't want to synchronise any health data to a company for privacy reasons. Therefore, openScale doesn't send any data to a cloud and not having permission to access the internet is a strong guarantee of that. Additionally, most of the used licenses by the cloud companies are incompatible with the GPLv3 license. So openScale won't support it directly but every user are free to do it anyway, just (auto) export your data and do whatever you like, see for example the [Autosync with Nextcloud tutorial](https://github.com/oliexdev/openScale/wiki/Autosync-with-Nextcloud). 

## Why is my language xyz is missing or incomplete?

I don't speak every language. If your language is missing or incomplete please translate the [strings.xml](https://github.com/oliexdev/openScale/blob/master/android_app/app/src/main/res/values/strings.xml) into your language and then create a pull request on GitHub or use [Weblate](https://hosted.weblate.org/engage/openscale/?utm_source=widget) to translate the app. The current translation status for openScale is as follow:

[![Translation status](https://hosted.weblate.org/widgets/openscale/-/multi-auto.svg)](https://hosted.weblate.org/engage/openscale/?utm_source=widget)

## How do I get on my SBF70 scale the body metrics (water, muscle, fat percentage)?

Probably you have user names assigned by the vendors app "health coach" that are not the same as in openScale. Please make first a backup of your measurements before you continue! To resolve this problem please delete all users in the vendors app (or reset the scale with the button on the back) and then open openScale and add a new user. After you synchronise to the SBF70 scale with openScale you should receive all other body metrics. 

## Which scale can you recommend?

As I don't own all scales by myself I can't really say which works the best with openScale but in [issue #96](https://github.com/oliexdev/openScale/issues/96) I wrote some thoughts which you may consider in your decision.

## Why do openScale needs permission to the coarse location?

The answer is simple because the Android API >= 6.0 needs access to the coarse location to search for Bluetooth devices. The App openScale doesn't read your location at any time. Read the official [Android Bluetooth Developer Guide](https://developer.android.com/guide/topics/connectivity/bluetooth-le.html#permissions) or read the [Android 6.0 Changelog](https://developer.android.com/about/versions/marshmallow/android-6.0-changes.html#behavior-hardware-id) for a more technical description.