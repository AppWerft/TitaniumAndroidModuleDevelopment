Ti.Scrape
=========

In last module [Ti.TimeZone](TimeZone.md) we used only system functions. In this module we try to use third-party libs. This is a big step and very useful ;-)

We have two possibilities: we import a compiled jar file (it is a zip and contains class files) or we import the sources. In our little demo we use both ways. 


First we use jsoup as jar. For this we right click on project, goes to properties and select Java BuildPath:

![](https://raw.githubusercontent.com/AppWerft/TitaniumAndroidModuleDevelopment/master/images/d2.jpg)

With "Add external jar" we select the jar from filesystem. Commonly we have copied it into lib.

If we only find a aar, the we can convert this into jar by simple unzip, rename the jar to the our name (in most cases the same name as the aar.)

For using the xpath functionality we copy the source tree:

![](https://raw.githubusercontent.com/AppWerft/TitaniumAndroidModuleDevelopment/master/images/d3.jpg) 

as sibling to our package tree. Now we can dive into code. Lets open ScraperModule.java.


```java
package de.appwerft.scraper;

import org.appcelerator.kroll.*;
import org.appcelerator.kroll.annotations.Kroll;
import org.appcelerator.kroll.common.Log;
import org.appcelerator.titanium.TiApplication;

import java.io.IOException;

import android.os.AsyncTask;

import java.net.MalformedURLException;
import java.util.List;
import java.util.Map;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Iterator;

import org.jsoup.nodes.Document;
import org.jsoup.Jsoup;
import us.codecraft.xsoup.*;

@Kroll.module(name = "Scraper", id = "de.appwerft.scraper")
public class ScraperModule extends KrollModule {
	// Standard Debugging variables
	private static final String LCAT = "HTMLScraper";
	public KrollFunction mCallback;

	public ScraperModule() {
		super();
	}

	@Kroll.onAppCreate
	public static void onAppCreate(TiApplication app) {
		Log.d(LCAT, "inside onAppCreate");
	}

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

