#Ti.IcyMetaClient

The sound file format embeds [ID3Tags](https://en.wikipedia.org/wiki/ID3) to deliver meta details about the trace. These data are in the tail of file. In field of live streaming is no end. Therefore the meta data will send on start of streaming and during streaming. 

![](http://www.smackfu.com/stuff/programming/shoutcast.gif)

If we start the streaming we send a special http header and then the server sends these meta data between the mp3/aac data in fixe distances. More you can find [here](http://www.smackfu.com/stuff/programming/shoutcast.html)

Now we have two ways t read this meta:

1. usage of special streaming client like [Trevor's one](https://github.com/trevorf/ti-android-streamer). It realize both: the sound data and meta data
2. or we debundle the problem and uses a  special client only for meta.

An inspiration for us is this [snippet](https://github.com/shinymayhem/radio-presets-widget/blob/master/src/com/radiopirate/android/service/IcyStreamMeta.java) The code improvements: 

1. the client should run as async task
2. the regex to parse the answer is bad (sometimes the value contains ';')

```java
package de.appwerft.icymetaclient;

@Kroll.proxy(creatableInModule = IcymetaclientModule.class)
public class IcyMetaClientProxy extends KrollProxy implements OnLifecycleEvent {
	private static final String LCAT = "ICYMETA";
	private URL url = null;
	private int pullInterval = 0; // sec
	private boolean autoStart = false;
	private String charset = "UTF-8";
	boolean isForeGround = false;

	IcyStreamMeta metaClient = null;
	KrollFunction loadCallback = null;
	KrollFunction errorCallback = null;
```
In constructor of KrollProxy we create an instance of the IcyStreamMeta class:
```java
	public IcyMetaClientProxy() {
		super();
		metaClient = new IcyStreamMeta();
	}
```
This is the try to add this 'OnLifecycleEventListener', but it doesn't work ;-(
```java
    @Override
    public void initActivity(Activity activity) {
        super.initActivity(activity);
        ((TiBaseActivity) getActivity()).addOnLifecycleEventListener(this);

    }
```
Here we import all options from javascript layer:
```java
	// Handle creation options
	@Override
	public void handleCreationDict(KrollDict options) {
		Log.d(LCAT, "Start handleCreationDict");
		super.handleCreationDict(options);
```
It is a good coding style to validate the URL parameter:
```java
		if (options.containsKey(TiC.PROPERTY_URL)) {
			try {
				url = new URL(options.getString(TiC.PROPERTY_URL));
				metaClient.setStreamUrl(url);

			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		if (options.containsKey("pullInterval")) {
			pullInterval = options.getInt("pullInterval");
		}
		if (options.containsKey("charset")) {
			charset = options.getString("charset");
		}
		if (options.containsKey("autoStart")) {
			this.autoStart = options.getBoolean("autoStart");
			Log.d(LCAT, "autoStart=" + this.autoStart);
			if (this.autoStart)
				metaClient.startTimer();
		}
```
For async callback communication we use two endpoints: 
```java
		if (options.containsKey(TiC.PROPERTY_ONLOAD)) {
			Object cb = options.get(TiC.PROPERTY_ONLOAD);
			if (cb instanceof KrollFunction) {
				loadCallback = (KrollFunction) cb;
			} else {
				Log.e(LCAT, "onload is not KrollFunction");
			}
			if (autoStart == true) {
				try {
					metaClient.refreshMeta();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
		if (options.containsKey(TiC.PROPERTY_ONERROR)) {
			Object cb = options.get(TiC.PROPERTY_ONERROR);
			if (cb instanceof KrollFunction) {
				errorCallback = (KrollFunction) cb;
			} else {
				Log.e(LCAT, "onerroris not KrollFunction");
			}
		}
	}
```
Here we expose a set of methods to javascript:
```java

	@Kroll.method
	public String getStreamURL() {
		return this.url.toString();
	}

	@Kroll.method
	public void start() {
		metaClient.startTimer();
	}

	@Kroll.method
	public void stop() {
		metaClient.stopTimer();
	}

	
	@Kroll.method
	public void setStreamUrl(String url) throws IOException {
		metaClient.setStreamUrl(new URL(url));
	}

	@Kroll.method
	public boolean isError() {
		return metaClient.isError();
	}

	@Kroll.method
	public void refreshMeta() throws IOException {
		metaClient.refreshMeta();
	}
```
These methods currently never fired. Target was to suppress net activities during sleeping.
```java
	@Override
	public void onResume(Activity activity) {
		super.onResume(activity);
		Log.d(LCAT, "onResume >>>>>>>>>>");
		isForeGround = true;
	}

	@Override
	public void onPause(Activity activity) {
		isForeGround = false;
		Log.d(LCAT, "onPause <<<<<<<<<<<<<");
		super.onPause(activity);
	}

	public void onStart(Activity activity) {
		super.onStart(activity);
		Log.d(LCAT, "onStart >>>>>>>>>>");
		isForeGround = true;
	}
```
Here the kernel - our logic:
```java

	private class IcyStreamMeta {
		private URL streamUrl;
		private Map<String, String> metadata;
		private boolean isError;
		private boolean isRunning;
		private Timer timer;
		private int oldHash = 0;

		public IcyStreamMeta() {
			// setStreamUrl(streamUrl);
			isError = false;
			timer = new Timer();
		}

```
The equivalent to javascript's setInterval() is these construct:
```java
		public void startTimer() {
			retreiveMetadata();
			if (pullInterval != 0) {
				timer.scheduleAtFixedRate(new TimerTask() {
					@Override
					public void run() {
						retreiveMetadata();
					}
				}, 0, pullInterval * 1000);
				isRunning = true;
			}
		}
```
And so we can stop the game:
```java
		public void stopTimer() {
			timer.cancel();
			isRunning = false;
		}

		@SuppressWarnings("unused")
		public URL getStreamUrl() {
			return streamUrl;
		}

		
		public void refreshMeta() throws IOException {
			retreiveMetadata();
		}

		private Map<String, String> getMetadata() throws IOException {
			if (metadata == null) {
				refreshMeta();
			}
			return metadata;
		}

		public boolean isError() {
			return isError;
		}

		public void setStreamUrl(URL streamUrl) {
			this.metadata = null;
			this.streamUrl = streamUrl;
			this.isError = false;
		}

		private void sendError(String msg) {
			KrollDict resultDict = new KrollDict();
			resultDict.put("error", msg);
			if (errorCallback != null)
				errorCallback.call(getKrollObject(), resultDict);
			else
				Log.e(LCAT, "errorCallback is null");

		}
```
The server give us a binary stream, we have to convert to string. The real data comes after metaDataOffset, therefore we have to jump into right position:
```java

		private String Stream2String(InputStream stream, int metaDataOffset) {

			// http://www.smackfu.com/stuff/programming/shoutcast.html

			final int BLOCKSIZE = 16;
			final int EOSTREAM = -1;
			if (stream == null)
				return null;
			int b; // 0...255
			int count = 0;
			int metaDataLength = BLOCKSIZE * 255; // 4080 is the max length (16

			byte[] bytesOfMetaData = new byte[metaDataLength + 1];
			boolean inData = false;
			try {
				int bytecount = 0;
				while ((b = stream.read()) != EOSTREAM) {
					count++;
					// detector.handleData(stream, 0, b);
					if (count == metaDataOffset + 1) {
						metaDataLength = b * BLOCKSIZE;
					}
					if (count > metaDataOffset + 1
							&& count < (metaDataOffset + metaDataLength)) {
						inData = true;
					} else {
						inData = false;
					}
					if (inData) {
						bytesOfMetaData[bytecount++] = (byte) b;
					}
					if (count > (metaDataOffset + metaDataLength)) {
						break;
					}
				}
			} catch (IOException e1) {
				e1.printStackTrace();
			}
			String detectedCharset = guessEncoding(bytesOfMetaData);
			String result = new String(bytesOfMetaData,
					Charset.forName(charset));
			int stringLength = result.lastIndexOf(";");
			try {
				stream.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
			if (stringLength != -1)
				return result.substring(0, stringLength).replaceAll("\r", "");
			else
				return null;
		}
```
The protocol doesn't contain data about charset but it proposes UTF-8. A lot of streaming sender (Ö1, WDR) uses Latin-1. It wbecomes a  problem if the text contains umlaute. The method 'guessEncoding' never detects a charset, don't no why. 
```java

		public String guessEncoding(byte[] bytes) {
			String DEFAULT_ENCODING = "UTF-8";
			org.mozilla.universalchardet.UniversalDetector detector = new org.mozilla.universalchardet.UniversalDetector(
					null);
			detector.handleData(bytes, 0, bytes.length);
			detector.dataEnd();
			String encoding = detector.getDetectedCharset();
			detector.reset();
			if (encoding == null) {
				encoding = DEFAULT_ENCODING;
			}
			return encoding;
		}
```
This is the part that speals with server. We start it as asnc task:
```java
		private void retreiveMetadata() {
			// Log.d(LCAT, "isForeGround=" + isForeGround);
			AsyncTask<Void, Void, Void> doRequest = new AsyncTask<Void, Void, Void>() {
				protected Void doInBackground(Void[] dummy) {
					// http://www.javased.com/?api=java.net.URLConnection
					URLConnection con = null;
```
In this KrollDict we collect all stuff:
```java

					KrollDict resultDict = new KrollDict();
					try {
						con = streamUrl.openConnection();
					} catch (IOException e) {
						sendError(e.getMessage());
						return null;
					}
```
Very important: this tells the server, we need more then audio:
```java
					
					con.setRequestProperty("Icy-MetaData", "1");
```
But only the data, no audio
```java
					
					con.setRequestProperty("Connection", "close");
					try {
						con.connect();
					} catch (IOException e) {
						sendError(e.getMessage());
						return null;
					}
```
Now we analye some headers:
```java

					int metaDataOffset = 0;
					Map<String, List<String>> headers = con.getHeaderFields();
					if (headers.containsKey("icy-name")) {
						resultDict.put("icy-name",
								headers.get("icy-name").get(0));
					}
					InputStream stream = null;
					try {
						stream = con.getInputStream();
					} catch (IOException e) {
						sendError(e.getMessage());
						return null;
					}
					if (headers.containsKey("icy-metaint")) {
						metaDataOffset = Integer.parseInt(headers.get(
								"icy-metaint").get(0));
					}

					if (metaDataOffset == 0 || stream == null) {
						isError = true;
						Log.e(LCAT, "metaDataOffset=0");
						return null;
					}
```
and now we parse the body:
```java

					String metaString = Stream2String(stream, metaDataOffset);
					if (metaString == null) {
						return null;
					}
					String[] metaParts = metaString.split("\';");
					for (int i = 0; i < metaParts.length; i++) {
						String line = metaParts[i];

						String[] keyval = line.split("=");
						if (keyval != null && keyval.length == 2) {
							String key = keyval[0];
							String val = keyval[1];
							String sanitizedValue = "";
							sanitizedValue = val.substring(1, val.length());
							resultDict.put(key, sanitizedValue);

						} else {
							Log.e(LCAT, "cannot split meta with =");

						}
					}
```
We remember the old state in a hash, because we want only send back to jaavscript layer after change of meta data
```java
					if (resultDict.hashCode() != oldHash) {
						if (loadCallback != null)
							loadCallback.call(getKrollObject(), resultDict);
						else
							Log.e(LCAT, "loadCallback is null");
						oldHash = resultDict.hashCode();
					}
					return null;
				} // do in background
			};// async task
			doRequest.execute();
		} // retreiveMetadata
	}// private class

} // main class
```
 

