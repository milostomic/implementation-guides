![Jumio](/images/netverify.png)

# Netverify Web v4 Migration Guide

Welcome to the new Netverify web client!

This document is intended as a reference manual for customers migrating from Netverify Web v3 to Netverify Web v4. ([Which version of the Netverify Web client I am using?](https://support.jumio.com/hc/en-us/articles/360007401353))

It provides an overview of the significant changes in the new web client and describes the adjustments that will be need to be made to your implementation to use the new version.

<br>

## Table of contents

- [Initiating a Netverify transaction](#initiating-a-netverify-transaction)
	- [Authentication and encryption](#authentication-and-encryption)
	- [Request headers](#request-headers)
	- [Request body](#request-body)
		- [Fields retained from v3 to v4](#fields-retained-from-v3-to-v4-with-changes-where-applicable)
		- [New optional fields](#new-optional-fields)
		- [Deprecated properties](#deprecated-properties)
	- [Response](#response)
		- [Fields retained from v3 to v4](#fields-retained-from-v3-to-v4-with-changes-where-applicable-1)
		- [Deprecated fields](#deprecated-fields)
	- [Examples](#examples)
		- [Sample NVW4 request](#sample-nvw4-request)
		- [Sample NVW4 response](#sample-nvw4-response)
- [Configuring settings in the Customer Portal](#configuring-settings-in-the-customer-portal)
	- [Application settings - General](#application-settings--general)
	- [Application settings - Redirect](#application-settings--redirect)
		- [Domain name prefix](#domain-name-prefix)
		- [Max attempts per user](#max-attempts-per-user)
		- [Default locale](#default-locale)
		- [Enable additional information field](#enable-additional-information-field)
	- [Customize client](#customize-client)
		- [General (Colors)](#general-colors)
		- [Redirect (Images)](#redirect-images)
- [Displaying Netverify](#displaying-netverify)
	- [Using Netverify in an iFrame](#using-netverify-in-an-iframe)
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
# Initiating a Netverify transaction

The Netverify Web v4 client uses a single API endpoint for all Netverify transactions.

Call the RESTful API POST endpoint **/initiate** with the updated JSON parameters as described below to create a transaction for each user. You will receive a JSON object in the response containing a timestamp, Jumio transaction reference (formerly scan reference), and a URL you can now use to present Netverify to your users in any way you like — embedded in an iFrame, as a link on your website, or shared


###### Netverify Web (Embedded) v3
**HTTP Request Method:** `POST`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/initiateNetverify`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/initiateNetverify`<br>

###### Netverify Web (Redirect) v3
**HTTP Request Method:** `POST`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/initiateNetverifyRedirect`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/initiateNetverifyRedirect`<br>

### NVW4 (to initiate all transactions)
**HTTP Request Method:** `POST`<br>
**REST URL (US)**: `https://netverify.com/api/v4/initiate`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/v4/initiate`<br>

<br>

## Authentication and encryption
No change from v3. See the new [Netverify Web v4 Implementation Guide](/netverify/netverify-web-v4.md) for further information.
<br>

|⚠️ Never share your API token, API secret, or Basic Auth credentials with *anyone* — not even Jumio Support.
|:----------|

## Request headers
No change from v3. See the new [Netverify Web v4 Implementation Guide](/netverify/netverify-web-v4.md) for further information.
<br>


## Request body
With this version of the Netverify web client, we have begun the process of improving consistency across all of our APIs. This will be done gradually, to make the transition as smooth as possible for our existing customers.

Some of the **initiate** API request fields have been retained, but have new names. Some previously mandatory fields are now optional, and some optional fields are now mandatory. We have also deprecated some fields and added a few new ones. Our changes are described below.

|ℹ️ Values set in your API request will override the corresponding settings configured in the Customer Portal.
|:----------|

<br>

### Fields retained from v3 to v4 (with changes where applicable)
Changes to the fields in the **initiate** API request are listed in the tables below.<br>
**Required fields appear in bold type.**  

|NVW3 embedded<br>NVW3 redirect|NVW4             |Type   | Max. length|Changes                                                                                              
|:---|:---                      |:---    |:---        |:---                                                                                                          |
|**merchantIdScanReference**<sup>1</sup>|**customerInternalReference**<sup>1</sup>|string |100        |Field has been renamed.                                                               |
|**customerId**<sup>1</sup><br>(optional for embedded)|**userReference**<sup>1</sup>           |string |100        |Field has been renamed and is mandatory.                                                                  |
|merchantReportingCriteria|reportingCriteria        |string |100        |Field has been renamed.                                                                      |
|**successUrl**<sup>2</sup><br>(optional for redirect)|successUrl<sup>2</sup>               |string |255        |Previously required for embedded, if not set in the Customer Portal. Now optional.<br>Overrides **Success URL** in the Customer Portal.|
|**errorUrl**<sup>2</sup><br>(optional for redirect)|errorUrl<sup>2</sup>                 |string |255        |Previously required for embedded, if not set in the Customer Portal. Now optional.<br>Overrides **Error URL** in the Customer Portal.|
|callbackUrl<sup>2</sup>|callbackUrl<sup>2</sup>              |string |255        |No change.<br>Overrides **Callback URL** in the Customer Portal.|
|locale|locale                   |string |5          |Hyphen replaces underscore in format for extended locale (language + country).<br>Overrides **Default locale** in the Customer Portal.<br>See [supported locale values](#supported-locale-values).|


<sup>1</sup> Values **must not** contain Personally Identifiable Information (PII) or other sensitive data such as email addresses.<br>
<sup>2</sup> See URL constraints for [Callback, Error, and Success URLs](/netverify/#callback-error-and-success-urls).

<br>

#### Supported `locale` values
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
|vl|Vietnamese|
|zh-CN|Simplified Chinese|
|zh-HK|Traditional Chinese|

<br>

### New optional fields
Netverify Web v4 introduces the concept of *acquisition workflows* and *presets* to allow you to customize every transaction based on your needs for each user.

The `workflowId` parameter lets you optionally specify a combination of verification and capture method options for each transaction. If you choose not to use this field, Identity Verification will be performed on all transactions if it is enabled in your account settings, and the **Capture method** you specify in the Customer Portal will apply.

The `presets` parameter lets you optionally specify the country and document type of the transaction in advance, to skip the selection screen and streamline the user journey.

|NVW4 Parameter                |Type   | Max. length|Description
|---|---|---|---|
|workflowId               |integer |3          |Applies this acquisition workflow to the transaction.<br>Overrides **Capture method** in the Customer Portal.<br>See [supported workflowId values](#supported-workflowid-values).|
|presets                  |JSON   |-|Presets country and document to skip selection screen.<br>See [supported presets values](#supported-presets-values).|-             |

<br>

#### Supported `workflowId` values

| ⚠️ Requests with a **workflowId** including Identity Verification will return `400 Bad Request` if it is not enabled.
|:----------|

|Value |Verification type |Capture method |
|:-----|:-----------------|:--------------|
|100   |ID only           |camera + upload|
|101   |ID only           |camera only    |
|102   |ID only           |upload only    |
|200   |ID + Identity     |camera + upload|
|201   |ID + Identity     |camera only    |
|202   |ID + Identity     |upload only    |

<br>

#### Supported `presets` values
If the optional **presets** field is used, all three values must be passed together as a JSON array (see [sample request](#sample-nvw4-request)) for the request to be valid. It is not possible to preset a single value.

**Required fields appear in bold type.**

|Parameter   |Type    |Max. length    |Options    |
|:------------|:-------|:--------------|:--------------|
|**index**|integer|1| &nbsp; must be set to `1`|
|**country**|string|3|Possible values:<br>•	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code <br /> • `XKX` (Kosovo) is also allowed|
|**type**|string|15|Possible values:<br>• `PASSPORT`<br>•	`DRIVING\_LICENSE`<br>•	`ID\_CARD`|

<br>

### Deprecated properties

|Property|Notes|
|:----|:----|
|clientIp|This property has been removed.|
|firstName|This property has been removed.|
|lastName|This property has been removed.|
|country|This property has been removed.|
|usState|This property has been removed.|
|expiry|This property has been removed.|
|number|This property has been removed.|
|dob|This property has been removed.|
|idType|This property has been removed.|
|personalNumber|This property has been removed.|
|authorizationTokenLifetime|This property has been removed.<br>The default authorization token lifetime of 30 minutes can be modified in the Customer Portal.|
|enabledFields|This property has been removed.<br>Mandatory fields and optional fields enabled in the Customer Portal will be extracted by default. <br>Address will be extracted if it is enabled for your account.<br>Identity Verification can now be requested on a per-transaction basis using [workflowId](#new-optional-fields).|
|captureMethod|This property has been replaced by [workflowId](#new-optional-fields).|
|presetCountry|This property has been replaced by [presets](#new-optional-fields).|
|presetIdType|This property has been replaced by [presets](#new-optional-fields).|

---
## Response
Unsuccessful **initiate** requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error.

Successful requests will return HTTP status code `200 OK` along with the NVW4 parameters listed below.

### Fields retained from v3 to v4 (with changes where applicable)
|NVW3 embedded<br>NVW3 redirect|NVW4|Type   | Max. length|Changes                                                                                              
|:---|:---                      |:---    |:---        |:---  
|timestamp|timestamp|String|24|No change.<br>Timestamp (UTC) of the response.<br>Format: *YYYY-MM-DDThh:mm:ss.SSSZ*|
|clientRedirectUrl<br>(redirect only)|redirectUrl|String|255|Field has been renamed.<br>This URL is now used to [display the Netverify Web client](#displaying-netverify) regardless of implementation.|
|jumioIdScanReference|transactionReference|String|36|Field has been renamed.<br>Jumio reference number for the transaction.|

### Deprecated fields

|Parameter|Notes|
|:----|:----|
|authorizationToken|This field has been removed.|


---
## Examples
### Sample NVW4 request

~~~
POST https://netverify.com/api/v4/initiate/ HTTP/1.1
Accept: application/json
Content-Type: application/json
Content-Length: 1234
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
{
  "customerInternalReference" : "transaction_1234",
  "userReference" : "user_1234",
  "successUrl" : "https://www.yourcompany.com/success",
  "errorUrl" : "https://www.yourcompany.com/error",
  "callbackUrl" : "https://www.yourcompany.com/callback",
  "reportingCriteria" : "myReport1234",
  "workflowId" : 200,
  "presets" :
    [
      {
        "index"   : 1,
        "country" : "AUT",
        "type"    : "PASSPORT"
      }
    ],
  "locale" : "en-GB"
}

~~~

|⚠️ Sample requests cannot be run as-is. Replace example data with your own parameter values.
|:----------|
<br>


### Sample NVW4 response

~~~
{
  "timestamp": "2018-07-03T08:23:12.494Z",
  "transactionReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "redirectUrl": "https://yourcompany.netverify.com/web/v4/app?locale=en-GB&authorizationToken=xxx"
}
~~~

<br>

---
# Configuring settings in the Customer Portal

In the **Settings** screen of the Customer Portal you can customize your settings and brand your Netverify page. <br>Save changes using your Customer Portal password to activate them.
<br>

|ℹ️ Values set in your API request will override the corresponding settings configured in the Customer Portal.
|:----------|

<br>

## Application settings — General

No change from v3. See the new [Netverify Web v4 Implementation Guide](/netverify/netverify-web-v4.md) for further information.
<br>

## Application settings — Redirect

### Domain name prefix

No change from v3. See the new [Netverify Web v4 Implementation Guide](/netverify/netverify-web-v4.md) for further information.

### Max attempts per user
This has been deprecated and will be removed.

### Default locale

No change from v3. See the new [Netverify Web v4 Implementation Guide](/netverify/netverify-web-v4.md) for further information.
<br>

### Enable additional information field

This has been deprecated and will be removed.

---

<br>

## Customize client

### General (Colors)

No change from v3. See the new [Netverify Web v4 Implementation Guide](/netverify/netverify-web-v4.md) for further information.
<br>

### Redirect (Images)

Add a **Header image** for each locale to brand your Netverify page.

**Intro image** has been deprecated and will be removed.

Add a **Success image** and **Error image** for each locale to be displayed on the Jumio default success and error pages when you do not specify your own **successUrl** and **errorUrl**.

Any locale which is not configured will first default to the root language (e.g. EN\_GB to EN), then to your default configuration, and finally to the Jumio default.

All images must be formatted as [JPG](https://jpeg.org/jpeg/) or [PNG](https://en.wikipedia.org/wiki/Portable_Network_Graphics) and must not exceed 5 MB.

**Label for customer ID** and **Label for additional information** have been deprecated and will be removed.

---

<br>

# Displaying Netverify

To make things easier, we've simplified displaying the Netverify web client.

The **redirectUrl** returned in the response to your **initate** API call, which loads your customized Netverify page, can be used in several ways:

* within an iFrame on your web page
* as a link on your web page
* as a link shared securely with a user

## Using Netverify in an iFrame
If you want to embed Netverify on a web page, there's no longer a need to run any scripts. Simply place the iFrame tag in your HTML code where you want the client to appear, and use the **redirectUrl** as the value of the `src` attribute. The `allow="camera"` attribute must be included to enable the camera for image capture in [supported browsers](#supported-browsers).

### Width and height
We recommend adhering to the responsive breaking points in the table below. The Netverify client will responsively fill the dimensions of your iFrame.

|Size class |Width|Height|
|:-------|---:|-------:|
|Large|≥ 900 px|≥ 710 px|
|Medium| 640 px|660 px|
|Small|560 px|600 px|
|X-Small|≤ 480 px|≤ 535 px|

### Example HTML  
```
<iframe src="https://yourcompany.netverify.com/web/v4/app?locale=en-GB&authorizationToken=xxxx
xxxxxxx" width="930" height="750" allow="camera"></iframe>
```
<br>

### Optional iFrame logging

We've updated our optional iFrame logging feature to extend its functionality. We've added some new data objects, but the mechanism remains the same.

When the Netverify client is embedded in an iFrame<sup>1</sup>, it will communicate with the containing page using the JavaScript [`window.postMessage()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) method to send **events** containing pre-defined data. This allows the containing page to react to events as they occur (e.g., by directing to a new page once the **success** event is received). Events include data that allows the containing page to identify which Netverify transaction triggered the event. Events are generated in a stateless way, so that each event contains general contextual information about the transaction (e.g., transaction reference, authorization token, etc.) in addition to data about the specific event that took occurred.

Using JavaScript, the containing page can receive the notification and consume the data it contains by listening for the `message` event on the global `window` object and reacting to it as needed. The data passed by the Netverify web client in this notification is represented as JSON in the `data` string property of the listener method's `event` argument. Parsing this JSON string results in an object with the properties described below.

All data is encoded with [UTF-8](https://tools.ietf.org/html/rfc3629).
<br>
<br>
<sup>1</sup> This functionality is not available for instances of Netverify running in a standalone window or tab.<br>

### `event.data` object

**Required fields appear in bold type.**  

|NVW3 |NVW4 |Type|Description
|:---|:-------|:---|:----------|
|**authorizationToken**|**authorizationToken**|string|Authorization token, valid for a specified duration.|
|**scanReference**|**transactionReference**|string|Jumio reference number for the transaction.|
|**timestamp**|**dateTime**|string|UTC timestamp of the event in the browser.<br>Format: *YYYY-MM-DDThh:mm:ss.SSSZ*|
||**customerInternalReference**|string|Your internal reference for the transaction.|
||**eventType**|integer|Type of event that has occurred.<br>Possible values: <br>• `510` (application state-change)|
||**payload**|JSON object|Information specific to the event generated (see [`event.data.payload` object](#eventdatapayload-object))|

<br>

### `event.data.payload` object
This new object gives us the option to notify you when the Netverify web client is loaded, and when the user journey ends with either a success or an error.

**Required fields appear in bold type.**  

|Property|Type|Description|
|:-------|:---|:----------|
|**value**|string|Possible values:<br>• `loaded` (Netverify loaded in the user's browser.)<br>• `success` (Images were accepted for verification.)<br>• `error` (Verification could not be completed due to an error.)|
|metainfo|JSON object|Additional meta-information for error events. <br>(see [`event.data.payload.metainfo` object](#eventdatapayloadmetainfo-object))|

<br>

### `event.data.payload.metainfo` object
When the user journey ends with an error, this new object allows us to give you more information about what happened.

**Required fields appear in bold type.**

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

At the end of the user journey, the user is directed to your **successUrl** if the images they submitted were accepted for processing. If no **successUrl** has been defined, the Jumio default success page will be displayed, including any custom success [image](#redirect-images) you have specified in the Customer Portal.

If acceptable images are not provided after three attempts (see [Reject reasons](/netverify/callback.md#reject-reason)), the user is directed to your **errorUrl**. If no **errorUrl** has been defined, the Jumio default error page will be displayed, including any custom error [image](#redirect-images) you have specified in the Customer Portal.

To display relevant information on your success or error page, you can use the following parameters which we append when redirecting to your **successUrl** or **errorUrl** as HTTP `GET` query string parameters<sup>1</sup>. It is also possible to set your **successUrl** and **errorUrl** to the same address, by using the query parameter `transactionStatus`.<br>

|NVW3|NVW4|Description|
|:---|:---|:---|
|idScanStatus|transactionStatus|Possible values:<br>`SUCCESS` for successful submissions <br> `ERROR` for errors and failure after 3 attempts|
|merchantIdScanReference|customerInternalReference|Your internal reference for the transaction.|
|jumioIdScanReference|transactionReference|Jumio reference number for the transaction.|
|errorCode|errorCode|Appears only when `transactionStatus` is `ERROR`.<br>Possible values: <br>• `9100` (Error occurred on our server.)<br>• `9200` (Authorization token missing, invalid, or expired.)<br>• `9210` (Session expired after the user journey started.)<br>• `9300` (Error occurred transmitting image to our server.)<br>• `9400` (Error occurred during verification step.)<br>• `9800` (User has no network connection.)<br>• `9801` (Unexpected error occurred in the client.)<br>• `9810` (Problem while communicating with our server.)<br>• `9820` (File upload not enabled and camera unavailable.)<br>• `9835` (No acceptable submission in 3 attempts.)|

<sup>1</sup> Because HTTP `GET` parameters can be manipulated on the client side, they may be used for display purposes only.

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

Jumio offers guaranteed support for Netverify on the following browsers and the latest major version of each operating system.



### Desktop

|Browser|Major version|Operating system |Supports<br>image upload |Supports<br>HTML5 video stream |
|:---|:---|:---|:---|:---|
|Google Chrome|current +<br> 1 previous|Windows + Mac|X|X|
|Mozilla Firefox|current +<br>1 previous|Windows + Mac|X|X|
|Apple Safari|current|Mac|X|X<sup>1</sup>|
|Microsoft Internet Explorer|current|Windows|X| |
|Microsoft Edge|current|Windows|X|X|

<sup>1</sup> Does not work inside an iFrame

### Mobile

|Browser name|Major browser version|Operating system |Supports<br>image upload |Supports<br>HTML5 video stream |
|:---|:---|:---|:---|:---|
|Google Chrome |current |Android|X|X|
|Apple Safari |current |iOS|X|X<sup>1</sup>|

<sup>1</sup> Does not work inside an iFrame



---
&copy; Jumio Corp. 268 Lambert Avenue, Palo Alto, CA 94306
