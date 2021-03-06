Playnomics PlayRM Android SDK Integration Guide
=============================================
If you're new to PlayRM and/or don't have a PlayRM account and would like to get started using PlayRM please visit   <a href="https://controlpanel.playnomics.com/signup">https://controlpanel.playnomics.com/signup</a> to sign up. Soon after creating an account you will receive a registration confirmation email permitting you access to your PlayRM control panel.

Within the control panel, click the <strong>applications</strong> tab and add your app. Upon doing so, you will receive an <strong>Application ID</strong> and an <strong>API KEY</strong>. These two components will enable you to begin the integration process.

Our integration has been optimized to be as straight forward and user friendly as possible. If you're feeling unsure or would like better understand the order the process before beginning integration, please take a moment to check out the <a href="http://integration.playnomics.com/technical/#integration-getting-started">getting started</a> page. Here you can find an overview of our integration process, and platform specific features, to help you better understand the PlayRM integration process.

Note, SDK is intended for working with Android apps built in Java, if you're using Unity and deploying your app to Android, please refer to the <a target="_blank" href="https://github.com/playnomics/unity-sdk#playnomics-playrm-unity-sdk-integration-guide">PlayRM Unity SDK</a>. This SDK is only currently compatible with the Google's venison of Android and doesn't support open-source Android implementations like the Amazon Kindle.


## Considerations for Cross-Platform Applications

If you want to deploy your app to multiple platforms (eg: Android and the Unity Web player), you'll need to create a separate Playnomics Applications in the control panel. Each application must incorporate a separate `<APPID>` particular to that application. In addition, message frames and their respective creative uploads will be particular to that app in order to ensure that they are sized appropriately - proportionate to your app screen size.

Basic Integration
=================

You can download the SDK by forking this repo or downloading the archived files. Include PlaynomicsAndroidSDK.jar file in the *build* folder. 

Add the JAR to your Android application build path. This example is for Eclipse.

<img src="http://integration.playnomics.com/img/android/eclipse-import.png"/>

Make sure the JAR file is included in your Export.

Add the following permissions to your Android application manifest file if they don't already exist:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

### Interacting with PlayRM in Your Application

All session-related calls are made through a `static` class `PlaynomicsSession` while all messaging-related calls are made through a `static` class `Messaging`. To work with any of these classes, you need to import the appropriate package file:

```java
import com.playnomics.playrm;
```

All public methods, except for messaging specific calls, return an enumeration `APIResult`. The values for this enumeration are:
<table>
    <thead>
        <tr>
            <td>Enumeration</td>
            <td>Description</td>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>SENT</code></td>
            <td>
                The event has been sent to PlayRM.
            </td>
        </tr>
        <tr>
            <td><code>SWITCHED</code></td>
            <td>
                The <code>Activity</code> context has been switched.
            </td>
        </tr>
        <tr>
            <td><code>STOPPED</code></td>
            <td></td>
        </tr>
        <tr>
            <td><code>ALREADY_STARTED</code></td>
            <td>
                You've already started the session. <code>PlaynomicsSession.start</code> was called unnecessarily.
            </td>
        </tr>
        <tr>
            <td><code>ALREADY_SWITCHED</code></td>
            <td>
                You've already switched the <code>Activity</code> context. <code>PlaynomicsSession.switchActivity</code> was called unnecessarily.
            </td>
        </tr>
        <tr>
            <td><code>ALREADY_STOPPED</code></td>
            <td>
                You've already stopped the session. <code>PlaynomicsSession.stop</code> was called unnecessarily.
            </td>
        </tr>
        <tr>
            <td><code>SESSION_RESUMED</code></td>
            <td>
                The session is being resumed. 
            </td>
        </tr>
        <tr>
            <td><code>START_NOT_CALLED</code></td>
            <td>
                You didn't start the session. The SDK won't be able to report any data to the PlayRM RESTful API, until this has been done.
            </td>
        </tr>
        <tr>
            <td><code>NO_INTERNET_PERMISSION</code></td>
            <td>
                The SDK has no permission to connect to the Internet. The SDK won't be able to report any data to the PlayRM RESTful API. 
                Check your manifest file permissions to fix this issue.
            </td>
        </tr>
        <tr>
            <td><code>FAIL_UNKNOWN</code></td>
            <td>
                An unknown exception occurred.
            </td>
        </tr>
    </tbody>
</table>

**You always need to start a session before making any other SDK calls.**

### Starting a User Session

To start collecting behavior data, you need to initialize the PlayRM session. In the first `Activity` of your application, start the PlayRM Session in the `didFinishLaunchingWithOptions` method.

