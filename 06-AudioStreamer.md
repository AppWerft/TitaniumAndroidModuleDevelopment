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

The *aacdecoder-android-0.8.jar* we copy into lib folder.

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

After forking the '.classpath' is missing. You can see it in Project/Properties in IDE too. Here a valid file:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<classpath>
	<classpathentry kind="src" path="android/src"/>
	<classpathentry kind="src" path="android/build/.apt_generated"/>
	<classpathentry kind="con" path="org.eclipse.jdt.launching.JRE_CONTAINER"/>
	<classpathentry kind="lib" path="/Users/fuerst/Library/android-sdk-macosx/platforms/android-23/android.jar"/>
	<classpathentry kind="lib" path="/Users/fuerst/Library/android-sdk-macosx/add-ons/addon-google_apis-google-23/libs/maps.jar"/>
	<classpathentry kind="lib" path="/Library/Application Support/Titanium/mobilesdk/osx/5.2.0.GENPERM/android/titanium.jar"/>
	<classpathentry kind="lib" path="/Library/Application Support/Titanium/mobilesdk/osx/5.2.0.GENPERM/android/kroll-common.jar"/>
	<classpathentry kind="lib" path="/Library/Application Support/Titanium/mobilesdk/osx/5.2.0.GENPERM/android/kroll-apt.jar"/>
	<classpathentry kind="src" path=".apt_generated">
		<attributes>
			<attribute name="optional" value="true"/>
		</attributes>
	</classpathentry>
	<classpathentry kind="output" path="bin"/>
</classpath>
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
We download from [here](https://github.com/trevorf/ti-android-streamer/issues/7) the [android.zip](https://github.com/vbartacek/aacdecoder-android/files/90565/android.zip). In our project folder we create a folder lib with three subfolders and copy all stuff into lib and libs folder:
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

Additional we add *aacdecoder-android-0.8.jar* to our Buildpath:

<img src="https://raw.githubusercontent.com/AppWerft/TitaniumAndroidModuleDevelopment/master/images/d10.png" width="640px">

##Code modifications

###Using of constants
For better legibility we use for states constants. The Kroll.annotation exports to jacascript layer. So we cann these constanst in javascript too.
```java
@Kroll.constant
public static final int STATE_STOPPED = 0;
@Kroll.constant
public static final int STATE_STARTED = 1;
@Kroll.constant
public static final int STATE_PLAYING = 2;
@Kroll.constant
public static final int STATE_STREAMERROR = 3;
```

###Extending of play()-parameters
In the original version of Trevor the Kroll.method *play()* has only one parameter *url*. We want extend this and need a KrollDict for it. For compatibilty reasons we need a switch:

```java
@Kroll.method
public void play(Object args) {
	String url = null, charset = "UTF-8";
		int expectedKBitSecRate = 0; // auto
		Log.d(LCAT,
				">>>>>>>>>>>>>>>>>>> Starting native streaming player with args="
						+ args.toString());
		if (args != null) {
			if (args instanceof HashMap) {
				Log.d(LCAT, "args are Dict/HashMap");
				KrollDict dict = null;
				try {
					dict = new KrollDict((HashMap<String, Object>) args);
				} catch (Exception e) {
					Log.e(LCAT, "Unable to parse args" + args.toString());
					return;
				}
				if (dict.containsKeyAndNotNull("url")) {
					url = dict.getString("url");
				}
				if (dict.containsKeyAndNotNull("charset")) {
					charset = dict.getString("charset");
				}
				if (dict.containsKeyAndNotNull("expectedKBitSecRate")) {
					expectedKBitSecRate = dict.getInt("expectedKBitSecRate");
				}
			} else if (args instanceof String) {
				Log.d(LCAT, "args is String");
				url = (String) args;
			} else {
				Log.d(LCAT, "args either dict or string");
			}
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
###Callback of audioSessionId

For usage of Visualizer we need the audioSessionId, which is generated of AudioTrack.
The method play() of Václav's Multiplayer will call with PlayerCallback-interface. This implementation we overrid with 
```java
@Override
public void playerAudioTrackCreated(AudioTrack audiotrack) {
	audioSessionId = audiotrack.getAudioSessionId();
	if (hasListeners("ready")) {
		KrollDict props = new KrollDict();
		props.put("audioSessionId", audioSessionId);
		fireEvent("ready", props);
	} else
		Log.e(LOG, "cannot fire event =" + audioSessionId);
}
```
###Removing of telephony stuff

In original version of Trevors module is a functionality coded, which pauses the streaming during telephone call. This goody makes trouble:

1. after telephone call the app crashes.
2. Since Marshmellow the app needs runtime permissions for requesting telephone state, maybe we do not because there are anxious users.
3. We can use an other module to realize this smart feature from outside
4. Instantiating is part of onAppCreate node. We cannot control it by paramters
Conclusion: we remove this part.




