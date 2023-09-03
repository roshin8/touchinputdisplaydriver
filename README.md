# Touch Input Display Driver for Android 

Android Things have been deprecated. My motivation behind trying to revive this project was that I wanted to run Android on RaspberryPi with a 3.5" GPIO display that had touch support.

However, due to my limited knowledge of Android development, although I got to get the Gradle build to pass and the Activity to display, the Android service that is supposed to run on the background that uses SPI library which enables touch doesn't work. 
This could be because I commented out a lot of code related to maven because I didn't understand how to use it. (Looks like jitpack.io is the way to go)

Anyway, I found that I can buy 3.5", 4.3" and 5" displays that has touch support with HDMI and USB which will satisfy my need.
So, I'm going to leave this code so that maybe someone in the future would still be able to make it work for GPIO displays.

## Useful articles from Original Author
https://medium.com/@dirkvranckaert/android-things-and-touch-display-compatibility-b013a77a8bb8
https://medium.com/@dirkvranckaert/android-things-and-a-waveshare-5-display-289c2ef2fe8c

## Troubleshooting Lineage OS to display using HDMI Output
https://forum.xda-developers.com/t/running-konstakang-lineage-os-on-my-rpi-4-and-hdmi-is-not-outputting.4431465/
```
Q: My display is not working. I can only see the rainbow screen but no Android boot animation. What should I do?
A: This build only supports HDMI displays that report supported resolutions using EDID. 
1920x1080 resolution is used by default with this build. You can change value in /boot/resolution.txt to use a different resolution that your display supports. 
Removing /boot/resolution.txt will use the preferred resolution of your display
```

## Displays with USB Touch Port
- 3.5"
  - https://www.amazon.com/DIYmalls-Display-Resistive-Touchscreen-Raspberry/dp/B0BFF163RR
  - https://www.amazon.com/Coolwell-Touchscreen-Raspberry-Adjustable-Brightness/dp/B0BV2FRPJ8
  - https://www.amazon.com/TUOPUONE-Capacitive-Compatible-Resolution-Adjustable/dp/B0CFDSHZVR/
  - https://www.amazon.com/OSOYOO-3-5inch-Display-Protective-Raspberry/dp/B085TC5YMR  (Maybe)
  - https://www.amazon.com/Capacitive-Viewing-Integrated-Raspberry-Computers/dp/B0BJ78JYYN (Maybe)
- 4.3"
  - https://www.amazon.com/4-3inch-HDMI-LCD-IPS-Capacitive/dp/B0852NW9FM/
  - https://www.amazon.com/Raspberry-display-screen-capacitive-touch/dp/B0C62K8P3C
- 5"
  - https://www.amazon.com/UCTRONICS-Raspberry-Portable-Capacitive-Touchscreen/dp/B07VV7RL7Y

## Additional Resources:
 - https://raspberrytips.com/android-raspberry-pi-4/
 - https://gist.github.com/talhashraf/bda3b35e98597e545103
 - https://docs.gradle.org/current/samples/sample_building_android_apps.html
 - https://github.com/android/app-bundle-samples/blob/main/InstantApps/service/app/src/main/AndroidManifest.xml


--- 
<br>
<br>

## Below are Notes from Original Author
### Prerequisites (Not sure if this is required)
Before using the driver you need to make sure you have the Input Device Confguration file (.idc) installed on your Android things device. To do so mount the SD card of your Android Things device and in the root partition under `/system/usr/idc/` you should copy the AndroidTouchInputDriver.idc file that is available in the root of this project. If you do not copy that file the driver will work but Android will think that the driver is a physical mouse, which obviously results in totally different behaviour.
### Gradle
If having issues with gradle build
 - https://github.com/phamtdat/AndroidSnippets/blob/master/Android12AndroidManifestAddMissingAndroidExported/build.gradle
 - https://confluence.atlassian.com/doc/setting-the-java_home-variable-in-windows-8895.html
 
### Default drivers
You can use the library in two different ways:

1. Using the `TouchScreenDriverApplication`
Add an `Application` to your source code and reference it from the AndroidManifest. Then extend your application from the `TouchScreenDriverApplication` and override the `getDriverProfile()` method.
The `DriverProfile` can be initiated using `WaveshareProfile.getInstance(WaveshareProfile.DIMENSION_800_480);` or `KedeiProfile.getInstance(KedeiProfile.DIMENSION_480_320);`. This will make sure the correct driver for your display configuration/dimensions will be loaded.

2. Implementing the `TouchScreenDriverManager` yourself
If you cannot extend your `Application` from the `TouchScreenDriverApplication` then you can use the `TouchScreenDriverManager` to load and unload the driver in the `onCreate` and `onDestroy` methods of your `Application` instance. The manager is a singleton class that manages any state for you. Using this method you will also have to impelement a callback that provides `DriverProfile` for your screen.

### Custom Drivers
Adding a custom driver is really simple. The hardest part in a custom driver is knowing how to read from your touch input display.
Eg: https://github.com/dirkvranckaert/touchinputdisplaydriver/pull/12/files

To implement the driver in the library you should extend the `eu.vranckaert.driver.touch.driver.Driver` class or the `eu.vranckaert.driver.touch.driver.SpiDriver` class in case your touch input display is SPI compatible. If your touch input display uses the XPT2046 touch controller you can even directly extend the `eu.vranckaert.driver.touch.driver.XPT2046Driver` class. It's the driver class that will contain the reading/writing logic for your touch display.

Next you need a `eu.vranckaert.driver.touch.profile.DriverProfile` which needs a `Vendor` (or `Vendor.UNKNOWN` in case your vendor is not yet listed), and a `ScreenDimension` specifying the width, height and screen ratio. The `Ratio` is an enum with fixed ratio values, or again a `Ratio.UNKNOWN` if the ratio is not yet listed.

### Demo
There's a demo module to test your drivers immediately. The demo contains a splash screen, some default Android controls ideally to test touch inputs (a slider, a switch and buttons) and a 'Touch Debugging' mode, when enabled drives a circle on the touch position that you discover from the display. This way you can almost perfect calibrate the driver against the specific display!