You can either provide a dynamic `<USER-ID>` to identify each user:

```java
APIResult start(Activity activity, Long applicationId,  String userId);
```

or have PlayRM, generate a *best-effort* unique-identifier for the user:

```java
APIResult start(Activity activity, Long applicationId);
```

If you do choose to provide a `<USER-ID>`, this value should be persistent, anonymized, and unique to each user. This is typically discerned dynamically when a user starts the application. Some potential implementations:

* An internal ID (such as a database auto-generated number).
* A hash of the user's email address.

**You cannot use the user's Facebook ID or any personally identifiable information (plain-text email, name, etc) for the `<USER-ID>`.**

You **MUST** make the initialization call before working with any other PlayRM modules. You only need to call this method once.

```java

import android.app.Activity;
import com.playnomics.playrm.*;

public class FirstGameActivity extends Activity {
    //...

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);

        //other init code for your app activity
        
        PlaynomicsSession.setTestMode(true);
        
        const Long appId = <APPID>;
        PlaynomicsSession.start(this, appId);
    }

    //...
}
```

Once started, the SDK will automatically begin collecting basic user information (including geo-location) and engagement data in **test mode** (be sure to switch to [production mode](#switch-sdk-to-production-mode) before deploying your application).

### Handling Multiple Activities

If your app has multiple `Activity` classes, you need to notify PlayRM that the activity context has changed by calling `switchActivity`:

```java
APIResult switchActivity(Activity activity);
```

```java

import android.app.Activity;
import com.playnomics.playrm.*;

public class NextGameActivity extends Activity {
    //...

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);

        //other init code for your app activity
        PlaynomicsSession.switchActivity(this);
    }
    //...
}
```

### Stopping a User Session

When the application is being shutdown, you need to explicitly `stop` the PlayRM Session. `PlaynomicsSession.stop` should only be called in primary, long-running `Activity` of your app. You should call this in the `onDestroy` event handler.

```java


public class FirstGameActivity extends Activity {
    //...
    @Override
    protected void onDestroy(){
        PlaynomicsSession.stop();
    }s
    //...
}

```


Congratulations! You've completed our basic integration. You will now be able to track engagement behaviors (having incorporated the Engagement Module) from the PlayRM dashboard. At this point we recommend that you use our integration validation tool to test your integration of our SDK in order insure that it has been properly incorporated in your app. 


PlayRM is currently operating in test mode. Be sure you switch to [production mode](#switch-sdk-to-production-mode), by implementing the code call outlined in our Basic Integration before deploying your app on the web or in an app store.


# Full Integration


<div class="outline">
<li>
<a href="#full-integration">Full Integration</a>

<ul>
<li><a href="#demographics-and-install-attribution">Demographics and Install Attribution</a></li>
<li>
<a href="#monetization">Monetization</a>

<ul>
<li><a href="#purchases-of-in-app-currency-with-real-currency">Purchases of In-App Currency with Real Currency</a></li>
<li><a href="#purchases-of-items-with-real-currency">Purchases of Items with Real Currency</a></li>
<li><a href="#purchases-of-items-with-premium-currency">Purchases of Items with Premium Currency</a></li>
</ul>
</li>
<li><a href="#custom-event-tracking">Custom Event Tracking</a></li>
<li><a href="#validate-integration">Validate Integration</a></li>
<li><a href="#switch-sdk-to-production-mode">Switch SDK to Production Mode</a></li>
</ul>
</li>
<li>
<a href="#messaging-integration">Messaging Integration</a>
<ul>
<li><a href="#sdk-integration">SDK Integration</a></li>
<li><a href="#using-rich-data-callbacks">Using Rich Data Callbacks</a></li>
</ul>
</li>
<ul>
<li><a href="#support-issues">Support Issues</a></li>
<li><a href="#change-log">Change Log</a></li>
</ul>
</div>


If you're reading this it's likely that you've integrated our SDK and are interested in tailoring PlayRM to suit your particular segmentation needs.

The index on the right provides a holistic overview of the <strong>full integration</strong> process. From it, you can jump to specific points in this document depending on what you're looking to learn and do.

To clarify where you are in the timeline of our integration process, you've completed our basic integration. Doing so will enable you to track engagement behaviors from the PlayRM dashboard (having incorporated the Engagement Module). The following documentation will provide succinct information on how to incorporate additional and more in-depth segmentation functionality by integrating any, or all of the following into your application:


<ul>
<li><strong>User Info Module:</strong> - provides basic user information</li>


<li><strong>Monetization Module:</strong> - tracks various monetization events and transactions</li>


<li><strong>Milestone Module:</strong> - tracks significant user events customized to your app</li>
</ul>


Along with integration instructions for our various modules, you will also find integration information pertaining to messaging frame setup, and push setup within this documentation.


## Demographics and Install Attribution

After the SDK is loaded, the user info module may be called to collect basic demographic and acquisition information. This data is used to segment users based on how/where they were acquired and enables improved targeting with basic demographics, in addition to the behavioral data collected using other events.

Provide each user's information using this call:

```java
APIResult userInfo(UserInfoType type, 
                    String country, 
                    String subdivision, 
                    UserInfoSex sex, 
                    Date birthday,
                    UserInfoSource source, 
                    String sourceCampaign, 
                    Date installTime);
```

If any of the parameters are not available, you should pass `null`.

<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>type</code></td>
            <td>UserInfoType</td>
            <td>There is currently only one option available for this: <strong>update</strong> for userInfo updates.</td>
        </tr>
        <tr>
            <td><code>country</code></td>
            <td>String</td>
            <td>This has been deprecated. Just pass <code>null</code>.</td>
        </tr>
        <tr>
            <td><code>subdivision</code></td>
            <td>String</td>
            <td>This has been deprecated. Just pass <code>null</code>.</td>
        </tr>
        <tr>
            <td><code>sex</code></td>
            <td>UserInfoSex</td>
            <td>
                <ul>
                    <li>Male</li>
                    <li>Female</li>
                    <li>Unknown</li>
                </ul>
            </td>
        </tr>
        <tr>
            <td><code>birthday</code></td>
            <td>Date</td>
            <td>A birthday for the user.</td>
        </tr>
        <tr>
            <td><code>source</code></td>
            <td>String</td>
            <td>
                Source of the user, such as "FacebookAds", "UserReferral", "Playnomics", etc. These are only suggestions; any 16-character or shorter string is acceptable.
            </td>
        </tr>
        <tr>
            <td><code>sourceCampaign</code></td>
            <td>String</td>
            <td>
                Any 16-character or shorter string to help identify specific campaigns.
            </td>
        </tr>
        <tr>
            <td><code>installTime</code></td>
            <td>Date</td>
            <td>
                When the user originally installed the app.
            </td>
        </tr>
    </tbody>
</table>

This method is overloaded, with the ability to use an enum for the source

```objectivec
APIResult userInfo(UserInfoType type, 
                    String country,
                    String subdivision,
                    UserInfoSex sex,
                    Date birthyear,
                    UserInfoSource source, 
                    String sourceCampaign, 
                    Date installTime);
```
<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>source</code></td>
            <td>UserInfoSource</td>
            <td>
                You can use a predefined enumeration of UserInfoSource
                <br/>
                <ul>
                    <li>Adwords</li>
                    <li>DoubleClick</li>
                    <li>YahooAds</li>
                    <li>MSNAds</li>
                    <li>AOLAds</li>
                    <li>Adbrite</li>
                    <li>FacebookAds</li>
                    <li>GoogleSearch</li>
                    <li>YahooSearch</li>
                    <li>BingSearch</li>
                    <li>FacebookSearch</li>
                    <li>Applifier</li>
                    <li>AppStrip</li>
                    <li>VIPGamesNetwork</li>
                    <li>UserReferral</li>
                    <li>InterGame</li>
                    <li>Other</li>
                </ul>
            </td>
        </tr>
    </tbody>
</table>


Since PlayRM uses the application client's IP address to determine geographic location, country and subdivision should be set to `null`.

```java

import android.app.Activity;
import com.playnomics.playrm.PlaynomicsSession;
import com.playnomics.playrm.PlaynomicsConstants;

public class FirstGameActivity extends Activity {
    //...

    @Override
    public void onCreate(Bundle savedInstanceState) {
        
        //after you initialize the application start

        Date installDate = isNewUser ? new Date() : null;

        Date birthDate = new Date("1/1/1980");
        UserInfoSex sex = UserInfoSex.Female;

        Playnomics.userInfo(UserInfoType.update, null, null, sex, birthDate, null, null, installDate);
    }
    
    //...
}
```
## Monetization

PlayRM provides a flexible interface for tracking monetization events. This module should be called every time a user triggers a monetization event.

This event tracks users that have monetized and the amount they have spent in total, *real* currency:
* FBC (Facebook Credits)
* USD (US Dollars)

or an in-app, *virtual* currency.

This outlines how currencies are described in PlayRM Android

<table>
    <thead>
        <tr>
            <th>Currency</th>
            <th>Currency Data Type</th>
            <th>CurrencyCategory</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Real</td>
            <td>
                <strong>CurrencyType</strong>
                <ul>
                    <ul>
                        <li>USD: USD dollars</li>
                        <li>FBC: Facebook Credits</li>
                    </ul>
                </ul>
            </td>
            <td>
                The <code>CurrencyCategory</code> is <strong>Real</strong>
            </td>
        </tr>
        <tr>
            <td>Virutal</td>
            <td>
                An <strong>String</strong> name, 16 characters or less.
            </td>
            <td>
                The <code>CurrencyCategory</code> is <strong>Virtual</strong>
            </td>
        </tr>
    </tbody>
</table>

```java
APIResult transaction(long transactionId, 
                        String itemId,
                        double quantity, 
                        TransactionType type, 
                        String otherUserId,
                        String[] currencyTypes, 
                        double[] currencyValues,
                        CurrencyCategory[] currencyCategories);
```
<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>transactionId</code></td>
            <td>signed long long</td>
            <td>
                A unique identifier for this transaction. If you don't have a transaction ID from payments system, you can generate large random number.
            </td>
        </tr>
        <tr>
            <td><code>itemId</code></td>
            <td>String</td>
            <td>If applicable, an identifier for the item. The identifier should be consistent.</td>
        </tr>
        <tr>
            <td><code>quantity</code></td>
            <td>double</td>
            <td>If applicable, the number of items being purchased.</td>
        </tr>
        <tr>
            <td><code>type<code></td>
            <td>TransactionType</td>
            <td>
                The type of transaction occurring:
                <ul>
                    <li>
                        <strong>BuyItem</strong>: A purchase of virtual item. The <code>quantity</code> is added to the user’s inventory</li>
                    <li>
                        <strong>SellItem</strong>: A sale of a virtual item to another user. The item is removed from the user’s inventory. Note: a sale of an item will result in two events with the same <code>transactionId</code>, one for the sale with type SellItem, and one for the receipt of that sale, with type BuyItem.
                    </li>
                    <li>
                        <strong>ReturnItem</strong>: A return of a virtual item to the store. The item is removed from the user’s inventory
                    </li>
                    <li><strong>BuyService</strong>: A purchase of a service, e.g., VIP membership </li>
                    <li><strong>SellService</strong>: The sale of a service to another user</li>
                    <li><strong>ReturnService</strong>: The return of a service</li>
                    <li>
                        <strong>CurrencyConvert</strong>: A conversion of currency from one form to another, usually in the form of real currency (e.g., US dollars) to virtual currency.  If the type of a transaction is CurrencyConvert, then there should be at least 2 elements in the <code>currencyTypes</code>, <code>currencyValues</code>, and <code>currencyCategories</code> arrays
                    </li>
                    <li>
                        <strong>Initial</strong>: An initial allocation of currency and/or virtual items to a new user
                    </li>
                    <li><strong>Free</strong>: Free currency or item given to a user by the application</li>
                    <li>
                        <strong>Reward</strong>: Currency or virtual item given by the application as a reward for some action by the user
                    </li>
                    <li>
                        <strong>GiftSend</strong>: A virtual item sent from one user to another. Note: a virtual gift should result in two transaction events with the same <code>transactionId</code>, one with the type GiftSend, and another with the type GiftReceive
                    </li>
                    <li>
                        <strong>GiftReceive</strong>: A virtual good received by a user. See note for <strong>GiftSend</strong> type
                     </li>
                </ul>
            </td>
        </tr>
        <tr>
            <td><code>otherUserId</code></td>
            <td>String</td>
            <td>
               If applicable, the other user involved in the transaction. A contextual example is a user sending a gift to another user.
            </td>
        </tr>
        <tr>
            <td><code>currencyTypes</code></td>
            <td>String[]</td>
            <td>
                An array of currencyTypes.
            </td>
        </tr>
        <tr>
            <td><code>currencyValues</code></td>
            <td>double[]</td>
            <td>An array of currency values, type is a double.</td>
        </tr>
        <tr>
            <td><code>currencyCategories</code></td>
            <td>CurrencyCategory[]</td>
            <td>
                An array of currency categories.
            </td>
        </tr>
    </tbody>
</table>

An overload for `transaction` with the `currencyTypes` are <strong>CurrencyType</strong>. This is for transactions in only <strong>real</strong> currencies.

```java
APIResult transaction(long transactionId, 
                        String itemId,
                        double quantity, 
                        TransactionType type, 
                        String otherUserId,
                        CurrencyType[] currencyTypes, 
                        double[] currencyValues,
                        CurrencyCategory[] currencyCategories);
```

An overload for `transaction` involving only a single `currencyType`, `currencyValue`, and `currencyCategory`.

```java
 APIResult transaction(long transactionId, 
                        String itemId,
                        double quantity, 
                        TransactionType type, 
                        String otherUserId,
                        CurrencyType currencyType, 
                        double currencyValue,
                        CurrencyCategory currencyCategory);
```

All arguments are the same as the first method, except that the `currencyType`,`currencyValue`, and `currencyCategory` are singular. The `currencyCategory` is likely <strong>Real</strong>.

An overload for `transaction` involving only a single `currencyType`, `currencyValue`, and `currencyCategory`.

```java
APIResult transaction(long transactionId, 
                        String itemId,
                        double quantity, 
                        TransactionType type, 
                        String otherUserId,
                        String currencyType, 
                        double currencyValue,
                        CurrencyCategory currencyCategory);
```

All arguments are the same as the first method, except that the `currencyType`,`currencyValue`, and `currencyCategory` are singular. The `currencyCategory` is likely <strong>Virtual</strong>.

We highlight three common use-cases below.
* [Purchases of In-App Currency with Real Currency](#purchases-of-in-app-currency-with-real-currency)
* [Purchases of Items with Real Currency](#purchases-of-items-with-real-currency)
* [Purchases of Items with In-App Currency](#purchases-of-items-with-in-app-currency)

### Purchases of In-App Currency with Real Currency

A very common monetization strategy is to incentivize users to purchase premium, in-app currency with real currency. PlayRM treats this like a currency exchange. This is one of the few cases where multiple currencies are used in a transaction (first implementation of `transactionWithId`. `itemId`, `quantity`, and `otherUserId` are left `null`.

```java
//user purchases 500 MonsterBucks for 10 USD

String[] currencyTypes = new String[2];
double[] currencyValues = new double[2];
CurrencyCategory[] currencyCategories = new CurrencyCategory[2];

//notice that we're adding all data about each currency in order

//in-app currency
currencyTypes[0] = "MonsterBucks";
currencyValues[0]= 500;
currencyCategories[0]  = CurrencyCategory.Virtual;

//real currency
currencyTypes[1] = CurrencyType.USD.toString();
currencyValues[1]= -10;
currencyCategories[1]  = CurrencyCategory.Real;

APIResult result = PlaynomicsSession.transaction(transactionId, 
                        null,
                        0, 
                        TransactionType.CurrencyConvert, 
                        null,
                        currencyTypes, 
                        currencyValues,
                        currencyCategories);
```

### Purchases of Items with Real Currency

```java
//user purchases a "Monster Trap" for $.99 USD

String trapItemId = "Monster Trap";
double quantity = 1;
double price = .99;

APIResult result =  PlaynomicsSession.transaction(transactionId, 
                        trapItemId,
                        quantity, 
                        TransactionType.BuyItem, 
                        null,
                        CurrencyType.USD, 
                        price,
                        CurrencyCategory.Real);
```

### Purchases of Items with Premium Currency

This event is used to segment monetized users (and potential future monetizers) by collecting information about how and when they spend their premium currency (an in-app currency that is primarily acquired using a *real* currency). This is one level of information deeper than the previous use-cases.

#### Currency Exchanges

This is a continuation on the first currency exchange example. It showcases how to track each purchase of in-app *attention* currency (non-premium virtual currency) paid for with a *premium*:

```java

//In this hypothetical, Energy is an attention currency that is earned over the lifetime of the app. 
//They can also be purchased with the premium MonsterBucks that the user may have purchased earlier.

//user buys 100 Mana with 10 MonsterBucks

String[] currencyTypes = new String[2];
double[] currencyValues = new double[2];
CurrencyCategory[] currencyCategories = new CurrencyCategory[2];

//notice that we're adding all data about each currency in order
//and that both currencies are virtual

//attention currency data
currencyTypes[0] = "Mana";
currencyValues[0]= 100;
currencyCategories[0]  = CurrencyCategory.Virtual;

//premium currency data
currencyTypes[1] = "MonsterBucks";
currencyValues[1]= -10;
currencyCategories[1]  = CurrencyCategory.Virtual;

APIResult result = PlaynomicsSession.transaction(transactionId, 
                        null,
                        0, 
                        TransactionType.CurrencyConvert, 
                        null,
                        currencyTypes, 
                        currencyValues,
                        currencyCategories);
```
#### Item Purchases

This is a continuation on the first item purchase example, except with premium currency.

```java
//user buys 20 light armor, for 5 MonsterBucks

double itemQuantity = 20;
String itemId = "Light Armor";

String premimumCurrency = "MonsterBucks";
double premiumCost = 5;

APIResult result = PlaynomicsSession.transaction(transactionId, 
                     itemId,
                     itemQuantity, 
                     TransactionType.BuyItem, 
                     null,
                     premimumCurrency, 
                     premiumCost,
                     CurrencyCategory.Virtual);
```

## Custom Event Tracking

Custom Events may be used in a number of ways.  They can be used to track certain key in-app events such as finishing a tutorial or receiving a high score. They may also be used to track other important lifecycle events such as level up, zone unlocked, etc.  PlayRM, by default, supports up to five custom events.  You can then use these custom events to create more targeted custom segments.

Each time a user completes a certain event, track it with this call:

```objectivec
APIResult milestone(long milestoneId, 
                    String milestoneName);
```
<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>milestoneId</code></td>
            <td>long</long>
            <td>
                A unique 64-bit numeric identifier for this custom event occurrence.
            </td>
        </tr>
        <tr>
            <td><code>andName</code></td>
            <td>String</td>
            <td>
                The name of the custom event, which should be "CUSTOMn", where n is 1 through 5.
                The name is case-sensitive.
            </td>
        </tr>
    </tbody>
</table>

Example client-side calls for users completing events, with generated IDs:

```java
Random rand = new Random();

// when custom event CUSTOM1 is completed
long milestoneCustom1Id = rand.nextLong();
PlaynomicsSession.milestone(milestoneCustom1Id, "CUSTOM1");
```
## Validate Integration
After configuring your selected PlayRM modules, you should verify your application's correct integration with the self-check validation service.

Simply visit the self-check page for your application: **`https://controlpanel.playnomics.com/applications/<APPID>`**

You can now see the most recent event data sent by the SDK, with any errors flagged. Visit the <a href="http://integration.playnomics.com/technical/#self-check">self-check validation guide</a> for more information.

We strongly recommend running the self-check validator before deploying your newly integrated application to production.

## Switch SDK to Production Mode
Once you have [validated](#validate-integration) your integration, switch the SDK from **test** to **production** mode by simply calling `PlaynomicsSession.setTestMode(false)` (or by removing/commenting out the call entirely) in the initialization block:

```java

import android.app.Activity;
import com.playnomics.playrm.*;

public class FirstGameActivity extends Activity {
    //...

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        //...
        
        PlaynomicsSession.setTestMode(false);
        
        const Long appId = <APPID>;
        PlaynomicsSession.start(this, appId);
    }

    //...
}
```

If you ever wish to test or troubleshoot your integration later on, simply set the test mode back to `true` and revisit the self-check validation tool for your application:

**`https://controlpanel.playnomics.com/applications/<APPID>`**

Messaging Integration
=====================
This guide assumes you're already familiar with the concept of placements and messaging, and that you have all of the relevant `placements` setup for your application.

If you are new to PlayRM's messaging feature, please refer to <a href="http://integration.playnomics.com" target="_blank">integration documentation</a>.

Once you have all of your placements created with their associated `<PLAYRM-FRAME-ID>`s, you can start the integration process.

## SDK Integration

To work with, the messaging you'll need to import the appropriate packages into the `Activity` where you want to use messaging.

```java
import com.playnomics.playrm.Frame;
import com.playnomics.playrm.Frame.DisplayResult;
import com.playnomics.playrm.Messaging;
import com.playnomics.playrm.PlaynomicsSession;
```

In the `onCreate` event handler of the `Activity`, you'll need to attach it to the Messaging class.

```java
import com.playnomics.playrm.Frame;
import com.playnomics.playrm.Frame.DisplayResult;
import com.playnomics.playrm.Messaging;
import com.playnomics.playrm.PlaynomicsSession;

public class MessagingActivity extends Activity{
    
    void onCreate(){
        //setup or switchActivity called before this step
        Messaging.setup(this);
    }
}
```

Loading placements through the SDK:

```java
Frame initWithFrameID(String frameId, Activity activity);
```
<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>frameId</code></td>
            <td>String</td>
            <td>Unique identifier for the placement, the <code>&lt;PLAYRM-FRAME-ID&gt;</code></td>
        </tr>
        <tr>
            <td><code>context</code></td>
            <td>Activity</td>
            <td>The activity your placement is running in.</td>
        </tr>
    </tbody>
</table>

Optionally, associate an implementation of FrameDelegate to process rich data callbacks. See [Using Rich Data Callbacks](#using-rich-data-callbacks) for more information.

```java
Frame initWithFrameID(String frameId, Activity context, FrameDelegate frameDelegate);
```

<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><code>frameId</code></td>
            <td>string</td>
            <td>Unique identifier for the placement, the <code>&lt;PLAYRM-FRAME-ID&gt;</code></td>
        </tr>
        <tr>
            <td><code>context</code></td>
            <td>Activity</td>
            <td>The activity your placement is running in.</td>
        </tr>
        <tr>
            <td><code>frameDelegate</code></td>
            <td>FrameDelegate</td>
            <td>
                Processes rich data callbacks, see <a href="#using-rich-data-callbacks">Using Rich Data Callbacks</a>
            </td>
        </tr>
    </tbody>
</table>

Placements are loaded asynchronously to keep your application responsive. The `initWithFrameID` call begins the loading process. However, until you call `start` on the placement, the placement will not be drawn in the UI. This gives you control over when a placement will appear. Placements are destroyed when closed.

In the example below, we initialize the placement when the `Activity` is first created and then show it after another event has taken place.

In practice, a placement can be loaded in a variety of ways.

```java
import com.playnomics.playrm.Frame;
import com.playnomics.playrm.Frame.DisplayResult;
import com.playnomics.playrm.Messaging;
import com.playnomics.playrm.PlaynomicsSession;

public class MessagingActivity extends Activity{
    
    private Frame frame;

    @Override
    protected void onCreate(){
        //PlaynomicsSession.start or PlaynomicsSession.switchActivity called before this step
        Messaging.setup(this);

        String frameId = "<PLAYRM-FRAME-ID>"
        this.frame = Messaging.initWithFrameID(frameId, this);
    }
}
```


## Using Rich Data Callbacks

Depending on your configuration, a variety of actions can take place when a placement’s message is pressed or clicked:

* Redirect the user to a web URL in the platform's browser application
* Firing a Rich Data callback in your app
* Or in the simplest case, just close the placement, provided that the **Close Button** has been configured correctly.

Rich Data is a JSON message that you associate with your message creative. When the user presses the message, the PlayRM SDK bubbles-up the associated JSON object to an implementation of the interface, `FrameDelegate` associated with the placement.

```java
public interface FrameDelegate {
    void onClick(org.json.JSONObject jsonData);
}
```

The actual contents of your message can be delayed until the time of the messaging campaign configuration. However, the structure of your message needs to be decided before you can process it in your app. 

**The Rich Data callback will not fire if the Close button is pressed.**

Here are three common use cases for placements and messaging campaigns:

* [App Start Placement](#app-start-placement)
* [Currency Balance Low Placement](#currency-balance-low-placement) - for instance, when the user is running low on premium currency
* [Level Complete Placement](#level-complete-placement)

### App Start Frame

In this use-case, we want to configure a placement that is always shown to users when they start a new session. The message shown to the user may change based on the desired segments:

<table>
    <thead>
        <tr>
            <th >
                Segment
            </th>
            <th>
                Priority
            </th>
            <th>
                Code Callback
            </th>
            <th style="width:250px;">
                Creative
            </th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                At-Risk
            </td>
            <td>1st</td>
            <td>
                In this case, we're worried once-active users are now in danger of leaving the app. We might offer them <strong>50 MonsterBucks</strong> to bring them back.
            </td>
            <td>
                <img src="http://playnomics.com/integration-dev/img/messaging/50-free-monster-bucks.png"/>
            </td>
        </tr>
        <tr>
            <td>
                Lapsed 7 or more days
            </td>
            <td>2nd</td>
            <td>
                In this case, we want to thank the user for coming back and incentivize these lapsed users to continue doing so. We might offer them <strong>10 MonsterBucks</strong> to increase their engagement and loyalty.
            </td>
            <td> 
                <img src="http://playnomics.com/integration-dev/img/messaging/10-free-monster-bucks.png"/>
            </td>
        </tr>
        <tr>
            <td>
                Default - users who don't fall into either segment.
            </td>
            <td>3rd</td>
            <td>
                In this case, we can offer a special item to them for returning to the app.
            </td>
            <td>
                <img src="http://playnomics.com/integration-dev/img/messaging/free-bfb.png"/>
            </td>
        </tr>
    </tbody>
</table>

We want our app to process messages for awarding items to users. We process this data with an implementation of the `FrameDelegate` interface.


```java
public class AwardFrameDelegate implements FrameDelegate {

    public void onClick(org.json.JSONObject data){

        if(!data.isNull("type") && 
            data.getString("type").equals("award")){
            
            if(!dataDict.isNull("award")){

                String item = data.getJSONObject("award").getString("item");
                int quantity = data.getJSONObject("award").getInt("quantity");

                //call your own inventory object
                Inventory.addItem(item, quantity);
            }

        }
    }
}
```

And then attaching this AwardFrameDelegate class to the placement shown in an activity:


```java
public class GameActivity extends Activity{
    @Override
    protected void onCreate(){
        //PlaynomicsSession.start or PlaynomicsSession.switchActivity called before this step
        
        Messaging.setup(this);

        String frameId = "<PLAYRM-FRAME-ID>"

        FrameDelegate frameDelegate = new AwardFrameDelegate();
        this.frame = Messaging.initWithFrameID(frameId, this, frameDelegate);
    }
}
```
The related messages would be configured in the Control Panel to use this callback by placing this in the **Target Data** for each message:

Grant 10 Monster Bucks
```json
{
    "type" : "award",
    "award" : 
    {
        "item" : "MonsterBucks",
        "quantity" : 10
    }
}
```

Grant 50 Monster Bucks
```json
{
    "type" : "award",
    "award" : 
    {
        "item" : "MonsterBucks",
        "quantity" : 50
    }
}
```

Grant Bazooka
```json
{
    "type" : "award",
    "award" :
    {
        "item" : "Bazooka",
        "quantity" : 1
    }
}
```

### Currency Balance Low Placement

An advantage of a *dynamic* placement is that it can be triggered by in-app events. For each in-app event you would configure a separate placement. While segmentation may be helpful in deciding what message you show, it may be sufficient to show the same message to all users.

For example, a user may deplete their premium currency and you want to remind them that they can re-up through your store. In this context, we display the same message to all users.

<table>
    <thead>
        <tr>
            <th>
                Segment
            </th>
            <th>
                Priority
            </th>
            <th>
                Code Callback
            </th>
            <th>
                Creative
            </th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                Default - all users, because this message is intended for anyone using the app.

            </td>
            <td>1st</td>
            <td>
                You notice that the user’s in-app, premium currency drops below a certain threshold, now you can prompt them to re-up with this <strong>message</strong>.
            </td>
            <td>
                <img src="http://playnomics.com/integration-dev/img/messaging/running-out-of-monster-bucks.png"/>
            </td>
        </tr>
    </tbody>
</table>

Related delegate callback code:

```csharp
public class StoreFrameDelegate implements FrameDelegate {

    public void onClick(org.json.JSONObject data){

        if(!data.isNull("type") && 
            data.getString("type").Equals("action")){
            
            if(!data.isNull("actionType") && 
                data.getString("actionType").equals("openStore"))
            {
                //opens the store in our app
                Store.open();
            }
        }
    }
}
```

The Default message would be configured in the Control Panel to use this callback by placing this in the **Target Data** for the message:

```json
{
    "type" : "action",
    "action" : "openStore"
}
```

### Level Complete Placement

In the following example, we wish to generate third-party revenue from users unlikely to monetize by showing them a segmented message after completing a level or challenge:

<table>
    <thead>
        <tr>
            <th>
                Segment
            </th>
            <th>
                Priority
            </th>
            <th>
                Callback Behavior
            </th>
            <th>
                Creative
            </th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                Non-Monetizers, in their 5th day of app usage
            </td>
            <td>1st</td>
            <td>Show them a 3rd party ad, because they are unlikely to monetize.</td>
            <td>
                <img src="http://playnomics.com/integration-dev/img/messaging/third-party-ad.png"/>
            </td>
        </tr>
        <tr>
            <td>
                Default: everyone else
            </td>
            <td>2nd</td>
            <td>
                You simply congratulate them on completing the level and grant them some attention currency, "Mana" for completing the level.
            </td>
            <td>
                <img src="http://playnomics.com/integration-dev/img/messaging/darn-good-job.png"/>
            </td>
        </tr>
    </tbody>
</table>

This is another continuation on the `AwardFrameDelegate`, with some different data. The related messages would be configured in the Control Panel:

* **Non-monetizers, in their 5th day of app usage**, a Target URL: `HTTP URL for Third Party Ad`
* **Default**, Target Data:

```json
{
    "type" : "award",
    "award" :
    {
        "item" : "Mana",
        "quantity" : 20
    }
}
```

Support Issues
==============
If you have any questions or issues, please contact <a href="mailto:support@playnomics.com">support@playnomics.com</a>.

Change Log
==========
#### Version 3.1
* Added support for Messaging with Rich Data Callbacks.

#### Version 3.0.1
* Bug fixes for reporting the device ID
* Performance improvements

#### Version 3
* Support for internal messaging
* Added custom event module

#### Version 2
* First release

View version tags <a href="https://github.com/playnomics/android-sdk/tags">here</a>
