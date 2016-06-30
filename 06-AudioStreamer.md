Ti.AudioStreamer
================

This module allows to stream aac/mp3 from live radio stations. Radiotext will supported. We show how you fork a project from github and add native components (C++)

<img src="https://avatars1.githubusercontent.com/u/8136319?v=3&s=460" width="80px""/>The original project is from [Trevor](https://github.com/trevorf). Thanks to him! 

The module is a proxy between javascript layer and the library [aacdecoder](https://github.com/vbartacek/aacdecoder-android) from Václav Bartáček.
<img src="https://avatars0.githubusercontent.com/u/11479601?v=3&s=460" width="80px""/>

The main functionality works in Václav's library. It connects to server, decides the protocol and calls the native decoder. For decoding we need a C++-library from [Freeware Advanced Audio (AAC) Decoder for Android](http://www.spoledge.com)

Setup
-----

First we fork the project from Trevors repo:

<img src="https://raw.githubusercontent.com/AppWerft/TitaniumAndroidModuleDevelopment/master/images/d6.png" width="400px">

<img src="https://raw.githubusercontent.com/AppWerft/TitaniumAndroidModuleDevelopment/master/images/d7.png" width="400px">

Now we go to repo of Václav and download the zipped stuff. After unzipping we copy the tree into project:

<img src="https://raw.githubusercontent.com/AppWerft/TitaniumAndroidModuleDevelopment/master/images/d8.png" width="600px">

We will  use the precompiled version of aacdecoder.so. If we would compile self we have to copy the jni folder.

After copyeing we see this folder structur:

<img src="https://raw.githubusercontent.com/AppWerft/TitaniumAndroidModuleDevelopment/master/images/d9.png" width="240px">

If we fork a project from git then the project has the wrong project nature and we cannot use the advantages of eclipse.

Let's open the  file *.project*:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<projectDescription>
	<name>AudioStreamer</name>
	<comment></comment>
	<projects>
	</projects>
	<buildSpec>
		<buildCommand>
			<name>com.appcelerator.titanium.core.builder</name>
			<arguments>
			</arguments>
		</buildCommand>
	</buildSpec>
	<natures>
		<nature>com.appcelerator.titanium.mobile.module.nature</nature>
	</natures>
</projectDescription>
```

We add two line into *natures* and now we have:

```xml
<natures>
     <nature>org.eclipse.jdt.core.javanature</nature>
     <nature>com.appcelerator.titanium.mobile.module.nature</nature>
     <nature>com.appcelerator.titanium.mobile.module.nature</nature>
</natures>
```
For pull request to Trevor's repo we need a .gitignore to supress pushing unused stuff. For this we open a shell (please green letters on black background!) and type in project root:

```
ls -1a > .gitignore
```

Now we can open the created file *.gitignore* with an editor your choise (vi, pico, joe, mc … notepad) and we see lines like:

```
.
..
.git
.gitignore
.manifest.swp
.project
CHANGELOG.txt
LICENSE
LICENSE.txt
README.md
android
assets
build
build.properties
build.xml
dist
documentation
example
hooks
libs
manifest
platform
src
timodule.xml
```

We can now remove all used folders:
```
android/build
build
.apt_generated
.gitignore
.project
CHANGELOG.txt
LICENSE.txt
android/build
bin
```

Using of precompiled aacdecoder.so's
------------------------------------

Since Marshmellow the makes trouble with old binaries (text relocations). thats why we need new one.
We download from [here](https://github.com/trevorf/ti-android-streamer/issues/7) the [android.zip](https://github.com/vbartacek/aacdecoder-android/files/90565/android.zip). In our project folder we create a folder lib with three subfolders and copy all stuff into lib folder:
We see these:
```
Rainers-MacBook-Pro-2:Ti-android-streamer fuerst$ ls ;ls -l lib/*
CHANGELOG.txt		android			build.xml		hooks			platform
LICENSE			assets			dist			lib			src
LICENSE.txt		build			documentation		libs			timodule.xml
README.md		build.properties	example			manifest
-rw-rw-r--@ 1 fuerst  staff  32724 Jan 14 11:07 lib/aacdecoder-android-0.8.jar

lib/armeabi:
-rwxr-xr-x@ 1 fuerst  staff  230588 Jan 14 11:07 libaacdecoder.so

lib/armeabi-v7a:
-rwxr-xr-x@ 1 fuerst  staff  226500 Jan 14 11:07 libaacdecoder.so

lib/mips:
-rwxr-xr-x@ 1 fuerst  staff  342760 Jan 14 11:07 libaacdecoder.so

lib/x86:
-rwxr-xr-x@ 1 fuerst  staff  255068 Jan 14 11:07 libaacdecoder.so
```


Code modifications
------------------
In the original version of Trevor the Kroll.method *play()* has only one parameter *url*. We want extend this and need a KrollDict for it. For compatibilty reasons we need a switch:

```java
@Kroll.method
public void play(Object args) {
	String url = null, charset = "UTF-8";
	int expectedKBitSecRate = 0; // auto
	if (args instanceof KrollDict) {
		KrollDict dict = (KrollDict)args;
		if (dict.containsKeyAndNotNull("url")) {
			url = dict.getString("url");
		}
		if (dict.containsKeyAndNotNull("charset")) {
			charset = dict.getString("charset");
		}
		if (dict.containsKeyAndNotNull("expectedKBitSecRate")) {
			expectedKBitSecRate = dict.getInt("expectedKBitSecRate");
		}
	} else if  (args instanceof String) {
		url = (String)args;	
	}
	if (!isCurrentlyPlaying) {
		try {
			if (aacPlayer == null) {
				aacPlayer = new MultiPlayer(playerCallback);
			}
			currentUrl = url;
			currentCharset = charset;
			if (expectedKBitSecRate == 0) {
				aacPlayer.playAsync(url);
			} else {
				// aacPlayer.playAsync(url, expectedKBitSecRate);
			}
			if (charset != null)
				aacPlayer.setMetadataCharEnc(charset);
			isCurrentlyPlaying = true;
		} catch (Throwable t) {
			Log.e(LOG, "Error starting stream: " + t);
		}
	} else {
		Log.e(LOG, "Player was currently playing");
	}
}
```
