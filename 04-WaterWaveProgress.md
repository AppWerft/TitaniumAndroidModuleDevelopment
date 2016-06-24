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


The bigger part is the ViewProxy.java
```java
package de.appwerft.waterwaveprogress;

import org.appcelerator.kroll.KrollDict;
import org.appcelerator.kroll.annotations.Kroll;
import org.appcelerator.kroll.common.Log;
import org.appcelerator.kroll.KrollProxy;
import org.appcelerator.titanium.util.TiConvert;
import org.appcelerator.titanium.proxy.TiViewProxy;
import org.appcelerator.titanium.view.TiUIView;
import org.appcelerator.titanium.TiApplication;
import org.appcelerator.kroll.common.TiMessenger;
import org.appcelerator.kroll.common.AsyncResult;
import android.os.Message;
import android.widget.LinearLayout;
import android.widget.LinearLayout.LayoutParams;
import android.app.Activity;
import android.os.Handler;
import android.content.Context;

import cn.modificator.waterwave_progress.*;
```
First we begin with the standard class annotation. With this we can call on javascript layer:
```javascript
Module.createView();
```
After this we declare all properties for the progress view.
```java
@Kroll.proxy(creatableInModule = WaterwaveprogressModule.class)
public class ViewProxy extends TiViewProxy {
	private static final int MSG_SET_PROGRESS = 70000;
	// Standard Debugging variables
	TiApplication appContext;
	Activity activity;
	private static final String LCAT = ViewProxy.class.getSimpleName();

	public WaterWaveProgress mWaterWaveProgressView;
	/* all default attributes: */
	private int mRingColor, mRingBgColor, mWaterColor, mWaterBgColor,
			mFontSize, mTextColor;
	float mCrestCount = 1.5f;
	int mProgress = 10, mMaxProgress = 100;
	private float mRingWidth, mRing2WaterWidth;
	private boolean mShowNumerical = true, mShowRing = true;
	private int mWaveFactor = 0;
	private boolean mIsWaving = false;
	private float mAmplitude = 30.0F; // 20F
	private float mWaveSpeed = 0.070F; // 0.020F
	private int mWaterAlpha = 255; // 255
	progressView view;
```
In constructor we generate us an *appcontext* and an *activity* reference.
```java
	public ViewProxy() {
		super();
		appContext = TiApplication.getInstance();
		activity = appContext.getCurrentActivity();
	}
```
Our class inherits all from TiUiProxy and that's why we have to override some methods. A couple of words to the architecture, we work in three levels:

