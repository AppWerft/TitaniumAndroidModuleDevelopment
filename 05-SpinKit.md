Ti.SpinKit
==========

<img src="https://raw.githubusercontent.com/ybq/AndroidSpinKit/master/art/screen.gif" width="180px" height="300px"/>

Lets look to the SpinKit. This is an extended ActivityIndicator. We have only constructor (create…) arguments, no methods at life time.  We go into lib folder an found the class [SpnKitView](https://github.com/ybq/Android-SpinKit/blob/master/library/src/main/java/com/github/ybq/android/spinkit/SpinKitView.java). Lets analyze the constructor:

```java
public SpinKitView(Context context) {
        this(context, null);
    }

    public SpinKitView(Context context, AttributeSet attrs) {
        this(context, attrs, R.attr.SpinKitViewStyle);
    }

    public SpinKitView(Context context, AttributeSet attrs, int defStyleAttr) {
        this(context, attrs, defStyleAttr, R.style.SpinKitView);
    }

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    public SpinKitView(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);
        TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.SpinKitView, defStyleAttr,
                defStyleRes);
        mStyle = Style.values()[a.getInt(R.styleable.SpinKitView_SpinKit_Style, 0)];
        mColor = a.getColor(R.styleable.SpinKitView_SpinKit_Color, Color.WHITE);
        a.recycle();
        init();
        setIndeterminate(true);
    }
```

The first method with only one argument calls the second *this(context, null);*. OK, then the second method calls the third and adds *R.attr.SpinKitViewStyle*.  OOps, here we have the evil R.class. We have to avoid this pattern. With other words, we cannot use this class as 1:1 copy. The plan is to use the simple (first) constructor and inject the parameters later programmatically and in the end we can call *init()* and *setIndeterminate(true)*.

SpinnerViewProxy.java
---------------------

With this class name we can call the spinner with:
```javascript
Module.createSpinnerView({…args…});
```

Now the class (I have compressed the imports):
```java

package de.appwerft.spinkit;

import org.appcelerator.kroll.KrollDict;
import android.graphics.Color;
import android.widget.LinearLayout;
import android.widget.LinearLayout.*;

import org.appcelerator.kroll.annotations.Kroll;
import org.appcelerator.titanium.proxy.TiViewProxy;
import org.appcelerator.titanium.view.TiUIView;

import com.github.ybq.android.spinkit.SpinKitView;
import com.github.ybq.android.spinkit.sprite.Sprite;
import com.github.ybq.android.spinkit.style.*;

import android.app.Activity;
```

As in every projects we have the KrollAnnotation for creater and the instance variables:
```java
@Kroll.proxy(creatableInModule = SpinKitModule.class)
public class SpinnerViewProxy extends TiViewProxy {
	public int spinnerStyle = 0;
	public int spinnerColor = Color.parseColor("#ffffff");
	private SpinnerView mView;
	public SpinnerViewProxy() {
		super();
	}
```
Here we override *createView()* and call our native view *SpinnerView*.
```java
	@Override
	public TiUIView createView(Activity activity) {
		mView = new SpinnerView(this);
		mView.getLayoutParams().autoFillsHeight = true;
		mView.getLayoutParams().autoFillsWidth = true;
		return mView;
	}
```
ANd now we import the both parameters color and type (types are constants in module):
```java
	@Override
	public void handleCreationDict(KrollDict options) {
		super.handleCreationDict(options);
		if (options.containsKeyAndNotNull("type")) {
			spinnerStyle = options.getInt("type");
		}
		if (options.containsKeyAndNotNull("color")) {
			spinnerColor = Color.parseColor(options.getString("color"));
		}
	}
```
Here begins the magic with our native view, which is called in viewproxy:
```java
	private class SpinnerView extends TiUIView {
		SpinKitView view;
		public SpinnerView(final TiViewProxy proxy) {
			super(proxy);
```
First we create a SpinKitView from library class:
```java			
			view = new SpinKitView(proxy.getActivity());
```
Attention: this is our work: we call a initView to transfer the parameter, we must implement it in library class later:
```java			
			view.initView(spinnerColor, spinnerStyle);
			view.setLayoutParams(new LayoutParams(LayoutParams.MATCH_PARENT, 1000));
```
and now we call the stuff from library class (we hoisted it from there to here)
```java
			view.refreshDrawableState();
			Sprite sprite = null;
			switch (spinnerStyle) {
			case 0:
				sprite = new RotatingPlane();
				break;
			case 1:
				sprite = new DoubleBounce();
				break;
			case 2:
				sprite = new Wave();
				break;
			case 3:
				sprite = new WanderingCubes();
				break;
			case 4:
				sprite = new Pulse();
				break;
			case 5:
				sprite = new ChasingDots();
				break;
			case 6:
				sprite = new ThreeBounce();
				break;
			case 7:
				sprite = new Circle();
				break;
			case 8:
				sprite = new CubeGrid();
				break;
			case 9:
				sprite = new FadingCircle();
				break;
			case 10:
				sprite = new FoldingCube();
				break;
			case 11:
				sprite = new RotatingCircle();
				break;
			}
			sprite.setColor(spinnerColor);
			view.setIndeterminateDrawable(sprite);
```
Following statement is important. It binds the native view to javascript layer.
```java
			setNativeView(view);
		}

		@Override
		public void processProperties(KrollDict d) {
			super.processProperties(d);
		}
	}

}
```
Its all for our view proxy, Now the modified library class:

SpinKitView.java
----------------
```java
package com.github.ybq.android.spinkit;

import android.content.Context;
import android.graphics.drawable.Drawable;
import android.view.View;
import android.widget.ProgressBar;

import com.github.ybq.android.spinkit.sprite.Sprite;

public class SpinKitView extends ProgressBar {
	private static final String LCAT = "SpinView";
	private Style mStyle;
	private int mColor;
	private Sprite mSprite;

	public SpinKitView(Context context) {
		super(context);
		setIndeterminate(true);
	}
```
This is our new new function, maybe it is possible to solve the problem with inheritage …
```java
	public void initView(int color, int style) {
		mColor = color;
		mStyle = Style.values()[style];
		Sprite sprite = SpriteFactory.create(mStyle);
		setIndeterminateDrawable(sprite);
	}

	@Override
	public void setIndeterminateDrawable(Drawable d) {
		if (d instanceof Sprite) {
			setIndeterminateDrawable((Sprite) d);
		}
	}

	public void setIndeterminateDrawable(Sprite d) {
		super.setIndeterminateDrawable(d);
		mSprite = d;
		if (mSprite.getColor() == 0) {
			mSprite.setColor(mColor);
		}
		onSizeChanged(getWidth(), getHeight(), getWidth(), getHeight());
		if (getVisibility() == VISIBLE) {
			mSprite.start();
		}
	}

	@Override
	public Sprite getIndeterminateDrawable() {
		return mSprite;
	}

	@Override
	public void unscheduleDrawable(Drawable who) {
		super.unscheduleDrawable(who);
		if (who instanceof Sprite) {
			((Sprite) who).stop();
		}
	}

	@Override
	public void onWindowFocusChanged(boolean hasWindowFocus) {
		super.onWindowFocusChanged(hasWindowFocus);
		if (hasWindowFocus) {
			if (mSprite != null && getVisibility() == VISIBLE) {
				mSprite.start();
			}
		}
	}

	@Override
	public void onScreenStateChanged(int screenState) {
		super.onScreenStateChanged(screenState);
		if (screenState == View.SCREEN_STATE_OFF) {
			if (mSprite != null) {
				mSprite.stop();
			}
		}
	}

}

```


