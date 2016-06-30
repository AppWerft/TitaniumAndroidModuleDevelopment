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
