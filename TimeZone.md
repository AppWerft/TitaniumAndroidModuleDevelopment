Module TimeZone
===============


As explained in [intro](README.md) we can create a boilerplate with wizard. After this a lot of code is ready to use. I try to explaine:

This is the package name and is related to path in filesystem under 'src'. A package groups java files and is in relation to visibility of method and properties: 

```java
package ti.timezone;
```

Now we import all required classes. We see some stuff from appcelerator and from java (system). If we see yellow warning on left side of screen with warning like "cannot resolve symbol xyz" we can press CtrlApple-O to solve it.


```java
import org.appcelerator.kroll.KrollModule;
import org.appcelerator.kroll.annotations.Kroll;
import org.appcelerator.titanium.TiApplication;
import org.appcelerator.kroll.KrollDict;
import java.util.TimeZone;
```

This part will built by wizard. This class (our class) inherites from [KrollModule](http://builds.appcelerator.com.s3.amazonaws.com/javadoc/org/appcelerator/kroll/KrollModule.html) 
```java
@Kroll.module(name = "Titimezone", id = "ti.timezone")
public class TitimezoneModule extends KrollModule {
```

This is constructor (same name as class, without parameters)
```java
public TitimezoneModule() {
super();
}
```

This is part of lifecycle stuff (unsused)
```java
@Kroll.onAppCreate
public static void onAppCreate(TiApplication app) {
}
```

Here begins our implementation: we define a method 'getDefaultTimezone()'. The annotation 'getProperty' is the binding to javascript layer. Return value is KrollDict. This is a special kind of HashMap and is for communiaction with javascript. First we retrieve from TimeZone class with 'getDefault()' a TimeZone object. Next we create a new empty KrollDict. And with put() we put some stuff into. In the end we return the result. Its all! 

```java
  @Kroll.getProperty
  public KrollDict getDefaultTimezone() {
    TimeZone tz = TimeZone.getDefault();
    KrollDict res = new KrollDict();
    res.put("short", tz.getDisplayName(false, TimeZone.SHORT));
    res.put("long", tz.getDisplayName(false, TimeZone.LONG));
    res.put("id", tz.getID());
    res.put("offset", (int) tz.getRawOffset());
    return res;
   }
} // end of class

```

Next we try a module-only module that is a little more complicate. The module access the internet, retrieves a XML or HTML file and parse it with xsoup. It is named [Ti.Scrape](TiScrape.md)