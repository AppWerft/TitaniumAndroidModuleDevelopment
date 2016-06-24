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