1. TiUiView as "container"
2. in the method *createView()* we make an instance of a view class, declared in same file
3. this native view class calls the original view
```java

	@Override
	public TiUIView createView(Activity activity) {
```
Here we create a new native view and set the dimensions:
```java
		TiUIView view = new progressView(this);
		view.getLayoutParams().autoFillsHeight = true;
		view.getLayoutParams().autoFillsWidth = true;
		return view;
	}

```
in *handleCreationDict()* we import all javascript arguments to our locale vars:
```java
	@Override
	public void handleCreationDict(KrollDict options) {
		super.handleCreationDict(options);
		Log.d(LCAT, "start ViewProxy::handleCreationDict");
		if (options.containsKeyAndNotNull("progress")) {
			mProgress = TiConvert.toInt(options, "progress");
		}
		if (options.containsKeyAndNotNull("maxProgress")) {
			mMaxProgress = TiConvert.toInt(options, "maxProgress");
		}
		if (options.containsKeyAndNotNull("ringWidth")) {
			mRingWidth = TiConvert.toFloat(options, "ringWidth");
		}
		if (options.containsKeyAndNotNull("fontSize")) {
			mFontSize = TiConvert.toInt(options, "fontSize");
		}
		if (options.containsKeyAndNotNull("ring2WaterWidth")) {
			mRing2WaterWidth = TiConvert.toFloat(options, "ring2WaterWidth");
		}
		if (options.containsKeyAndNotNull("ringColor")) {
			mRingColor = TiConvert.toColor(options, "ringColor");
		}
		if (options.containsKeyAndNotNull("ringBgColor")) {
			mRingBgColor = TiConvert.toColor(options, "ringBgColor");
		}
		if (options.containsKeyAndNotNull("waterColor")) {
			mWaterColor = TiConvert.toColor(options, "waterColor");
		}
		if (options.containsKeyAndNotNull("waterBgColor")) {
			mWaterBgColor = TiConvert.toColor(options, "waterBgColor");
		}
		if (options.containsKeyAndNotNull("textColor")) {
			mTextColor = TiConvert.toColor(options, "textColor");
		}
		if (options.containsKeyAndNotNull("showNumerical")) {
			mShowNumerical = TiConvert.toBoolean(options, "showNumerical");
		}
		if (options.containsKeyAndNotNull("showRing")) {
			mShowRing = TiConvert.toBoolean(options, "showRing");
		}
		if (options.containsKeyAndNotNull("crestCount")) {
			mCrestCount = TiConvert.toFloat(options, "crestCount");
		}
		if (options.containsKeyAndNotNull("amplitude")) {
			mAmplitude = TiConvert.toFloat(options, "amplitude");
		}
		if (options.containsKeyAndNotNull("α")) {
			mWaterAlpha = TiConvert.toInt(options, "α");
		}
		if (options.containsKeyAndNotNull("waveFactor")) {
			mWaveFactor = TiConvert.toInt(options, "waveFactor");
		}
		Log.d(LCAT, "ViewProxy::handleCreationDict finished ");
	}

	/*
	 * The progressView class extends the TiUIView class. The TiUIView can be
	 * added to other Titanium views and windows, which makes it the perfect
	 * place for a UIView to be added so that it can be displayed in a Titanium
	 * app. This class creates the native view to display. The class implements
	 * the the constructor and one method of the TiUIView class, and custom
	 * setter methods:
	 */
	public class progressView extends TiUIView {
		public progressView(TiViewProxy proxy) {
			super(proxy);
			Context context  = TiApplication
					.getInstance().getApplicationContext();
			Log.d(LCAT, "progressView started");
			LayoutParams lp = new LayoutParams(LayoutParams.WRAP_CONTENT,
					LayoutParams.WRAP_CONTENT);
			LinearLayout container = new LinearLayout(proxy.getActivity());
			container.setLayoutParams(lp);

			mWaterWaveProgressView = new WaterWaveProgress(context);

			mWaterWaveProgressView.setAmplitude(mAmplitude);
			mWaterWaveProgressView.setCrestCount(mCrestCount);
			mWaterWaveProgressView.setFontSize(mFontSize);
			mWaterWaveProgressView.setIsWaving(mIsWaving);
			mWaterWaveProgressView.setProgress(mProgress);
			mWaterWaveProgressView.setMaxProgress(mMaxProgress);
			mWaterWaveProgressView.setRingWidth(mRingWidth);
			mWaterWaveProgressView.setShowRing(mShowRing);
			mWaterWaveProgressView.setShowNumerical(mShowNumerical);
			mWaterWaveProgressView.setWaterColor(mWaterColor);
			mWaterWaveProgressView.setWaterBgColor(mWaterBgColor);
			mWaterWaveProgressView.setRingColor(mRingColor);
			mWaterWaveProgressView.setRingBgColor(mRingBgColor);
			mWaterWaveProgressView.setTextColor(mTextColor);
			mWaterWaveProgressView.setRingWidth(mRingWidth);
			mWaterWaveProgressView.setRing2WaterWidth(mRing2WaterWidth);
			mWaterWaveProgressView.setWaveFactor(mWaveFactor);
			mWaterWaveProgressView.setWaveSpeed(mWaveSpeed);
			mWaterWaveProgressView.setWaterAlpha(mWaterAlpha);
			Log.d(LCAT, "All attributes for View are set");
			mWaterWaveProgressView.initView(context);
			container.addView(mWaterWaveProgressView);
			setNativeView(container);
			if (proxy.hasListeners("viewCreated")) {
				fireEvent("viewCreated", null);
			}
		}

		/*
		 * This method allows the application to processes properties passed in
		 * when the view is created. In this example, the application intercepts
		 * the color property to set the native view's background color.
		 */
		@Override
		public void processProperties(KrollDict d) {
			Log.d(LCAT, "processProperties triggered");
			super.processProperties(d);
		}
	}

	/* I N T E R F A C E S to Titanium */

	@Kroll.setProperty
	@Kroll.method
	public void setProgress(final int arg) {
		if (mWaterWaveProgressView != null)
			if (!TiApplication.isUIThread()) {
				// If we are not on the UI thread, need to use a message to set
				// the progress
				TiMessenger.sendBlockingMainMessage(new Handler(TiMessenger
						.getMainMessenger().getLooper(),
						new Handler.Callback() {
							public boolean handleMessage(Message msg) {
								switch (msg.what) {
								case MSG_SET_PROGRESS: {
									AsyncResult result = (AsyncResult) msg.obj;
									result.setResult(null);
									mWaterWaveProgressView
											.setProgress(TiConvert.toInt(arg));
									return true;
								}
								}
								return false;
							}
						}).obtainMessage(MSG_SET_PROGRESS), arg);
			} else {
				mWaterWaveProgressView.setProgress(TiConvert.toInt(arg));
			}
	}

	@Kroll.method
	public void hideNumerical() {
		if (mWaterWaveProgressView != null)
			mWaterWaveProgressView.setShowNumerical(false);
	}

	@Kroll.method
	public void showNumerical() {
		if (mWaterWaveProgressView != null)
			mWaterWaveProgressView.setShowNumerical(true);
	}

	@Kroll.method
	public void hideRing() {
		if (mWaterWaveProgressView != null)
			mWaterWaveProgressView.setShowRing(false);
	}

	@Kroll.method
	public void showRing() {
		if (mWaterWaveProgressView != null)
			mWaterWaveProgressView.setShowRing(true);
	}

	@Kroll.method
	public void setCrestCount(int arg) {
		if (mWaterWaveProgressView != null)
			mWaterWaveProgressView.setCrestCount(TiConvert.toInt(arg));
	}

	@Kroll.method
	public void setRingWidth(float arg) {
		if (mWaterWaveProgressView != null)
			mWaterWaveProgressView.setRingWidth(TiConvert.toFloat(arg));
	}

	@Kroll.method
	public void setCrestCount(float arg) {
		if (mWaterWaveProgressView != null)
			mWaterWaveProgressView.setCrestCount(TiConvert.toFloat(arg));
	}

	@Kroll.method
	public void setAmplitude(float arg) {
		if (mWaterWaveProgressView != null)
			mWaterWaveProgressView.setAmplitude(TiConvert.toFloat(arg));
	}

	@Kroll.method
	public void setWaveSpeed(float arg) {
		if (mWaterWaveProgressView != null)
			mWaterWaveProgressView.setWaveSpeed(TiConvert.toFloat(arg));
	}

	@Kroll.method
	public void setWaterAlpha(float arg) {
		if (mWaterWaveProgressView != null)
			mWaterWaveProgressView.setWaterAlpha(TiConvert.toFloat(arg));
	}

}
```
