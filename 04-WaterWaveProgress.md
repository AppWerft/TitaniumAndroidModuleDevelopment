Ti.WaterWaveProgress
====================

![](https://github.com/Modificator/water-wave-progress/blob/master/screenshot/p2en.gif)

In this lecture I explain how we can use Android libs from github or other open source depots for building an Android module.

We found on github this nice ActivityIndicator with progress and plan a converting. First we go into example and we find in [MainActivity.java] this line

```java
setContentView(R.layout.activity_main);
```

WTF is this R and this res folder? Generally it is possible to build all views programmatically. That means we create a view, setting dimensions and constraints and we would add children to it. But Android in most cases works swarter. We have a separeting of logic and view. In e special folder res are a lot of XML files with all the needed details. During the build process runs[aapt](http://elinux.org/Android_aapt) and this script build a R class and with this class we cann access the view components at runtime.

Now we have two problems: a Titanium project can only contains  one R class and the deveoper aspects an other pattern, he aspects a programmatically pattern.  A main part of converting is to refactore this stuff. 

First we create a new empty project and copy from github repo the package tree: 

![](https://raw.githubusercontent.com/AppWerft/TitaniumAndroidModuleDevelopment/master/images/d5.jpg)

In our folder we have created the module and the proxy file. In library tree we have to modify the file *WaterWaveProgress.java*. 

First we analyze the module file. It is very minimalistic because I have deleted all unsused lines:

```java
package de.appwerft.waterwaveprogress;

import org.appcelerator.kroll.KrollModule;
import org.appcelerator.kroll.annotations.Kroll;
import org.appcelerator.titanium.TiApplication;

@Kroll.module(name = "Waterwaveprogress", id = "de.appwerft.waterwaveprogress")
public class WaterwaveprogressModule extends KrollModule {
	public WaterwaveprogressModule() {
		super();
	}
	@Kroll.onAppCreate
	public static void onAppCreate(TiApplication app) {
	}
}
```
