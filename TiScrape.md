Ti.Scrape
=========

In last module [Ti.TimeZone](TimeZone.md) we used only system functions. In this module we try to use *third-party libs* and *background tasks*. This is a big step and very useful ;-)

For usage of third-party libraries have two possibilities: we import a compiled jar file (it is a zip file and contains class file tree ) or we import the sources. In our little demo we use both ways. 

First we use [jsoup](https://jsoup.org/download) as jar file. For this we right click on project, go to properties and select *Java BuildPath*:

![](https://raw.githubusercontent.com/AppWerft/TitaniumAndroidModuleDevelopment/master/images/d2.jpg)

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


```java
	@Kroll.method
	public void createScraper(final KrollDict options,
			final @Kroll.argument(optional = true) KrollFunction mCallback) {
		AsyncTask<Void, Void, Void> doRequest = new AsyncTask<Void, Void, Void>() {
			@SuppressWarnings({ "unchecked", "rawtypes" })
			@Override
			protected Void doInBackground(Void[] arg0) {
				int timeout = 10000;
				String url = null;
				String useragent = "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.10; rv:46.0) Gecko/20100101 Firefox/46.0";
				String rootXpath = null;
				Map<String, String> filterList = new HashMap<String, String>();
				/* reading of proxy properties: */
				if (options.containsKey("timeout")) {
					timeout = options.getInt("timeout");
				}
				if (options.containsKey("url")) {
					url = options.getString("url");
				}
				if (options.containsKey("useragent")) {
					useragent = options.getString("useragent");
				}
				if (options.containsKey("rootXpath")) {
					rootXpath = options.getString("rootXpath");
				}
				if (options.containsKey("subXpaths")) {
					filterList = (Map) options.getKrollDict("subXpaths");
				}
				KrollDict data = new KrollDict();
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
		doRequest.execute();
	}
}

```

