Ti.PayPal
=========

For this module we need a reference (PayPalPayment) of not simple object. For this we begin to dive in proxy world.

After creating of module we copy the jar *PayPalAndroidSDK-2.14.2.jar* into lib. During implementation we will find that this closed source libary uses two other libraries. After downloading and copying iunto libs we have to add these three jars to Build Path as explained in older lectures.

![](https://raw.githubusercontent.com/AppWerft/TitaniumAndroidModuleDevelopment/master/images/d4.jpg)

First we look into the module:

```java

package de.appwerft.paypal;

import java.util.Locale;
import java.util.Currency;
import java.util.ArrayList;

import org.appcelerator.kroll.KrollModule;
import org.appcelerator.titanium.TiProperties;
import org.appcelerator.kroll.KrollDict;
import org.appcelerator.kroll.annotations.Kroll;
import org.appcelerator.titanium.TiApplication;
import org.appcelerator.titanium.util.TiConvert;
import org.appcelerator.kroll.common.Log;

import com.paypal.android.sdk.payments.PayPalConfiguration;


@Kroll.module(name = "Paypal", id = "de.appwerft.paypal")
public class PaypalModule extends KrollModule {
	private static final String LCAT = "PaypalModule";
```
Here we crete some vars for handling inside module:
```java

	private int debugLevel;
	private String clientIdSandbox;
	pivate String clientIdProduction;
	private static String clientId;
	private static int environment;
	rivate static String CONFIG_ENVIRONMENT;
```
With this annotation we can declare constants. These constants are exposed to javascript layer. Now you can use it as parameter in methodes.
```java
	@Kroll.constant
	public static final int ENVIRONMENT_SANDBOX = 0;
	@Kroll.constant
	public static final int ENVIRONMENT_PRODUCTION = 1;
	@Kroll.constant
	public static final int PAYMENT_INTENT_SALE = 0;
	@Kroll.constant
	public static final int PAYMENT_INTENT_AUTHORIZE = 1;
	@Kroll.constant
	public static final int PAYMENT_INTENT_ORDER = 2;
```
The user of the module can  predefine keys in tiapp.xml. For this we need an instance of TiProperties.
```java

	private TiProperties appProperties;

	public PaypalModule() {
		super();
	}
```
First we need a method *initialize()* for importing paramters from function arguments and from tiapp.xml
```java
	@Kroll.method
	public void initialize(@Kroll.argument(optional = true) KrollDict args) {
```
With this pattern we have access to app properties:
```java
		appProperties = TiApplication.getInstance().getAppProperties();
		String environmentString = appProperties.getString(
				"PAYPAL_ENVIRONMENT", "SANDBOX");
		if (environmentString.equals("SANDBOX")) {
			environment = ENVIRONMENT_SANDBOX;
		}
		if (environmentString.equals("PRODUCTION")) {
			environment = ENVIRONMENT_PRODUCTION;
		}
		clientIdSandbox = appProperties.getString("PAYPAL_CLIENT_ID_SANDBOX",
				"");
		clientIdProduction = appProperties.getString(
				"PAYPAL_CLIENT_ID_PRODUCTION", "");

		Log.d(LCAT, "clientIdSandbox after reading of properties="
				+ clientIdSandbox);
```
We have arguments and they are not null:
```java

		if (args != null && args instanceof KrollDict) {
```
Now we test all keys. With Ti.Convert.to* we convert from untyped Js to java. Remark: in Js we should write

```javascript
this.clientIdSandbox
```

On Java the compiler knows it.
```java

			if (args.containsKeyAndNotNull("clientIdSandbox")) {
				clientIdSandbox = TiConvert.toString(args
						.get("clientIdSandbox"));
			}
			if (args.containsKeyAndNotNull("clientIdProduction")) {
				clientIdProduction = TiConvert.toString(args
						.get("clientIdProduction"));
			}
			if (args.containsKeyAndNotNull("environment")) {
				environment = TiConvert.toInt(args.get("environment"));
			}
			if (environment == ENVIRONMENT_SANDBOX) {
				CONFIG_ENVIRONMENT = PayPalConfiguration.ENVIRONMENT_SANDBOX;
				clientId = clientIdSandbox;
			}
			if (environment == ENVIRONMENT_PRODUCTION) {
				CONFIG_ENVIRONMENT = PayPalConfiguration.ENVIRONMENT_PRODUCTION;
				clientId = clientIdProduction;
			}
		}

		Log.d(LCAT, "clientId=" + clientId);
	}

```
In iOS module we can create a configuration object, we can add it to the later call, but for compatibility we have this dummy function:
```java

	@Kroll.method
	public KrollDict createConfiguration(KrollDict args) {
		return args;

	}

	@Kroll.method
	public KrollDict createPaymentItem(KrollDict args) {
		return args;
	}

	@Kroll.method
	public void setDebuglevel(int level) {
		debugLevel = level;
	}
```
This method is a helper to expose all available currencies on system:
```java

	@Kroll.method
	public ArrayList<KrollDict> getAllCurrencySigns() {
		ArrayList<KrollDict> list = new ArrayList<KrollDict>();
		Locale[] locales = Locale.getAvailableLocales();
		for (Locale l : locales) {
			if (null == l.getCountry() || l.getCountry().isEmpty())
				continue;
			Currency c = Currency.getInstance(l);
			KrollDict item = new KrollDict();
			item.put("country", l.getCountry());
			item.put("displayCountry", l.getDisplayCountry());
			item.put("iso3country", l.getISO3Country());
			item.put("symbol", c.getSymbol());
			item.put("displayName", c.getDisplayName());
			list.add(item);
		}
		return list;
	}
}

```



```java
package de.appwerft.paypal;

import java.math.BigDecimal;
import java.net.URI;
import java.net.URISyntaxException;
import java.util.ArrayList;
import java.util.List;

import org.appcelerator.kroll.KrollDict;
import org.appcelerator.kroll.KrollModule;
import org.appcelerator.kroll.KrollProxy;
import org.appcelerator.kroll.annotations.Kroll;
import org.appcelerator.kroll.common.Log;
import org.appcelerator.titanium.TiApplication;
import org.appcelerator.titanium.TiLifecycle.OnActivityResultEvent;
import org.appcelerator.titanium.util.TiConvert;
import org.json.JSONException;

import android.app.Activity;
import android.content.Context;
import android.content.Intent;
import android.net.Uri;

import com.paypal.android.sdk.payments.PayPalAuthorization;
import com.paypal.android.sdk.payments.PayPalConfiguration;
import com.paypal.android.sdk.payments.PayPalFuturePaymentActivity;
import com.paypal.android.sdk.payments.PayPalItem;
import com.paypal.android.sdk.payments.PayPalPayment;
import com.paypal.android.sdk.payments.PayPalPaymentDetails;
import com.paypal.android.sdk.payments.PayPalProfileSharingActivity;
import com.paypal.android.sdk.payments.PayPalService;
import com.paypal.android.sdk.payments.PaymentActivity;
import com.paypal.android.sdk.payments.PaymentConfirmation;

```
After the standard header we see this construct. If the file and the class named *PaymentProxy* (inherited from module name) the this annotation give us the possibility to call
```javascript
var Payment = Module.createPayment();
```
Because the PayPal stuff opens a new activity and we want know the result (OK or Cancel) we have to add the interface *OnActivityResultEvent*. Attention! the arguments from javascript doesn't appears in the constructor of this class. The arguments are arguments of the KrollProxy method *handleCreationDict*.

```java
@Kroll.proxy(creatableInModule = PaypalModule.class)
public class PaymentProxy extends KrollProxy implements OnActivityResultEvent {
	// Standard Debugging variables
	private static final String LCAT = "PaymentProxy";
	String currencyCode;
	String shortDescription;

	String clientId;
	int intentMode;
	boolean futurePayment = false;
	BigDecimal amount, shipping, tax;
	KrollModule proxy; // for event firing
	private static final int REQUEST_CODE_PAYMENT = 1,
			REQUEST_CODE_FUTUREPAYMENT = 2, REQUEST_CODE_PROFILESHARING = 3;
	PayPalConfiguration ppConfiguration = new PayPalConfiguration();
	List<KrollDict> paymentItems = null;

	public PaymentProxy() {
		super();
	}
```
Here we implement the code for result callback. The arguments are self explaining. The code is from example PayPal implementation. Depending of resultcode and payservice different events will fire to the caller object. 
```java
	@Override
	public void onActivityResult(Activity act, int REQUEST_CODE, int resCode,
			Intent data) {
		if (REQUEST_CODE == REQUEST_CODE_PAYMENT) {
			if (resCode == Activity.RESULT_OK) {
				PaymentConfirmation confirm = data
						.getParcelableExtra(PaymentActivity.EXTRA_RESULT_CONFIRMATION);
				if (confirm != null) {
					try {
						if (proxy.hasListeners("paymentDidComplete")) {
							KrollDict event = new KrollDict();
							event.put("success", true);
							event.put("confirm", confirm.toJSONObject()
									.toString(4));
							event.put("payment", confirm.getPayment()
									.toJSONObject());
							proxy.fireEvent("paymentDidComplete", event);
						}

					} catch (JSONException e) {
						Log.e(LCAT, "an extremely unlikely failure occurred: ",
								e);
					}
				}
			} else if (resCode == Activity.RESULT_CANCELED) {
				if (proxy.hasListeners("paymentDidCancel")) {
					KrollDict event = new KrollDict();
					event.put("success", false);
					proxy.fireEvent("paymentDidCancel", event);
				}
			} else if (resCode == PaymentActivity.RESULT_EXTRAS_INVALID) {
			}
		} else if (REQUEST_CODE == REQUEST_CODE_FUTUREPAYMENT) {
			if (resCode == Activity.RESULT_OK) {
				PayPalAuthorization auth = data
						.getParcelableExtra(PayPalFuturePaymentActivity.EXTRA_RESULT_AUTHORIZATION);
				if (auth != null) {
					try {
						Log.i("FuturePaymentExample", auth.toJSONObject()
								.toString(4));

						String authorization_code = auth.getAuthorizationCode();
						Log.i("FuturePaymentExample", authorization_code);

						sendAuthorizationToServer(auth);
						// displayResultText("Future Payment code received from PayPal");

					} catch (JSONException e) {
						Log.e("FuturePaymentExample",
								"an extremely unlikely failure occurred: ", e);
					}
				}
			} else if (resCode == Activity.RESULT_CANCELED) {
				Log.i("FuturePaymentExample", "The user canceled.");
				if (proxy.hasListeners("paymentDidCancel")) {
					KrollDict event = new KrollDict();
					event.put("success", false);
					proxy.fireEvent("paymentDidCancel", event);
				}
			} else if (resCode == PayPalFuturePaymentActivity.RESULT_EXTRAS_INVALID) {
				Log.i("FuturePaymentExample",
						"Probably the attempt to previously start the PayPalService had an invalid PayPalConfiguration. Please see the docs.");
			}
		} else if (REQUEST_CODE == REQUEST_CODE_PROFILESHARING) {
			if (resCode == Activity.RESULT_OK) {
				PayPalAuthorization auth = data
						.getParcelableExtra(PayPalProfileSharingActivity.EXTRA_RESULT_AUTHORIZATION);
				if (auth != null) {
					try {
						Log.i("ProfileSharingExample", auth.toJSONObject()
								.toString(4));

						String authorization_code = auth.getAuthorizationCode();
						Log.i("ProfileSharingExample", authorization_code);

						sendAuthorizationToServer(auth);
						// displayResultText("Profile Sharing code received from PayPal");

					} catch (JSONException e) {
						Log.e("ProfileSharingExample",
								"an extremely unlikely failure occurred: ", e);
					}
				}
			} else if (resCode == Activity.RESULT_CANCELED) {
				if (proxy.hasListeners("paymentDidCancel")) {
					KrollDict event = new KrollDict();
					event.put("success", false);
					proxy.fireEvent("paymentDidCancel", event);
				}
				Log.i("ProfileSharingExample", "The user canceled.");
			} else if (resCode == PayPalFuturePaymentActivity.RESULT_EXTRAS_INVALID) {
				Log.i("ProfileSharingExample",
						"Probably the attempt to previously start the PayPalService had an invalid PayPalConfiguration. Please see the docs.");
			}
		}

	}

```
OK, this was the second step as first. Now we implement the billing layer. All public methods with the annotation *@Kroll.method* will exposed to javascript.
```java
	@Kroll.method
	public void show() {
```
In every cases if the origianl android code needs a context or this, we have to call this boilerplate the get the context:
```java
	
		Context context = TiApplication.getInstance().getApplicationContext();
```
An intent is a component for communication between components like activiteis or services. In this case for calling the billing activity.
```java

		Intent intent = new Intent(context, PaymentActivity.class);
		if (futurePayment == false) {
			PayPalPayment thingsToBuy = null;
			if (intentMode == PaypalModule.PAYMENT_INTENT_SALE) {
				thingsToBuy = getStuffToBuy(PayPalPayment.PAYMENT_INTENT_SALE);
			} else if (intentMode == PaypalModule.PAYMENT_INTENT_AUTHORIZE) {
				thingsToBuy = getStuffToBuy(PayPalPayment.PAYMENT_INTENT_AUTHORIZE);
			} else if (intentMode == PaypalModule.PAYMENT_INTENT_ORDER) {
				thingsToBuy = getStuffToBuy(PayPalPayment.PAYMENT_INTENT_ORDER);
			}

			/* putting configuration */
			intent.putExtra(PayPalService.EXTRA_PAYPAL_CONFIGURATION,
					ppConfiguration);
			/* putting payload */
			intent.putExtra(PaymentActivity.EXTRA_PAYMENT, thingsToBuy);
			/* start the overlay */
			TiApplication.getAppRootOrCurrentActivity().startActivityForResult(
					intent, REQUEST_CODE_PAYMENT);
		} else {
			intent.putExtra(PayPalService.EXTRA_PAYPAL_CONFIGURATION,
					ppConfiguration);
			TiApplication.getAppRootOrCurrentActivity().startActivityForResult(
					intent, REQUEST_CODE_FUTUREPAYMENT);
		}

	}

	@SuppressWarnings("unchecked")
	@Override
	public void handleCreationDict(KrollDict options) {
		super.handleCreationDict(options);
		if (options.containsKeyAndNotNull("intent")) {
			this.intentMode = TiConvert.toInt(options.get("intent"));
		}
		if (options.containsKeyAndNotNull("futurePayment")) {
			this.futurePayment = TiConvert.toBoolean(options
					.get("futurePayment"));
		}
		if (options.containsKeyAndNotNull("currencyCode")) {
			this.currencyCode = TiConvert.toString(options.get("currencyCode"));
		}
		if (options.containsKeyAndNotNull("shortDescription")) {
			this.shortDescription = TiConvert.toString(options
					.get("shortDescription"));
		}
		if (options.containsKeyAndNotNull("amount")) {
			this.amount = new BigDecimal(TiConvert.toString(options
					.get("amount")));
		}
		if (options.containsKeyAndNotNull("tax")) {
			this.tax = new BigDecimal(TiConvert.toString(options.get("tax")));
		}
		if (options.containsKeyAndNotNull("shipping")) {
			this.shipping = new BigDecimal(TiConvert.toString(options
					.get("shipping")));
		}
		if (options.containsKeyAndNotNull("items")) {
			List<KrollDict> paymentItems = new ArrayList<KrollDict>();
			this.paymentItems = (ArrayList<KrollDict>) options.get("items");
			if (!(paymentItems instanceof Object)) {
				throw new IllegalArgumentException("Invalid argument type `"
						+ paymentItems.getClass().getName()
						+ "` passed to consume()");
			}

		}
		if (options.containsKeyAndNotNull("configuration")) {
			KrollDict configurationDict = options.getKrollDict("configuration");
			if (!(configurationDict instanceof KrollDict)) {
				throw new IllegalArgumentException("Invalid argument type `"
						+ configurationDict.getClass().getName()
						+ "` passed to consume()");
			}
			ppConfiguration.environment(PaypalModule.CONFIG_ENVIRONMENT);
			if (configurationDict.containsKeyAndNotNull("merchantName")) {
				ppConfiguration.merchantName(configurationDict
						.getString("merchantName"));
			}
			if (configurationDict
					.containsKeyAndNotNull("merchantPrivacyPolicyURL")) {
				try {
					ppConfiguration.merchantPrivacyPolicyUri(Uri
							.parse(configurationDict
									.getString("merchantPrivacyPolicyURL")));
				} catch (NullPointerException e) {
				}
				try {
					ppConfiguration.merchantUserAgreementUri(Uri
							.parse(configurationDict
									.getString("merchantUserAgreementURL")));
				} catch (NullPointerException e) {
				}
			}
			ppConfiguration.clientId(PaypalModule.clientId);
			Log.d(LCAT, ppConfiguration.toString());
		}
	}

	private PayPalPayment getStuffToBuy(String paymentIntent) {
		if (paymentItems == null) {
			return new PayPalPayment(amount, currencyCode, shortDescription,
					paymentIntent);
		}
		/* iterating thrue all items from KrollDict: */
		PayPalItem[] items = new PayPalItem[this.paymentItems.size()];
		for (int i = 0; i < this.paymentItems.size(); i++) {
			String name = "", sku = "", currency = "EU";
			BigDecimal price = new BigDecimal(0);
			int quantify = 1;
			KrollDict paymentItem = paymentItems.get(i);
			if (paymentItem.containsKeyAndNotNull("name")) {
				name = TiConvert.toString(paymentItem.get("name"));
			}
			if (paymentItem.containsKeyAndNotNull("sku")) {
				sku = TiConvert.toString(paymentItem.get("sku"));
			}
			if (paymentItem.containsKeyAndNotNull("currency")) {
				currency = TiConvert.toString(paymentItem.get("currency"));
			}
			if (paymentItem.containsKeyAndNotNull("quantify")) {
				quantify = TiConvert.toInt(paymentItem.get("quantify"));
			}
			if (paymentItem.containsKeyAndNotNull("price")) {
				price = new BigDecimal(TiConvert.toString(paymentItem
						.get("price")));
			}
			items[i] = new PayPalItem(name, quantify, price, currency, sku);

		}
		BigDecimal subtotal = PayPalItem.getItemTotal(items);
		BigDecimal shipping = this.shipping;
		BigDecimal tax = this.tax;
		PayPalPaymentDetails paymentDetails = new PayPalPaymentDetails(
				shipping, subtotal, tax);
		BigDecimal amount = subtotal.add(shipping).add(tax);
		PayPalPayment payment = new PayPalPayment(amount, this.currencyCode,
				this.shortDescription, paymentIntent);
		payment.items(items).paymentDetails(paymentDetails);
		payment.custom("This is text that will be associated with the payment that the app can use.");
		return payment;
	}

	private void sendAuthorizationToServer(PayPalAuthorization authorization) {
	}

}
```



