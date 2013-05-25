Playnomics PlayRM Android SDK Integration Guide
=============================================
If you're new to PlayRM or don't yet have an account with <a href="http://www.playnomics.com">Playnomics</a>, please take a moment to <a href="http://integration.playnomics.com/technical/#integration-getting-started">get acquainted with PlayRM</a>.

This SDK is intended for working with Android games built in Java, if you're using Unity and deploying your game to Android, please refer to the <a target="_blank" href="https://github.com/playnomics/unity-sdk#playnomics-playrm-unity-sdk-integration-guide">PlayRM Unity SDK</a>. This SDK is only currently compatible with the Google's verison of Android and doesn't support open-source Android implementations like the Amazon Kindle.

Outline
=======
* [Basic Integration](#basic-integration)
    * [Installing the SDK](#installing-the-sdk)
        * [Interacting with PlayRM in Your Game](interacting-with-playrm-in-your-game)
        * [Starting a Player Session](#starting-a-player-session)
        * [Handling Multiple Activities](#handling-multiple-activities)
        * [Stopping a Player Session](#stopping-a-player-session)
    * [Demographics and Install Attribution](#demographics-and-install-attribution)
    * [Monetization](#monetization)
        * [Purchases of In-Game Currency with Real Currency](#purchases-of-in-game-currency-with-real-currency)
        * [Purchases of Items with Real Currency](#purchases-of-items-with-real-currency)
        * [Purchases of Items with Premium Currency](#purchases-of-items-with-premium-currency)
    * [Invitations and Virality](#invitations-and-virality)
    * [Custom Event Tracking](#custom-event-tracking)
* [Messaging Integration](#messaging-integration)
    * [Setting up a Frame](#setting-up-a-frame)
    * [SDK Integration](#sdk-integration)
    * [Using Code Callbacks](#using-code-callbacks)
* [Support Issues](#support-issues)
* [Change Log](#change-log)

Basic Integration
=================

## Installing the SDK
You can download the SDK by forking this repo or downloading the archived files. Include PlaynomicsAndroidSDK.jar file in the *build* folder. 

Add the JAR to your Android application build path. This example is for Eclipse.

<img src="http://integration.playnomics.com/img/android/eclipse-import.png"/>

Make sure the JAR file is included in your Export.

Add the following permissions to your Android application manifest file if they don't already exist:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

### Interacting with PlayRM in Your Game

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

### Starting a Player Session

To start collecting behavior data, you need to initialize the PlayRM session. In the first `Activity` of your game, start the PlayRM Session in the `didFinishLaunchingWithOptions` method.

You can either provide a dynamic `<USER-ID>` to identify each player:

```java
APIResult start(Activity activity, Long applicationId,  String userId);
```

or have PlayRM, generate a *best-effort* unique-identifier for the player:

```java
APIResult start(Activity activity, Long applicationId);
```

If you do choose to provide a `<USER-ID>`, this value should be persistent, anonymized, and unique to each player. This is typically discerned dynamically when a player starts the game. Some potential implementations:

* An internal ID (such as a database auto-generated number).
* A hash of the user’s email address.

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

        //other init code for your game activity
        
        const Long appId = <APPID>;
        PlaynomicsSession.start(this, appId);
    }

    //...
}
```

Once started, the SDK will automatically begin collecting basic user information (including geo-location) and engagement data.

### Handling Multiple Activities

If your game has multiple `Activity` classes, you need to notify PlayRM that the activity context has changed by calling `switchActivity`:

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

        //other init code for your game activity
        PlaynomicsSession.switchActivity(this);
    }
    //...
}
```

### Stopping a Player Session

When the application is being shutdown, you need to explicitally `stop` the PlayRM Session. `PlaynomicsSession.stop` should only be called in primary, long-running `Activity` of your game. You should call this in the `onDestroy` event handler.

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
            <td>A birthday for the player.</td>
        </tr>
        <tr>
            <td><code>source</code></td>
            <td>String</td>
            <td>
                Source of the user, such as "FacebookAds", "UserReferral", "Playnomics", etc. These are only suggestions, any 16-character or shorter string is acceptable.
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
                When the player originally installed the game.
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


Since PlayRM uses the game client’s IP address to determine geographic location, country and subdivision should be set to `null`.

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

PlayRM provides a flexible interface for tracking monetization events. This module should be called every time a player triggers a monetization event.

This event tracks users that have monetized and the amount they have spent in total, *real* currency:
* FBC (Facebook Credits)
* USD (US Dollars)

or an in-game, *virtual* currency.

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
                A unique identifier for this transaction. If you don't have a transaction ID from payments system, you can genenate large random number.
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
                        <strong>BuyItem</strong>: A purchase of virtual item. The <code>quantity</code> is added to the player's inventory</li>
                    <li>
                        <strong>SellItem</strong>: A sale of a virtual item to another player. The item is removed from the player's inventory. Note: a sale of an item will result in two events with the same <code>transactionId</code>, one for the sale with type SellItem, and one for the receipt of that sale, with type BuyItem.
                    </li>
                    <li>
                        <strong>ReturnItem</strong>: A return of a virtual item to the store. The item is removed from the player's inventory
                    </li>
                    <li><strong>BuyService</strong>: A purchase of a service, e.g., VIP membership </li>
                    <li><strong>SellService</strong>: The sale of a service to another player</li>
                    <li><strong>ReturnService</strong>: The return of a service</li>
                    <li>
                        <strong>CurrencyConvert</strong>: A conversion of currency from one form to another, usually in the form of real currency (e.g., US dollars) to virtual currency.  If the type of a transaction is CurrencyConvert, then there should be at least 2 elements in the <code>currencyTypes</code>, <code>currencyValues</code>, and <code>currencyCategories</code> arrays
                    </li>
                    <li>
                        <strong>Initial</strong>: An initial allocation of currency and/or virtual items to a new player
                    </li>
                    <li><strong>Free</strong>: Free currency or item given to a player by the application</li>
                    <li>
                        <strong>Reward</strong>: Currency or virtual item given by the application as a reward for some action by the player
                    </li>
                    <li>
                        <strong>GiftSend</strong>: A virtual item sent from one player to another. Note: a virtual gift should result in two transaction events with the same <code>transactionId</code>, one with the type GiftSend, and another with the type GiftReceive
                    </li>
                    <li>
                        <strong>GiftReceive</strong>: A virtual good received by a player. See note for <strong>GiftSend</strong> type
                     </li>
                </ul>
            </td>
        </tr>
        <tr>
            <td><code>otherUserId</code></td>
            <td>String</td>
            <td>
               If applicable, the other user involved in the transaction. A contextual example is a player sending a gift to another player.
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
* [Purchases of In-Game Currency with Real Currency](#purchases-of-in-game-currency-with-real-currency)
* [Purchases of Items with Real Currency](#purchases-of-items-with-real-currency)
* [Purchases of Items with In-Game Currency](#purchases-of-items-with-in-game-currency)

### Purchases of In-Game Currency with Real Currency

A very common monetization strategy is to incentivize players to purchase premium, in-game currency with real currency. PlayRM treats this like a currency exchange. This is one of the few cases where multiple currencies are used in a transaction (first implementation of `transactionWithId`. `itemId`, `quantity`, and `otherUserId` are left `null`.

```java
//player purchases 500 MonsterBucks for 10 USD

String[] currencyTypes = new String[2];
double[] currencyValues = new double[2];
CurrencyCategory[] currencyCategories = new CurrencyCategory[2];

//notice that we're adding all data about each currency in order

//in-game currency
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
//player purchases a "Monster Trap" for $.99 USD

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

This event is used to segment monetized players (and potential future monetizers) by collecting information about how and when they spend their premium currency (an in-game currency that is primarily acquired using a *real* currency). This is one level of information deeper than the previous use-cases.

#### Currency Exchanges

This is a continuation on the first currency exchange example. It showcases how to track each purchase of in-game *attention* currency (non-premium virtual currency) paid for with a *premium*:

```java

//In this hypothetical, Energy is an attention currency that is earned over the lifetime of the game. 
//They can also be purchased with the premium MonsterBucks that the player may have purchased earlier.

//player buys 100 Mana with 10 MonsterBucks

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
//player buys 20 light armor, for 5 MonsterBucks

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

## Invitations and Virality

The virality module allows you to track a single invitation from one player to another (e.g., inviting friends to join a game).

If multiple requests can be sent at the same time, a separate call should be made for each recipient.

```objectivec
APIResult invitationSent(long invitationId,
                            String recipientUserId, 
                            String recipientAddress, 
                            String method);
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
            <td><code>invitationId</code></td>
            <td>long</td>
            <td>
                A unique 64-bit integer identifier for this invitation. 

                If no identifier is available, this could be a hash/MD5/SHA1 of the sender's and neighbor's IDs concatenated. <strong>The resulting identifier can not be personally identifiable.</strong>
            </td>
        </tr>
        <tr>
            <td><code>recipientUserId</code></td>
            <td>String</td>
            <td>This can be a hash/MD5/SHA1 of the recipient's Facebook ID, their Facebook 3rd Party ID or an internal ID. It cannot be a personally identifiable ID.</td>
        </tr>
        <tr>
            <td><code>recipientAddress</code></td>
            <td>String</td>
            <td>
                An optional way to identify the recipient, for example the <strong>hashed email address</strong>. When using <code>recipientUserId</code> this can be <code>null</code>.
            </td>
        </tr>
        <tr>
            <td><code>method</code></td>
            <td>String</td>
            <td>
                The method of the invitation request will include one of the following:
                <ul>
                    <li>facebookRequest</li>
                    <li>email</li>
                    <li>twitter</li>
                </ul>
            </td>
        </tr>
    </tbody>
</table>

You can then track each invitation response. IMPORTANT: you will need to pass the invitationId through the invitation link.

```objectivec
APIResult invitationResponse(long invitationId,
                                String recipientUserId, 
                                ResponseType response);
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
            <td><code>invitationId</code></td>
            <td>Long</td>
            <td>The ID of the corresponding <code>invitationSent</code> event.</td>
        </tr>
        <tr>
            <td><code>recipientUserId</code></td>
            <td>String</td>
            <td>The <code>recipientUserID</code> used in the corresponding <code>invitationSent</code> event.</td>
        </tr>
        <tr>
            <td><code>responseType</code></td>
            <td>ResponseType</td>
            <td>
                Currently the only response PlayRM tracks acceptance
                <ul>
                    <li>accepted</li>
                </ul>
            </td>
        </tr>
    </tbody>
</table>

Example calls for a player’s invitation and the recipient’s acceptance:

```java
Long invitationId = 112345675;
String recipientUserId = "10000013";

APIResult sentResult = PlaynomicsSession.invitationSentWithId(invitationId , recipientUserId,
                                    null, null);

//later on the recipient accepts the invitation

APIResult responseResult = PlaynomicsSession.invitationResponseWithId(invitationId
                                    recipientUserId,
                                    ResponseType.accepted];
```

## Custom Event Tracking

Milestones may be defined in a number of ways. They may be defined at certain key gameplay points like, finishing a tutorial, or may they refer to other important milestones in a player's lifecycle. PlayRM, by default, supports up to five custom milestones.  Players can be segmented based on when and how many times they have achieved a particular milestone.

Each time a player reaches a milestone, track it with this call:

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
                A unique 64-bit numeric identifier for this milestone occurrence.
            </td>
        </tr>
        <tr>
            <td><code>andName</code></td>
            <td>String</td>
            <td>
                The name of the milestone, which should be "TUTORIAL" or "CUSTOMn", where n is 1 through 5.
                The name is case-sensitive.
            </td>
        </tr>
    </tbody>
</table>

Example client-side calls for a player reaching a milestone, with generated IDs:

```java
Random rand = new Random();

//when the player completes the tutorial
long milestoneTutorialId = rand.nextLong();
PlaynomicsSession.milestone(milestoneTutorialId, "TUTORIAL");

//when milestone CUSTOM2 is reached
long milestoneCustom2Id = rand.nextLong();
PlaynomicsSession.milestone(milestoneCustom2Id, "CUSTOM2");
```

Messaging Integration
=====================
This guide assumes you're already familiar with the concept of frames and messaging, and that you have all of the relevant `frames` setup for your application.

If you are new to PlayRM's messaging feature, please refer to <a href="http://integration.playnomics.com" target="_blank">integration documentation</a>.

Once you have all of your frames created with their associated `<PLAYRM-FRAME-ID>`s, you can start the integration process.

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
import com.playnomics.playrm.PlaynomicsSession;s

public class MessagingActivity extends Activity{
    
    void onCreate(){
        //setup or switchActivity called before this step
        Messaging.setup(this);
    }
}
```

Loading frames through the SDK:

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
            <td>Unique identifier for the frame, the <code><PLAYRM-FRAME-ID></code></td>
        </tr>
        <tr>
            <td><code>frameId</code></td>
            <td>Activity</td>
            <td>The activity your frame is running in.</td>
        </tr>
    </tbody>
</table>

Frames are loaded asynchronously to keep your game responsive. The `initWithFrameID` call begins the frame loading process. However, until you call `start` on the frame, the frame will not be drawn in the UI. This gives you control over when a frame will appear. Frames are destroyed when closed.

In the example below, we initialize the frame when the `Activity` is first created and then show it after another event has taken place.

In practice, a frame can be loaded in a variety of ways.

```java
import com.playnomics.playrm.Frame;
import com.playnomics.playrm.Frame.DisplayResult;
import com.playnomics.playrm.Messaging;
import com.playnomics.playrm.PlaynomicsSession;s

public class MessagingActivity extends Activity{
    
    private Frame frame;

    @Override
    protected void onCreate(){
        //PlaynomicsSession.start or PlaynomicsSession.switchActivity called before this step
        Messaging.setup(this);

        String frameId = "<PLAYRM-FRAME-ID>"
        this.frame = Messaging.initWithFrameID(frameId, this);
        //enable code callbacks
        this.frame.setEnableAdCode(true);
    }
}
```

## Using Code Callbacks

Depending on your configuration, a variety of actions can take place when a frame's message is pressed or clicked:

* Redirect the player to a web URL in Safari
* Fire a code callback in your game
* Or in the simplest case, just close, provided that the **Close Button** has been configured correctly.

All of this setup, takes place at the the time of the messaging campaign configuration. However, all code callbacks need to be configured before PlayRM can interact with them. The SDK uses reflection to find a public method on the `Activity` that you registered your frame with. This method serves as the callback.

* The callback cannot accept any parameters.
* It should return `void`.

**The code callback will not fire if the Close button is pressed.**

Here are three common use cases for frames and a messaging campaigns:

* [Game Start Frame](#game-start-frame)
* [Event Driven Frame - Open the Store](#event-driven-frame-open-the-store) for instance, when the player is running low on premium currency
* [Event Driven Frame - Level Completion](#event-driven-drame-level-completion)

### Game Start Frame

In this use-case, we want to configure a frame that is always shown to players when they start playing a new game. The message shown to the player may change based on the desired segments:

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
                In this case, we're worried once-active players are now in danger of leaving the game. We might offer them <strong>50 MonsterBucks</strong> to bring them back.
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
                In this case, we want to thank the player for coming back and incentivize these lapsed players to continue doing so. We might offer them <strong>10 MonsterBucks</strong> to increase their engagement and loyalty.
            </td>
            <td> 
                <img src="http://playnomics.com/integration-dev/img/messaging/10-free-monster-bucks.png"/>
            </td>
        </tr>
        <tr>
            <td>
                Default - players who don't fall into either segment.
            </td>
            <td>3rd</td>
            <td>
                In this case, we can offer a special item to them for returning to the grame.
            </td>
            <td>
                <img src="http://playnomics.com/integration-dev/img/messaging/free-bfb.png"/>
            </td>
        </tr>
    </tbody>
</table>

```java
import com.playnomics.playrm.Frame;
import com.playnomics.playrm.Frame.DisplayResult;
import com.playnomics.playrm.Messaging;
import com.playnomics.playrm.PlaynomicsSession;s

public class MessagingActivity extends Activity{
    
    private Frame frame;

    @Override
    protected void onCreate(){
        //PlaynomicsSession.start or PlaynomicsSession.switchActivity called before this step
        Messaging.setup(this);

        String frameId = "<PLAYRM-FRAME-ID>"
        this.frame = Messaging.initWithFrameID(frameId, this);
        //enable code callbacks
        this.frame.setEnableAdCode(true);
    }

    public void grant10MonsterBucks(){
        //grant 10 MonsterBucks
    }

    public void grant50MonsterBucks(){
        //grant 50 MonsterBucks
    }

    public void grantBazooka(){
        //grant a bazooka
    }
}
```

The related messages would be configured in the Control Panel to use this callback by placing this in the **Target URL** for each message :

* **At-Risk Message** : `pnx://grant50MonsterBucks`
* **Lapsed 7 or more days** : `pnx://grant10MonsterBucks`
* **Default** : `pnx://grantBazooka`

### Event Driven Frame - Open the Store

An advantage of a *dynamic* frames is that they can be triggered by in-game events. For each in-game event you would configure a separate frame. While segmentation may be helpful in deciding what message you show, it may be sufficient to show the same message to all players.

In particular one event, for examle, a player may deplete their premium currency and you want to remind them that they can re-up through your store. In this context, we display the same message to all players.

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
                Default - all players, because this message is intended for anyone playing the game.
            </td>
            <td>1st</td>
            <td>
                You notice that the player's in-game, premium currency drops below a certain threshold, now you can prompt them to re-up with this <strong>message</strong>.
            </td>
            <td>
                <img src="http://playnomics.com/integration-dev/img/messaging/running-out-of-monster-bucks.png"/>
            </td>
        </tr>
    </tbody>
</table>

```java
import com.playnomics.playrm.Frame;
import com.playnomics.playrm.Frame.DisplayResult;
import com.playnomics.playrm.Messaging;
import com.playnomics.playrm.PlaynomicsSession;s

public class MessagingActivity extends Activity{
    
    private Frame frame;

    @Override
    protected void onCreate(){
        //PlaynomicsSession.start or PlaynomicsSession.switchActivity called before this step
        Messaging.setup(this);

        String frameId = "<PLAYRM-FRAME-ID>"
        this.frame = Messaging.initWithFrameID(frameId, this);
        //enable code callbacks
        this.frame.setEnableAdCode(true);
    }

    public void openStore(){
        //open the store
    }
}
```

The Default message would be configured in the Control Panel to use this callback by placing this in the **Target URL** for the message : `pnx://ClickHandler.openStore`.

### Event Driven Frame - Level Completion

In the following example, we wish to generate third-party revenue from players unlikely to monetize by showing them a segmented message after completing a level or challenge: 

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
                Non-monetizers, in their 5th day of game play
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
                You simply congratulate them on completing the level and grant them some attention currency, "Mana" for completeing the level.
            </td>
            <td>
                <img src="http://playnomics.com/integration-dev/img/messaging/darn-good-job.png"/>
            </td>
        </tr>
    </tbody>
</table>

```java
import com.playnomics.playrm.Frame;
import com.playnomics.playrm.Frame.DisplayResult;
import com.playnomics.playrm.Messaging;
import com.playnomics.playrm.PlaynomicsSession;s

public class MessagingActivity extends Activity{
    
    private Frame frame;

    @Override
    protected void onCreate(){
        //PlaynomicsSession.start or PlaynomicsSession.switchActivity called before this step
        Messaging.setup(this);

        String frameId = "<PLAYRM-FRAME-ID>"
        this.frame = Messaging.initWithFrameID(frameId, this);
        //enable code callbacks
        this.frame.setEnableAdCode(true);
    }

    public void grantMana(){
        //grantMana
    }
}
```

The related messages would be configured in the Control Panel to use this callback by placing this in the **Target URL** for each message :

* **Non-monetizers, in their 5th day of game play** : `HTTP URL for Third Party Ad`
* **Default** : `pnx://grantMana`

Support Issues
==============
If you have any questions or issues, please contact <a href="mailto:support@playnomics.com">support@playnomics.com</a>.

Change Log
==========
#### Version 3
* Support for internal messaging
* Added milestone module

#### Version 2
* First release

View version tags <a href="https://github.com/playnomics/android-sdk/tags">here</a>
