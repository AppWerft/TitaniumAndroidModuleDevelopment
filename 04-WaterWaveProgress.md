Ti.WaterWaveProgress
====================

![](https://github.com/Modificator/water-wave-progress/blob/master/screenshot/p2en.gif)

In this lecture I explain how we can use Android libs from github or other open source depots for building an Android module.

We found on github this nice ActivityIndicator with progress and plan a converting. First we go into example and we find in [MainActivity.java] this line

```java
setContentView(R.layout.activity_main);
```

WTF is this R and this res folder? Generally it is possible to build all views programmatically. That means we create a view, setting dimensions and constraints and we would add children to it. But Android in most cases works swarter. We have a separeting of logic and view. In e special folder res are a lot of XML files with all the needed details. During the build process runs[aapt](http://elinux.org/Android_aapt) and this script build a R class and with this class we cann access the view components at runtime.
