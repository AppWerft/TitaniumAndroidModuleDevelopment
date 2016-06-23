Ti.Scrape
=========

In last module [Ti.TimeZone](TimeZone.md) we used only system functions. In this module we try to use third-party libs. This is a big step and very useful ;-)

We have two possibilities: we import a compiled jar file (it is a zip and contains class files) or we import the sources. In our little demo we use both ways. 


First we use jsoup as jar. For this we right click on project, goes to properties and select BuildPath:
![](https://raw.githubusercontent.com/AppWerft/TitaniumAndroidModuleDevelopment/master/images/d2.jpg)

With "Add External Jar" we select the jar from filesystem. Commonly we have copied it into lib.

If we only find a aar, the we can convert this into jar by simple unzip, rename the jar to the our name (in most cases the same name as the aar.)