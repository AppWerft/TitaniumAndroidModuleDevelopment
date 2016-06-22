Intro / Motivation
------------------


The Titanium frameworks covers a big part of native functions of Android. If you need more then you need a module. The module exposes the native API to Javascript layer


Prerequisites
-------------

First you need the newest version of Android SDK and NDK. You will find it in the internet. In standard way both folders are in ~/Library.

If you develope the modules always on the same machine (standard case), then you can create a standard *build.properties* in your home and you can copy in every new project:

```xml
titanium.platform=/Library/Application Support/Titanium/mobilesdk/osx/5.3.0.GA/android
android.platform=/Users/fuerst/Library/android-sdk-macosx/platforms/android-23
android.ndk=/Users/fuerst/Library/android-ndk-macosx/
android.sdk=/Users/fuerst/Library/android-sdk-macosx/
google.apis=/users/fuerst/Library/android-sdk-macosx//add-ons/addon_google_apis_google_inc_8
```

In the end you must change the account name ;-)

Creation of module boilerplate
------------------------------

For this we can use wizard of Studio. Nerds know how with CLI.

![](https://raw.githubusercontent.com/AppWerft/TitaniumAndroidModuleDevelopment/master/images/d1.jpg)


