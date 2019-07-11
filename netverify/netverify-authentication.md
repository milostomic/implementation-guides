![Jumio](/images/authentication.png)

# Authentication for Web (Beta)

This is a reference manual and configuration guide for Jumio Authentication for Web. It describes how to initiate a transaction, how to customize your settings and branding, and how to display Authentication to your users.
<br><br>
Biometric-based Jumio Authentication establishes the digital identities of your users through the simple act of taking a selfie. Advanced 3D face map technology quickly and securely authenticates users and unlocks their digital identities.
<br>
### Revision history

Information about changes to features and improvements documented in each release is available in our [Revision history](/netverify/README.md).
<br>

## Table of contents

- [Initiating an Authentication transaction](#initiating-an-authentication-transaction)
	- [Authentication and encryption](#authentication-and-encryption)
	- [Request headers](#request-headers)
	- [Request body](#request-body)
		- [Supported locale values](#supported-locale-values)
	- [Response](#response)
	- [Examples](#examples)
		- [Sample request](#sample-request)
		- [Sample response](#sample-response)
- [Configuring settings in the Customer Portal](#configuring-settings-in-the-customer-portal)
	- [Application settings - General](#application-settings--general)
		- [Callback, Error, and Success URLs](#callback-error-and-success-urls)
		- [Authorization token lifetime](#authorization-token-lifetime)
	- [Application settings - Redirect](#application-settings--redirect)
		- [Domain name prefix](#domain-name-prefix)
		- [Default locale](#default-locale)
	- [Customize client](#customize-client)
		- [Colors](#colors)
- [Displaying Authentication](#displaying-authentication)
	- [Using Authentication in an iFrame](#using-authentication-in-an-iframe)
		- [Width and height](#width-and-height)
		- [Example HTML](#example-html)
		- [Optional iFrame Logging](#optional-iframe-logging)
			- [Example iFrame logging code](#example-iframe-logging-code)
- [After the user journey](#after-the-user-journey)
	- [Sample success redirect](#sample-success-redirect)
	- [Sample error redirect](#sample-error-redirect)
- [Supported browsers](#supported-browsers) 		



<br>

---
# Initiating an Authentication transaction

Call the RESTful API POST endpoint **/initiate** with a JSON object containing the properties described below to create a transaction for each user. You will receive a JSON object in the response containing a timestamp, Jumio transaction reference, and a URL which you can use to present Authentication to your user.

**HTTP Request Method:** `POST`<br>
**REST URL (US)**: `https://netverify.com/api/auth/v1/initiate`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/auth/v1/initiate`<br>

<br>

## Authentication and encryption
Authentication API calls are protected using [HTTP Basic Authentication](https://tools.ietf.org/html/rfc7617). Your Basic Auth credentials are constructed using your API token as the user-id and your API secret as the password. You can view and manage your API token and secret in the Customer Portal under **Settings > API credentials**.
<br>

|⚠️ Never share your API token, API secret, or Basic Auth credentials with *anyone* — not even Jumio Support.
|:----------|

The [TLS Protocol](https://tools.ietf.org/html/rfc5246) is required to securely transmit your data, and we strongly recommend using the latest version. For information on cipher suites supported by Jumio during the TLS handshake see [Supported cipher suites](/netverify/supported-cipher-suites.md).

<br>

## Request headers

The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Content-Type: application/json`<br>
`Content-Length:`  (see [RFC-7230](https://tools.ietf.org/html/rfc7230#section-3.3.2))<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

<br>

## Request body

The body of your **initiate** API request allows you to

- provide your own internal tracking information for the user and transaction.
- indicate where the user should be directed after the user journey.
- select the language to be displayed.

|ℹ️ Values set in your API request will override the corresponding settings configured in the Customer Portal.
|:----------|

<br>

**Required items appear in bold type.**  

|Name               |Type   | Max. length|Description                                                                                                  |
|:---                      |:---    |:---        |:---                                                                                                          |
|**enrollmentTransactionReference**|string |36        |The transaction reference from the onboarding ID verification to be used for authentication.<br><br>ℹ️ Transaction has to be<br> •	verificationStatus = APPROVED_VERIFIED<br> •	similarity = MATCH<br> •	validity = TRUE 		|
|**customerInternalReference**<sup>1</sup>|string |100        |Your internal reference for the transaction.		|
|**callbackUrl**<sup>2</sup>              |string |255        |Sends verification result to this URL upon completion.<br>Overrides [Callback URL](#callback-error-and-success-urls) in the Customer Portal.		|
|successUrl<sup>2</sup>               |string |2047        |Redirects to this URL after a successful transaction.<br>Overrides [Success URL](#callback-error-and-success-urls) in the Customer Portal.		|
|errorUrl<sup>2</sup>                 |string |255        |Redirects to this URL after an unsuccessful transaction.<br>Overrides [Error URL](#callback-error-and-success-urls) in the Customer Portal.		|
|tokenLifetimeInMinutes  |number 	|Max. value: 86400 |Time in minutes until the authorization token expires. (minimum: 5, maximum: 86400)<br>Overrides [Authorization token lifetime](#authorization-token-lifetime) in the Customer Portal.		|
|locale                   |string |5          |Renders content in the specified language.<br>Overrides [Default locale](#default-locale) in the Customer Portal.<br>See [supported locale values](#supported-locale-values).		|

<sup>1</sup> Values **must not** contain Personally Identifiable Information (PII) or other sensitive data such as email addresses.<br>
<sup>2</sup> See URL constraints for [Callback, Error, and Success URLs](#callback-error-and-success-urls).

<br>

### Supported `locale` values
Hyphenated combination of [ISO 639-1:2002 alpha-2](https://en.wikipedia.org/wiki/ISO_639-1) language code plus [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) country (where applicable).


|Value  |Locale|
|:--------------|:--------------|
|bg|Bulgarian|
|cs|Czech|
|da|Danish|
|de|German|
|el|Greek|
|en|American English (**default**)|
|en-GB|British English|
|es|Spanish|
|es-MX|Mexican Spanish|
|et|Estonian|
|fi|Finnish|
|fr|French|
|hu|Hungarian|
|it|Italian|
|ja|Japanese|
|ko|Korean|
|lt|Lithuanian|
|nl|Dutch|
|no|Norwegian|
|pl|Polish|
|pt|Portuguese|
|pt-BR|Brazilian Portuguese|
|ro|Romanian|
|ru|Russian|
|sk|Slovak|
|sv|Swedish|
|tr|Turkish|
|vi|Vietnamese|
|zh-CN|Simplified Chinese|
|zh-HK|Traditional Chinese|

---
## Response
Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:----|:----|:----|:----|
|**timestamp**|String|24|Timestamp (UTC) of the response.<br>Format: *YYYY-MM-DDThh:mm:ss.SSSZ*|
|**redirectUrl**|String|255|URL used to load Authentication client.|
|**transactionReference**|String|36|Jumio reference number for the Authentication transaction.|

---
## Examples
### Sample request

~~~
POST https://netverify.com/api/v4/initiate/ HTTP/1.1
Accept: application/json
Content-Type: application/json
Content-Length: 1234
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
{
	"enrollmentTransactionReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
	"customerInternalReference": "transaction_1234",
	"callbackUrl": "https://www.yourcompany.com/callback",
	"successUrl" : "https://www.yourcompany.com/success",
	"errorUrl" : "https://www.yourcompany.com/error",
	"tokenLifetimeInMinutes": 5,
	"locale": "de"
}

~~~

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

### Sample response

~~~
{
  "timestamp": "2019-07-12T08:23:12.494Z",
  "transactionReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "redirectUrl": "https://yourcompany.netverify.com/web/v4/app?authorizationToken=xxxxxxxxxxxxxxlocale=en-GB"
}
~~~

<br>

---
# Configuring settings in the Customer Portal

In the **Settings** screen of the Customer Portal you can customize your settings and brand your Authentication page. <br>Save changes using your Customer Portal password to activate them.
<br>

|ℹ️ Values set in your API request will override the corresponding settings configured in the Customer Portal.
|:----------|

<br>

## Application settings — General

### Callback, Error, and Success URLs

Define a **Callback URL** to receive verification results and extracted user data from Jumio when a transaction is completed. For more information, see our [Callback](/netverify/callback.md) documentation.

Define a **Success URL** to direct the user after images are accepted for processing. If no **Success URL** is specified in the Customer Portal or the **initiate** API request, the Jumio default success page will be displayed, including any custom [images](#images) you have specified.

Define an **Error URL** to direct the user when the verification process ends with an error or a failure after 3 submission attempts. If no **Error URL** is specified in the Customer Portal or the **initiate** API request, the Jumio default error page will be displayed, including any custom [images](#images) you have specified.



#### URL requirements:

* HTTPS using the [TLS Protocol](https://tools.ietf.org/html/rfc5246) (most recent version recommended)
* Valid [URL](https://tools.ietf.org/html/rfc3986) using [ASCII characters](https://en.wikipedia.org/wiki/ASCII) or [IDNA Punycode](https://tools.ietf.org/html/rfc3492)

#### URL restrictions:

* IP addresses, ports, certain query parameters and fragment identifiers are not allowed.
* Personally identifiable information (PII) is not allowed in any form.

Jumio appends the following parameters to your Success or Error URL to redirect your user at the conclusion of the user journey. These cannot be used as part of your Success or Error URL:

|Name|Description|
|:---|:---|
|transactionStatus| • `SUCCESS` for successful submissions. <br> • `ERROR`for errors and failure after 3 attempts.|
|customerInternalReference<sup>1</sup>|Your internal reference for the transaction.|
|transactionReference|Jumio reference number for the transaction.|
|errorCode|Displayed when `transactionStatus` is `ERROR`.|

<sup>1</sup> Values **must not** contain Personally Identifiable Information (PII) or other sensitive data such as email addresses.
<br>

### Authorization token lifetime

Specify the duration of time for which your `redirectUrl` will remain valid. Enter the value in minutes (minimum 5, maximum 86400). The default value is 30 minutes.

<br>

## Application settings — Redirect

### Domain name prefix

You can optionally define a domain name prefix (`https://yourcompany.netverify.com`) for the URL of your Authentication page.


- Allowed characters are letters `a-z`, numbers `0-9`, `-`
- Must not start or end with `-`
- Max. 63 characters

<br>

### Default locale

Select a language from the dropdown list to set your display language for Authentication. If no language is selected, Authentication will be displayed in **English (US)**.<br>

Choose from:<br>

- English
- English (United Kingdom)
- German
- Turkish
- Finnish
- Norwegian
- Polish
- Swedish
- Russian
- Portuguese
- Portuguese (Brazil)
- Spanish
- Spanish (Mexico)
- Italian
- French
- Dutch
- Bulgarian
- Chinese (China)
- Chinese (Hong Kong)
- Czech
- Danish
- Greek
- Hungarian
- Japanese
- Korean
- Romanian
- Slovak
- Vietnamese
- Lithuanian
- Estonian

---

## Customize client

### Colors

Specify primary and secondary colors for each locale to give Authentication your own look and feel.

Any locale which is not configured will first default to the root language (e.g. EN\_GB to EN), then to your default configuration, and finally to the Jumio default.

You can also reset all colors to the Jumio default.

---

<br>

# Displaying Authentication

The **redirectUrl** returned in the response to your **initate** API call, which loads your customized Authentication page, can be used in several ways:

* within an iFrame on your web page
* as a link on your web page
* as a link shared securely with a user

## Using Authentication in an iFrame
If you want to embed Authentication on a web page, place the iFrame tag in your HTML code where you want the client to appear. Use the `redirectUrl` as value of the `src` attribute.

|⚠️ The `allow="camera;fullscreen" allowfullscreen` attributes must be included to enable the camera for image capture in [supported browsers](#supported-browsers) in full screen mode.
|:----------|

|⚠️ In case you are nesting the iFrame in another iFrame the `allow="camera;fullscreen" allowfullscreen` attribute must be added to every iFrame.
|:----------|

### Width and height
We recommend adhering to the responsive breaking points in the table below.

|Size class |Width|Height|
|:-------|---:|-------:|
|Large|≥ 900 px|≥ 710 px|
|Medium| 640 px|660 px|
|Small|560 px|600 px|
|X-Small|≤ 480 px|≤ 535 px|

Note: When specifying the width and height of your iFrame you may prefer to use percentage values so that the iFrame behaves responsively on your page.

|⚠️The Authentication client itself will responsively fill the iFrame that it is loaded into.
|:----------|


### Example HTML

#### Absolute sizing example
```
<iframe src="https://yourcompany.netverify.com/web/v4/app?locale=en-GB&authorizationToken=xxx" width="930" height="750" allow="camera;fullscreen" allowfullscreen></iframe>
```

#### Responsive sizing example
```
<iframe src="https://yourcompany.netverify.com/web/v4/app?locale=en-GB&authorizationToken=xxx" width="70%" height="80%" allow="camera;fullscreen" allowfullscreen></iframe>
```


<br>



### Optional iFrame logging

When the Authentication client is embedded in an iFrame<sup>1</sup>, it will communicate with the containing page using the JavaScript [`window.postMessage()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) method to send events containing pre-defined data. This allows the containing page to react to events as they occur (e.g., by directing to a new page once the `success` event is received). Events include data that allows the containing page to identify which Authentication transaction triggered the event. Events are generated in a stateless way, so that each event contains general contextual information about the transaction (e.g., transaction reference, authorization token, etc.) in addition to data about the specific event that occurred.

Using JavaScript, the containing page can receive the notification and consume the data it contains by listening for the `message` event on the global `window` object and reacting to it as needed. The data passed by the Authentication client in this notification is represented as JSON in the `data` string property of the listener method's `event` argument. Parsing this JSON string results in an object with the properties described below.

All data is encoded with [UTF-8](https://tools.ietf.org/html/rfc3629).
<br>
<br>
<sup>1</sup> This functionality is not available for instances of Authentication running in a standalone window or tab.<br>

### `event.data` object

**Required items appear in bold type.**  

|Property|Type|Description
|:-------|:---|:----------|
|**authorizationToken**|string|Authorization token, valid for a specified duration.|
|**transactionReference**|string|Jumio reference number for the transaction.|
|**customerInternalReference**<sup>1</sup>|string|Your internal reference for the transaction.|
|**eventType**|integer|Type of event that has occurred.<br>Possible values: <br>• `510` (application state-change)|
|**dateTime**|string|UTC timestamp of the event in the browser.<br>Format: *YYYY-MM-DDThh:mm:ss.SSSZ*|
|**payload**|JSON object|Information specific to the event generated. <br>(see [`event.data.payload` object](#eventdatapayload-object))|

<sup>1</sup> Values **must not** contain Personally Identifiable Information (PII) or other sensitive data such as email addresses.

<br>

### `event.data.payload` object

**Required items appear in bold type.**  

|Name|Type|Description|
|:-------|:---|:----------|
|**value**|string|Possible values:<br>• `loaded` (Authentication loaded in the user's browser.)<br>• `success` (Images were accepted for verification.)<br>• `error` (Verification could not be completed due to an error.)|
|metainfo|JSON object|Additional meta-information for error events. <br>(see [`metainfo` object](#metainfo-object))|

<br>

### `event.data.payload.metainfo` object

**Required items appear in bold type.**

|Property|Type|Description|
|:-------|:---|:----------|
|**code**|integer|[see **errorCode** values](#after-the-user-journey)|
<br>

### Example iFrame logging code
~~~javascript
function receiveMessage(event) {
	var data = window.JSON.parse(event.data);
	console.log('Netverify Web was loaded in an iframe.');
	console.log('auth token:', data.authorizationToken);
	console.log('transaction reference:', data.transactionReference);
	console.log('customer internal reference:', data.customerInternalReference);
	console.log('event type:', data.eventType);
	console.log('date-time:', data.dateTime);
	console.log('event value:', data.payload.value);
	console.log('event metainfo:', data.payload.metainfo);
}
window.addEventListener("message", receiveMessage, false);
~~~

<br>

## After the user journey

At the end of the user journey, the user is directed to your **Success URL** if the images they submitted were accepted for processing. If no **Success URL** has been defined, the Jumio default success page will be displayed, including any custom success [image](#images) you have specified in the Customer Portal.

If acceptable images are not provided after three attempts (see [Reject reasons](/netverify/callback.md#reject-reason)), the user is directed to your **Error URL**. If no **Error URL** has been defined, the Jumio default error page will be displayed, including any custom error [image](#images) you have specified in the Customer Portal.

To display relevant information on your success or error page, you can use the following parameters which we append when redirecting to your `successUrl` or `errorUrl` as HTTP `GET` query string parameters<sup>1</sup>. It is also possible to set `successUrl` and `errorUrl` to the same address, by using the query parameter `transactionStatus`.<br>

**Required items appear in bold type.**

|Name|Description|
|:---|:---|
|**transactionStatus**|Possible values:<br>• `SUCCESS` for successful submissions. <br> • `ERROR`for errors and failure after 3 attempts.|
|**customerInternalReference**<sup>2</sup>|Your internal reference for the transaction.|
|**transactionReference**|Jumio reference number for the transaction.|
|errorCode|Displayed when `transactionStatus` is `ERROR`.<br>Possible values: <br>• `9100` (Error occurred on our server.)<br>• `9200` (Authorization token missing, invalid, or expired.)<br>• `9210` (Session expired after the user journey started.)<br>• `9300` (Error occurred transmitting image to our server.)<br>• `9400` (Error occurred during verification step.)<br>• `9800` (User has no network connection.)<br>• `9801` (Unexpected error occurred in the client.)<br>• `9810` (Problem while communicating with our server.)<br>• `9820` (Camera unavailable.)<br>• `9821` (The Authentication capture process failed after 3 attempts.)<br>• `9836` (No acceptable submission in 3 attempts.)|

<sup>1</sup> Because HTTP `GET` parameters can be manipulated on the client side, they may be used for display purposes only.<br>
<sup>2</sup> Values **must not** contain Personally Identifiable Information (PII) or other sensitive data such as email addresses.

### Sample success redirect

```
https://www.yourcompany.com/success/?transactionStatus=SUCCESS&customerInternalReference=YOUR_REF&transactionReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

### Sample error redirect

```
https://www.yourcompany.com/error/?transactionStatus=ERROR&customerInternalReference=YOUR_REF&transactionReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&errorCode=9820
```


---
## Supported browsers

Jumio offers guaranteed support for Authentication on the following browsers and the latest major version of each operating system.



### Desktop

|Browser|Major version|Operating system |Supports<br>Authentication|
|:---|:---|:---|:---:|
|Google Chrome|current +<br> 1 previous|Windows + Mac|X|
|Mozilla Firefox|current +<br>1 previous|Windows + Mac|X|
|Apple Safari|current|Mac|X|
|Microsoft Internet Explorer|current|Windows| |
|Microsoft Edge|current|Windows|X|

### Mobile

Authentication does not support WebViews.

|Browser name|Major browser version|Operating system |Supports<br>Authentication|
|:---|:---|:---|:---:|
|Google Chrome |current |Android|X|
|Samsung Internet |current |Android|X|
|Apple Safari |current |iOS|X<sup>1</sup>|

<sup>1</sup>Partial support refers to supporting only iPad, not iPhone. Shows an overlay button which can not be disabled.


---
&copy; Jumio Corp. 268 Lambert Avenue, Palo Alto, CA 94306
