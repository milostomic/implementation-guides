![Jumio](/images/bam_checkout.png)

# BAM Checkout Mobile Web Implementation Guide

This is a reference manual and configuration guide for the BAM Checkout Mobile Web product. It illustrates how to embed a client into your mobile web page, and implement the BAM Checkout Mobile Web API.

## Table of contents

- [Release notes](#release-notes)
- [Embedding BAM Checkout into your page](#embedding-bam-checkout-into-your-page)
- [Global BAM Checkout settings](#global-bam-checkout-settings)
- [Supported environments and credit cards](#supported-environments-and-credit-cards)

## Release notes

**Version**|**Date**|**Description**
:---|:---|:---
1.8.0|2016-01-26|Updated static endpoint URLs
1.7.1|2015-08-11|Added error code 120 "Server connection timed out"
1.7.0|2015-06-30|Introduced EU data center
1.6.0|2015-06-16|Introduced BAM Checkout Mobile Web code samples
1.5.0|2015-05-21|Introduced account number and sort code recognition for card scanner and API
1.4.3|2015-03-24|Added WebView as supported "browser" on iOS
1.4.2|2015-01-27|Changed supported environments from iOS 6 to 7 and from Android 4 to 4.1.2
1.4.1|2014-12-02|Added embed code to show the client after initial page load
1.4.0|2014-11-20|Introduced card type recognition
1.3.0|2014-10-21|Introduced card scanner without manual entry <br>Added warning event and function<br>Renamed parameter "errorCode" to "code" and "errorMessage" to "message"
1.2.1|2014-09-25|Added code samples for authorization token creation
1.2.0|2014-09-03|Introduced BAM Checkout Mobile Web API<br>Added error event<br>Added Android Browser support
1.1.0|2014-08-19|Introduced card holder recognition
1.0.2|2014-07-08|Added two-factor authentication<br>Added embed code for AngularJS applications
1.0.1|2014-06-24|Removed "baseUrl" setting from embed code
1.0.0|2014-06-12|Initial release

## Embedding BAM Checkout into your page

<a name="InitiatingTheTransaction"></a>
### Initiating the transaction

Before BAM Checkout Mobile Web can process a user's credit card, you need to create a transaction-specific authorization token which is valid for 5 minutes.

To create an authorization token, perform these 4 steps on the server-side to not expose your secret encryption and checksum keys.

1. **AES-256** (Advanced Encryption Standard)
  1. Create the **message** string which contains the following data, separated by a semicolon:
      * Time in milliseconds since January 1, 1970 midnight UTC (long integer)
      * Your reference for each scan (max. 100 characters) <br>_Example:_
      ```
      "1397016803796;e68714bc-623f-44bf-b999-0033479a317e"
      ```
  1. Generate a random, 16-bytes array as the **IV** (Initialization Vector). A new, cryptographically strong number needs to be generated for each transaction.
  1. Use **AES-256** to encrypt
      * the **message** (see 1. 1)
      * with your Base64-decoded **encryption key** (see [Global BAM Checkout settings](#GlobalBAMcheckoutSettings) chapter)
      * in mode **CBC** (Cipher Block Chaining)
      * with **PKCS7** padding
      * and the **IV** (see 1. 2).
1. **HMAC** (Hashed Message Authentication Code)
  1. Concatenate the **encrypted message** (see 1. 3) and the **IV** (see 1. 2) into one **byte array**.
  1. Use **HMAC-SHA1** to calculate the **HMAC**
      * from the **byte array** (see 2. 1)
      * with your Base64-decoded **checksum key** (see [Global BAM Checkout settings](#GlobalBAMcheckoutSettings) chapter).
1.  Concatenate the **encrypted message** (see 1. 3), the **IV** (see 1. b) and the **HMAC** (see 2. 2) into one **byte array**.<br>Note: The size of your encrypted message will be n x 16 bytes (= AES block size of 128 bits). The number n depends on the length of your input message string.<br>![BAM Checkout Mobile Web Byte Array](https://www.jumio.com/app/uploads/2016/04/bcmw_bytearray.png "BCMW byte array")
1.  Generate your **authorization token** by **Base64**-encoding the **byte array** (see 3.).<br>Note: Your authorization token must not contain line breaks, which are added by some Base64 encoders after 76 characters.<br> _Example:_
```
"C1ZNpA+rIucsVPMzQgM0i4iGy0aDcahNTVFgndC71o1sCjqb975PyCRcQGBjUyXOGYPFAjUXsIqyW1Dsnj5NqNP1p82ONMKy34dyLGzWG8POR3p1am+fX1fLzvxYHcNdRNoZIw=="
```

**The following code samples are illustrative and are therefore not containing official implementation guidelines.**

##### Java sample (using Java Cryptography Extension framework)

```
/* Note: This sample requires
 * installation of Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files for AES-256
 * BouncyCastleProvider library (https://www.bouncycastle.org/) for PKCS7 padding
 * Apache Commons Codec library (http://commons.apache.org/proper/commons-codec/) for Base64 encoding without line breaks
 */

Provider provider = Security.getProvider(BouncyCastleProvider.PROVIDER_NAME);
if (provider == null) {
  Security.addProvider(new BouncyCastleProvider());
}

byte[] encryptionKey = Base64.decodeBase64("YOURENCRYPTIONKEY");
byte[] checksumKey = Base64.decodeBase64("YOURCHECKSUMKEY");

// Step 1. 1
String message = System.currentTimeMillis() + ";" + "YOURREFERENCE";

// Steps 1. 2 and 1. 3
Cipher cipher = Cipher.getInstance("AES/CBC/PKCS7Padding");
cipher.init(Cipher.ENCRYPT_MODE, new SecretKeySpec(encryptionKey, "AES"));
byte[] iv = cipher.getIV();
byte[] encryptedMessage = cipher.doFinal(message.getBytes(Charset.forName("UTF-8")));

// Step 2
/* Note:
 * Instead of concatenating encryptedMessage and iv into one byte array (step 2. 1)
 * and passing it to hmacSha1.doFinal(), this sample uses hmacSha1.update() separately.
 */

Mac hmacSha1 = Mac.getInstance("HmacSHA1");
hmacSha1.init(new SecretKeySpec(checksumKey, "HmacSHA1"));
hmacSha1.update(encryptedMessage);
hmacSha1.update(iv);
byte[] hmac = hmacSha1.doFinal();

// Step 3
byte[] finalByteArray = new byte[encryptedMessage.length + iv.length + hmac.length];
System.arraycopy(encryptedMessage, 0, finalByteArray, 0, encryptedMessage.length);
System.arraycopy(iv, 0, finalByteArray, encryptedMessage.length, iv.length);
System.arraycopy(hmac, 0, finalByteArray, encryptedMessage.length + iv.length, hmac.length);

// Step 4
String authorizationToken = Base64.encodeBase64String(finalByteArray);
```

##### Java sample (using Bouncy Castle library)

```
/* Note: This sample requires
 * BouncyCastleProvider library (https://www.bouncycastle.org/)
 * Apache Commons Codec library (http://commons.apache.org/proper/commons-codec/) for Base64 encoding without line breaks
 */

byte[] encryptionKey = Base64.decodeBase64("YOURENCRYPTIONKEY");
byte[] checksumKey = Base64.decodeBase64("YOURCHECKSUMKEY");

// Step 1. 1
String message = System.currentTimeMillis() + ";" + "YOURREFERENCE";

// Step 1. 2
BufferedBlockCipher cipher = new PaddedBufferedBlockCipher(new CBCBlockCipher(new AESFastEngine()), new PKCS7Padding());
SecureRandom random = SecureRandom.getInstance("SHA1PRNG");
byte[] iv = new byte[cipher.getBlockSize()];
random.nextBytes(iv);

// Step 1. 3
cipher.init(true, new ParametersWithIV(new KeyParameter(encryptionKey), iv));
byte[] messageByteArray = message.getBytes(Charset.forName("UTF-8"));
byte[] encryptedMessage = new
byte[cipher.getOutputSize(messageByteArray.length)];
int offset = cipher.processBytes(messageByteArray, 0, messageByteArray.length, encryptedMessage, 0);
cipher.doFinal(encryptedMessage, offset);

// Step 2
/* Note:
 * Instead of concatenating encryptedMessage and iv into one hmacByteArray (step 2. a)
 * and passing it to hmacSha1.doFinal(hmacByteArray), this sample uses hmacSha1.update().
 */

Mac hmacSha1 = new HMac(new SHA1Digest());
hmacSha1.init(new KeyParameter(checksumKey));
hmacSha1.update(encryptedMessage, 0, encryptedMessage.length);
hmacSha1.update(iv, 0 , iv.length);
byte[] hmac = new byte[hmacSha1.getMacSize()];
hmacSha1.doFinal(hmac, 0);

// Step 3
byte[] finalByteArray = new byte[encryptedMessage.length + iv.length + hmac.length];
System.arraycopy(encryptedMessage, 0, finalByteArray, 0, encryptedMessage.length);
System.arraycopy(iv, 0, finalByteArray, encryptedMessage.length, iv.length);
System.arraycopy(hmac, 0, finalByteArray, encryptedMessage.length + iv.length, hmac.length);

// Step 4
String authorizationToken = Base64.encodeBase64String(finalByteArray);
```

##### PHP sample

```
$encryptionKey = base64_decode("YOURENCRYPTIONKEY");
$checksumKey = base64_decode("YOURCHECKSUMKEY");

// Step 1. 1
$timestamp = floor(microtime(true)*1000);
$message = $timestamp.";"."YOURREFERENCE";

// Manual implementation of PKCS7 padding, as not provided by PHP's mcrypt functions

$blockSize = mcrypt_get_block_size(MCRYPT_RIJNDAEL_128, MCRYPT_MODE_CBC);
$padding = $blockSize - (strlen($message) % $blockSize);
$message .= str_repeat(chr($padding), $padding);

// Step 1. 2
srand(time());
$ivSize = mcrypt_get_iv_size(MCRYPT_RIJNDAEL_128, MCRYPT_MODE_CBC);
$iv = mcrypt_create_iv($ivSize, MCRYPT_RAND);

// Step 1. 3
$encryptedMessage = mcrypt_encrypt(MCRYPT_RIJNDAEL_128, $encryptionKey, $message, MCRYPT_MODE_CBC, $iv);

// Step 2
$hmac = hash_hmac("sha1", $encryptedMessage.$iv, $checksumKey, true);

// Steps 3 and 4
$authorizationToken = base64_encode($encryptedMessage.$iv.$hmac);
```

##### C\# sample

```
byte[] encryptionKey = Convert.FromBase64String("YOURENCRYPTIONKEY");
byte[] checksumKey = Convert.FromBase64String("YOURCHECKSUMKEY");

// Step 1. 1
DateTime epoch = new DateTime(1970, 1, 1, 0, 0, 0, 0, DateTimeKind.Utc);
long timestamp = (long)(DateTime.UtcNow - epoch).TotalMilliseconds;
string message = timestamp + ";" + "YOURREFERENCE";

// Steps 1. 2 and 1. 3
RijndaelManaged rijndaelManaged = new RijndaelManaged();
rijndaelManaged.Key = encryptionKey;
rijndaelManaged.Mode = CipherMode.CBC;
rijndaelManaged.Padding = PaddingMode.PKCS7;
rijndaelManaged.GenerateIV();
ICryptoTransform encryptor = rijndaelManaged.CreateEncryptor();
byte[] encryptedMessage = encryptor.TransformFinalBlock(System.Text.UTF8Encoding.UTF8.GetBytes(message), 0, message.Length);

// Step 2
byte[] hmacByteArray = new byte[encryptedMessage.Length + rijndaelManaged.IV.Length];
encryptedMessage.CopyTo(hmacByteArray, 0);
rijndaelManaged.IV.CopyTo(hmacByteArray, encryptedMessage.Length);
HMACSHA1 hmacSha1 = new HMACSHA1(checksumKey);
byte[] hmac = hmacSha1.ComputeHash(hmacByteArray);

// Step 3
byte[] finalByteArray = new byte[hmacByteArray.Length + hmac.Length];
hmacByteArray.CopyTo(finalByteArray, 0);
hmac.CopyTo(finalByteArray, hmacByteArray.Length);

// Step 4
string authorizationToken = Convert.ToBase64String(finalByteArray);
```

##### JavaScript sample

**Note:** Must be used on the server-side.

```
/* Note: This sample requires
 * CryptoJS library (https://code.google.com/p/crypto-js/)
 */

var encryptionKey = CryptoJS.enc.Base64.parse('YOURENCRYPTIONKEY');
var checksumKey = CryptoJS.enc.Base64.parse('YOURCHECKSUMKEY');

// Step 1. 1
var message = new Date().getTime() + ';' + "YOURREFERENCE";

// Step 1.2
var iv = CryptoJS.lib.WordArray.random(128/8);

// Step 1.3
var encryptedMessage = CryptoJS.AES.encrypt(message, encryptionKey, {iv: iv});

// Step 2
/* Note:
 * Instead of concatenating encryptedMessage and iv into one byte array (step 2. 1),
 * this sample uses hmacSha1.update() separately.
 */

var hmacSha1 = CryptoJS.algo.HMAC.create(CryptoJS.algo.SHA1, checksumKey);
hmacSha1.update(encryptedMessage.ciphertext);
hmacSha1.update(iv);
var hmac = hmacSha1.finalize();

// Steps 3 and 4
var authorizationToken = CryptoJS.enc.Base64.stringify(encryptedMessage.ciphertext.concat(iv).concat(hmac));
```

### Displaying and configuring your BAM Checkout client

**Step 1:** Copy and paste the following script into your page's `<head>` section right before the `</head>` tag. If your customer account is in the EU data center, use _static-bcmw.lon.jumio.com_ instead of _static-bcmw.jumio.com_.

```
<link rel="stylesheet" type="text/css" href="https://static-bcmw.jumio.com/styles/main.min.css">
<script type="text/javascript" src="https://static-bcmw.jumio.com/js/widget-sdk.js"></script>
```

If you are using the AngularJS framework in your application, use `plain-widget-sdk.js` instead of `widget-sdk.js`.

```
<script type="text/javascript" src="https://static-bcmw.jumio.com/js/plain-widget-sdk.js"></script>
```

**Note:** For a full responsive client, we recommend to paste the following meta tag into your `<head>` section.

```
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

**Step 2:** Specify the attribute `data-ng-app` in your body tag as follows.

```
<body data-ng-app="nshtml5sdk">
```

If you are using the AngularJS framework in your application, add the `nshtml5sdk` Angular module to your Angular app dependencies instead of setting it as the root module.

**Step 3:** To display BAM Checkout, paste

```
<div data-ns-bootstrapper="YOURAUTHORIZATIONTOKEN" data-public-identifier="YOURPUBLICIDENTIFIER"></div>
```

or

```
<div data-ns-card-scanner="YOURAUTHORIZATIONTOKEN" data-public-identifier="YOURPUBLICIDENTIFIER"></div>
```

into your page's `<body>` section where you want the client to appear. If your customer account is in the EU data center, add the tag attribute `data-region="eu"`.

**Note:** `data-ns-card-scanner` allows scanning without manual entry.

If you add the above "div" tag after your initial page has been rendered, you need to initialize BAM Checkout with the following JavaScript code before displaying the client:

```
angular.bootstrap(document, ['nshtml5sdk']);
```

##### Embed code parameters

**Parameter**|**Type**|**Description**
:---|:---|:---
data-ns-bootstrapper<br>data-ns-card-scanner|String|Your generated, transaction-specific authorization token (see [Initiating the transaction](#InitiatingTheTransaction))
data-public-identifier|String|Log into your Jumio customer portal and you can find your public identifier on the "Settings" page under "API credentials".

### Retrieving information

Implement the following event listeners for submit, warning and error notifications by specifying the names of `yourEventListener` and `yourEvent`. The parameter `detail` contains card, warning or error information. Make sure the event listeners are bound before displaying the client.

```
document.addEventListener('ns.submit', function yourEventListener(yourEvent) {
// YOURCODE
// var cardNumber = yourEvent.detail.cardNumber;
// var cardExpiryDate = yourEvent.detail.cardExpiryDate;
// var cardSecurityCode = yourEvent.detail.cardSecurityCode;
// var cardHolderName = yourEvent.detail.cardHolderName;
// var cardType = yourEvent.detail.cardType;
// var cardAccountNumber = yourEvent.detail.cardAccountNumber;
// var cardSortCode = yourEvent.detail.cardSortCode;
});
```

```
document.addEventListener('ns.warning', function yourEventListener(yourEvent) {
// YOURCODE
// var code = yourEvent.detail.code;
// var message = yourEvent.detail.message;
});
```

```
document.addEventListener('ns.error', function yourEventListener(yourEvent) {
// YOURCODE
// var code = yourEvent.detail.code;
// var message = yourEvent.detail.message;
});
```

##### Submit object `detail`

**Parameter**|**Type**|**Max. length**|**Description**
:---|:---|:---|:---
cardNumber|String|16|Full credit card number
cardExpiryDate|String|5|Date card expires in the format MM/YY
cardSecurityCode|String|4|Entered CVV
cardHolderName|String|100|Card holder name
cardType|String| |Possible types: <br> - VISA <br> - MASTERCARD <br> - AMERICAN_EXPRESS <br> - DINERS_CLUB <br> - DISCOVER <br> - CHINA_UNIONPAY <br> - JCB
cardAccountNumber|String|8|For card scanner only: <br> Account number if available
cardSortCode|String|8|For card scanner only: <br> Sort code in the format xx-xx-xx or xxxxxx if available

Using `data-ns-card-scanner`, card details except CVV will be returned if/as readable.

##### Warning object `detail`

**Parameter**|**Type**|**Max. length**|**Description**
:---|:---|:---|:---
code|String|3|See possible codes and messages below
message|String|100|Possible codes and messages: <br>320 - Card not detected<br>330 - Data not extracted

##### Error object `detail`

**Parameter**|**Type**|**Max. length**|**Description**
:---|:---|:---|:---
code|String|3|See possible codes and messages below
message|String|100|Possible codes and messages:<br>110 - Network communication problem<br>120 - Server connection timed out<br>210 - Authentication failed<br>220 - Unsupported SDK version<br>310 - Public identifier missing<br>310 - Authorization token missing

## Using the BAM Checkout Mobile Web API

The BAM Checkout Mobile Web API offers card scanning without a Jumio-hosted user interface. Simply dispatch a custom JavaScript event containing the customer's credit card image and implement functions for successful scans and error notifications.

**Step 1:** Copy and paste the following script into your page's `<head>` section right before the `</head>` tag. If your customer account is in the EU data center, use `static-bcmw.lon.jumio.com` instead of `static-bcmw.jumio.com`.

```
<script type="text/javascript" src="https://static-bcmw.jumio.com/js/widget-sdk.js"></script>
```

If you are using the AngularJS framework in your application, use `plain-widget-sdk.js` instead of `widget-sdk.js`.

```
<script type="text/javascript" src="https://static-bcmw.jumio.com/js/plain-widget-sdk.js"></script>
```

**Step 2:** Initialize the API by adding the following tag in your HTML code. If your customer account is in the EU data center, add the tag attribute `data-region="eu"`.

```
<div id="YOURID" data-ng-app="nshtml5sdk" bc-scan-listener></div>
```

If you are using the AngularJS framework in your application, add the `nshtml5sdk` Angular module to your Angular app dependencies instead of setting it as the root module.

```
<div id="YOURID" bc-scan-listener></div>
```

**Step 3:** Implement functions for successful scans, warning and error notifications.

```
var yourSuccessFunction = function (cardInformation) {
// YOURCODE
// var cardNumber = cardInformation.cardNumber;
// var cardExpiryDate = cardInformation.cardExpiryDate;
// var cardHolderName = cardInformation.cardHolderName;
// var cardType = cardInformation.cardType;
// var cardAccountNumber = cardInformation.cardAccountNumber;
// var cardSortCode = cardInformation.cardSortCode;
};
```

```
var yourWarningFunction = function (warning) {
// YOURCODE
// var code = warning.code;
// var message = warning.message;
};
```

```
var yourErrorFunction = function (error) {
// YOURCODE
// var code = error.code;
// var message = error.message;
};
```

##### Object `cardinformation`

**Parameter**|**Type**|**Max. length**|**Description**
:---|:---|:---|:---
cardNumber|String|16|Full credit card number
cardExpiryDate|String|5|Date card expires in the format MM/YY
cardHolderName|String|100|Card holder name
cardType|String| |Possible types: <br> - VISA <br> - MASTERCARD <br> - AMERICAN_EXPRESS <br> - DINERS_CLUB <br> - DISCOVER <br> - CHINA_UNIONPAY <br> - JCB
cardAccountNumber|String|8|Account number if available
cardSortCode|String|8|Sort code in the format xx-xx-xx or xxxxxx if available

**Note:** Card information parameters will be returned if/as readable.

##### Object `warning`

**Parameter**|**Type**|**Max. length**|**Description**
:---|:---|:---|:---
code|String|3|See possible codes and messages below
message|String|100|Possible codes and messages: <br>320 - Card not detected<br>330 - Data not extracted

##### Object `error`

**Parameter**|**Type**|**Max. length**|**Description**
:---|:---|:---|:---
code|String|3|See possible codes and messages below
message|String|100|Possible codes and messages:<br>110 - Network communication problem<br>120 - Server connection timed out<br>210 - Authentication failed<br>220 - Unsupported SDK version<br>310 - Public identifier missing<br>310 - Authorization token missing

**Step 4:** To submit an image, create an event of type `CustomEvent`,

```
var yourCustomEvent = document.createEvent('CustomEvent');
```

initialize it as follows

```
yourCustomEvent.initCustomEvent('bc.creditCardScan',
    false,
    true,
    {
        publicIdentifier: YOURPUBLICIDENTIFIER,
        authorizationToken: YOURAUTHORIZATIONTOKEN,
        cardImage: YOURCARDIMAGE,
        onSuccess: YOURSUCCESSFUNCTION,
        onWarning: YOURWARNINGFUNCTION,
        onError: YOURERRORFUNCTION
    });
```

and dispatch it to your tag.

```
document.getElementById('YOURID').dispatchEvent(yourCustomEvent);
```

Make sure your tag is loaded (see step 2) and your functions are defined (see step 3) before submitting the image.

##### Event object `detail`

**Parameter**|**Type**|**Description**
:---|:---|:---
publicIdentifier|String|Log into your Jumio customer portal and you can find your public identifier on the "Settings" page under "API credentials".
authorizationToken|String|Your generated, transaction-specific authorization token (see [Initiating the transaction](#InitiatingTheTransaction))
cardImage|HTMLImageElement|Credit card image
onSuccess|Function|Invoked function for successful scans
onWarning|Function|Invoked function for warning notifications
onError|Function|Invoked function for error notifications

<a name="GlobalBAMcheckoutSettings"></a>

## Global BAM Checkout settings

In your Jumio portal settings you can configure the following value.

##### API credentials

Generate, activate and save your keys used for [Initiating the transaction](#InitiatingTheTransaction).

**Note:** Keys are shown Base64-encoded.

## Supported environments and credit cards

BAM Checkout Mobile Web is supported since iOS 7, Android 4.1.2 and Windows Phone 8.1 within the following browsers:

-   iOS: Safari, Chrome and WebView
-   Android: Android Browser, Chrome
-   Windows Phone: Internet Explorer

BAM Checkout supports AMEX, CUP, DINERS, DISCOVER, JCB, MasterCard and VISA.

## Two-factor authentication

If you want to enable two-factor authentication for your Jumio customer portal please contact [support@jumio.com](mailto:support@jumio.com?subject=Two-factor-authentication). Once enabled, users will be guided through the setup upon their first login to obtain a security code using the "Google Authenticator" app.

## BAM Checkout Mobile Web code samples

BAM Checkout Mobile Web Samples for all implementation types are published on the below link. Each code sample contains setup instructions within its README file, and can be downloaded via ZIP file or HTTPS clone URL.

[BAM Checkout Mobile Web Samples](https://github.com/Jumio/BC-Samples/releases)

## Copyright

&copy; Jumio Corp. 268 Lambert Avenue, Palo Alto, CA 94306
