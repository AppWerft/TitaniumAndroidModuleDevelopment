Ti.Scrape
=========

In last module [Ti.TimeZone](TimeZone.md) we used only system functions. In this module we try to use *third-party libs* and *background tasks*. This is a big step and very useful ;-)

For usage of third-party libraries have two possibilities: we import a compiled jar file (it is a zip file and contains class file tree ) or we import the sources. In our little demo we use both ways. 

First we use [jsoup](https://jsoup.org/download) as jar file. For this we right click on project, go to properties and select *Java BuildPath*:

![](https://raw.githubusercontent.com/AppWerft/TitaniumAndroidModuleDevelopment/master/images/d2.jpg)

<<<<<<< HEAD
With "Add External Jar" we select the jar from filesystem. Commonly we have copied it into lib.
=======
First we download the file *jsoup-1.9.2.jar* to our lib folder. This is the standard folder for this. With *Add external Jar* we select the jar from filesystem. 

A moderne form of jar is aar. An aar is a jar plus resources. If we only find a aar in net , the we can convert this aar into jar in three steps:

1.    Extract the AAR file using standard zip extract
2.    Find the classes.jar file in the extracted files
3.    Rename it to your liking and use the wanted jar file in your project

The second way is to use the original sources. Good for code review, but in every compile in ant all the stuff will touches. 

First we look for the [sources on github](https://github.com/code4craft/xsoup). In folder *src* we find 2 trees. We take *main/java/us/codecraft/xsoup*. For this we go back to parent node and downlaod zip. After downloading we open the zip, go to src and copy the *main/java/us/codecraft/xsoup* to our src tree:

![](https://raw.githubusercontent.com/AppWerft/TitaniumAndroidModuleDevelopment/master/images/d3.jpg) 

as sibling to our package tree. 

An alternative is to build a jar file feom sources. For this we need two steps:

1. checkout the repo with git
2. starting maven to build the project

Details you can find in [Maven howto](http://maven.apache.org/plugins/maven-source-plugin/usage.html)

Now we can dive into code. Lets open ScraperModule.java. The ScraperProxy.java I have deleted. WE dont need a proxy and intances, because teh main work is done in an async task and we have a strong binding in callback. With other workds, we have no return value.

First in the header we import all required stuff, first the appc stuff, the java and android stuff and in the and we import both thirtparty libraries:

```java
package de.appwerft.scraper;

import org.appcelerator.kroll.*;
import org.appcelerator.kroll.annotations.Kroll;
import org.appcelerator.kroll.common.Log;
import org.appcelerator.titanium.TiApplication;

import java.io.IOException;
import java.net.MalformedURLException;
import java.util.List;
import java.util.Map;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Iterator;

import android.os.AsyncTask;

import org.jsoup.nodes.Document;
import org.jsoup.Jsoup;
import us.codecraft.xsoup.*;
```

Next comes the standard stuf from wizard:

```java
@Kroll.module(name = "Scraper", id = "de.appwerft.scraper")
public class ScraperModule extends KrollModule {
	// Standard Debugging variables
	private static final String LCAT = "HTMLScraper";
```

For sending the result we use a callback and for this we need a KrollFunction as class variable. The other way could be the usage of a *fireEvent*. It is simpler to implement, but if we start more the one scraper we would not have a reference to source of event and would test a parameter in payload. It seems more complicate.

```java
	public KrollFunction mCallback;

	public ScraperModule() {
		super();
	}

	@Kroll.onAppCreate
	public static void onAppCreate(TiApplication app) {
		Log.d(LCAT, "inside onAppCreate");
	}
```

The module has only one method named *createScraper*. This method has two parameters: first an object (dictionairy) with parameters like url and xpath strings and the seconde parameter is the (optional) callback. If a paramter is optional we need this special annotation *@Kroll.argument(optional = true)*. The main part of createScraper is an [async task](http://www.compiletimeerror.com/2013/01/why-and-how-to-use-asynctask.html). The constructor has three parameters. All keeps empty, because we realize the communcation in other way. 

```java
	@Kroll.method
	public void createScraper(final KrollDict options,
			final @Kroll.argument(optional = true) KrollFunction mCallback) {
		AsyncTask<Void, Void, Void> doRequest = new AsyncTask<Void, Void, Void>() {
			@SuppressWarnings({ "unchecked", "rawtypes" })
			
```
In our case we only implement the method *doInBackground*.
```java
			@Override
			protected Void doInBackground(Void[] arg0) {
```
First we set the parameters to default values:
```java
			
				int timeout = 10000;
				String url = null;
				String useragent = "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.10; rv:46.0) Gecko/20100101 Firefox/46.0";
				String rootXpath = null;
				Map<String, String> filterList = new HashMap<String, String>();
				/* reading of proxy properties: */
```
The next part is part of every module: we have to import the options from Javascript layer. Javascript is untyped language and Kroll provide us a lot of [helper functions](http://builds.appcelerator.com.s3.amazonaws.com/javadoc/org/appcelerator/kroll/KrollDict.html). The pattern is *containsKeyAndNotNull* and *getTYPE(KEY)* For some paramters like *with*, *backgroundColor* etc we have macros and it is a good idea to use it. 
```java
				if (options.containsKeyAndNotNull("timeout")) {
					timeout = options.getInt("timeout");
				}
				if (options.containsKeyAndNotNull(KEY_URL)) {
					url = options.getString(KEY_URL);
				}
				if (options.containsKeyAndNotNull("useragent")) {
					useragent = options.getString("useragent");
				}
				if (options.containsKey("rootXpath")) {
					rootXpath = options.getString("rootXpath");
				}
				if (options.containsKey("subXpaths")) {
					filterList = (Map) options.getKrollDict("subXpaths");
				}
```
Now we have imported all stuff. If the paramter object is not plain (like in this project) and has more the one level, you can use this patterns recursivly. In this case you can use *getKrollDict(NAME_OF_PROPERTY)* Please look to import of region in ti.map. As second parameter of get*() you use a default value.


Jsoup has an own httpclient implemetation, we us it. The result is a Document and now we can use the xsoup library to parse. paramter from JS layer is an object. Keys are only keys form communiaction and the values are xpath's.  
```java
				KrollDict data = new KrollDict(); // collector for result
				try {
					Document rootDoc = null;
					Document pageDoc = Jsoup.connect(url).userAgent(useragent)
							.timeout(timeout).ignoreContentType(true).get();
					if (rootXpath != null) {
						rootDoc = Jsoup.parse(Xsoup.compile(rootXpath)
								.evaluate(pageDoc).get());
					} else {
						rootDoc = pageDoc;
					}
					List<HashMap<String, String>> resultList = getMatchesByFilter(
							rootDoc, filterList);
					data.put("items", resultList.toArray());
					data.put("count", resultList.size());
					data.put("success", true);
```

Now in *data* all results. With the next line we call the callback.

```java
					mCallback.call(getKrollObject(), data);
				} catch (MalformedURLException e) {
					data.put("error", "MalformedURLException");
					mCallback.call(getKrollObject(), data);
					e.printStackTrace();
				} catch (IOException e) {
					data.put("error", "IOException");
					mCallback.call(getKrollObject(), data);
					e.printStackTrace();
				}
				return null;
			}
```
The logic above generates a HashMap of ArrayLists. On JS layer we aspect an array of object. For this conversion we need this private method:

```java
			private List<HashMap<String, String>> getMatchesByFilter(
					Document rootDoc, Map<String, String> filterList) {
				List<HashMap<String, String>> resultList = new ArrayList<HashMap<String, String>>();
				@SuppressWarnings("rawtypes")
				Iterator it = filterList.entrySet().iterator();
				/* iterating thru sub xpath's (keys) */
				// Log.d("DOC", rootDoc.toString());
				while (it.hasNext()) {
					@SuppressWarnings("rawtypes")
					Map.Entry pair = (Map.Entry) it.next();
					String key = (String) pair.getKey();
					String filter = (String) pair.getValue();
					/* getting all matches: */

					List<String> matchingList = Xsoup.compile(filter)
							.evaluate(rootDoc).list();
					// Log.d("MATCH", key + "=" + matchingList.toString());
					while (resultList.size() < matchingList.size()) {
						resultList.add(new HashMap<String, String>());
					}
					/* iterating thru matchingList and inserting of match: */
					int i = 0;
					while (i < matchingList.size()) {
						/* update map at position : */
						resultList.get(i).put(key, matchingList.get(i));
						i++;
					}
					it.remove();
				}
				return resultList;
			}
		};
```
Executimg of async task:
```java
		
		doRequest.execute();
	}
}

```
>>>>>>> a710c01035f54ee8a5e2a8e177bd5c43d3cc96a1

