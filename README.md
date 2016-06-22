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

In the next dialog we put name and the moduleID with pattern like 'com.domain.myfirst'.

If you can see, you must decide iOS or Android. If you want build a module for both platforms you can create two modules with same moduleID and copy in one folder.

The wizard creates a folder in workspace with a lot of subfolders and files. The main part works in folder */android/src/com/domain/myfirst*. We see two java files. Supposed the choosen name of Module is 'abcxyz' we see 'AbcxyzModule.java' and 'AbcxyzProxy.java'.

The 'AbcxyzModule.java' is related to javascript code

```javascript
var myModule = require('com.domain.myfirst');

```

This module followes the singleton pattern. If we require the module more then once, the content of properties is the same. We don't generate instances. 

The seconde file 'AbcxyzProxy.java' we only need if we need an instance or a return value that is more as a simple object. I.e. a view etc.

We begin with a simple module that only expose a simple API like [TimeZone](TimeZone.md). In this case you can remove the proxy. 


