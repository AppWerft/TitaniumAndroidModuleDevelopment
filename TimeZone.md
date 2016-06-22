Module TimeZone
===============


As explained in [intro](README.md) we can create a boilerplate with wizard. After this a lot of code is ready to use. I try to explaine:

This is the package name and is related to path in filesystem under 'src'. A package groups java files and is in relation to visibility of method and properties: 

```java
package ti.timezone;
```


```java
import org.appcelerator.kroll.KrollModule;
import org.appcelerator.kroll.annotations.Kroll;
import org.appcelerator.titanium.TiApplication;
import org.appcelerator.kroll.KrollDict;
import java.util.TimeZone;
```

```java
@Kroll.module(name = "Titimezone", id = "ti.timezone")
public class TitimezoneModule extends KrollModule {
```

```java
public TitimezoneModule() {
super();
}
```

```java
@Kroll.onAppCreate
public static void onAppCreate(TiApplication app) {
}
```


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
