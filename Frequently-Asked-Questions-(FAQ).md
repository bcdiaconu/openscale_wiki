* [My scale xyz is not supported why?](https://github.com/oliexdev/openScale/wiki/Frequently-Asked-Questions-(FAQ)#my-scale-xyz-is-not-supported-why)
* [Why are the measurements are not synchronise to a cloud (e.g. Google fit / Garmin Connect / Dropbox and so on)?](https://github.com/oliexdev/openScale/wiki/Frequently-Asked-Questions-(FAQ)#why-are-the-measurements-are-not-synchronise-to-a-cloud-eg-google-fit--garmin-connect--dropbox-and-so-on)
* [Why is my language xyz is missing or incomplete?](https://github.com/oliexdev/openScale/wiki/Frequently-Asked-Questions-(FAQ)#why-is-my-language-xyz-is-missing-or-incomplete)
* [How do I get on my SBF70 scale the body metrics (water, muscle, fat percentage)?](https://github.com/oliexdev/openScale/wiki/Frequently-Asked-Questions-(FAQ)#how-do-i-get-on-my-sbf70-scale-the-body-metrics-water-muscle-fat-percentage)
* [Which scale can you recommend?](https://github.com/oliexdev/openScale/wiki/Frequently-Asked-Questions-%28FAQ%29/_edit#which-scale-can-you-recommend)

## My scale xyz is not supported why?

I don't own every scale. But openScale is open source and lives from the open source community if you want to help you could try to reverse engineer your scale by analysing the Bluetooth 4.x protocol between your scale and the original vendor app, see [here](https://github.com/oliexdev/openScale/wiki/How-to-reverse-engineer-a-Bluetooth-4.x-scale) for further information. If you understand the protocol then it is not complicated to implement it into openScale.

## Why are the measurements are not synchronise to a cloud (e.g. Google fit / Garmin Connect / Dropbox and so on)?

Personally I don't want to synchronise any health data to a company for privacy reasons but every user are free to do it, just export your data and do what you like. Additionally most of the used licenses by the cloud companies are incompatible with the GPLv3 license.

## Why is my language xyz is missing or incomplete?

I don't speak every language. If your language is missing or incomplete please translate the [strings.xml](https://github.com/oliexdev/openScale/blob/master/android_app/app/src/main/res/values/strings.xml) into your language and then create a pull request on GitHub or send it to [olie.xdev@googlemail.com](mailto:olie.xdev@googlemail.com).

## How do I get on my SBF70 scale the body metrics (water, muscle, fat percentage)?

Probably you have user names assigned by the vendors app "health coach" that are not the same as in openScale. Please make first a backup of your measurements before you continue! To resolve this problem please delete all users in the vendors app (or reset the scale with the button on the back) and then open openScale and add a new user. After you synchronise to the SBF70 scale with openScale you should receive all other body metrics. 

## Which scale can you recommend?

As I don't own all scales by myself I can't really say which works the best with openScale but in [issue #96](https://github.com/oliexdev/openScale/issues/96) I wrote some thoughts which you may consider in your decision.