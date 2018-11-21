![Jumio](/images/netverify.png)

# Netverify Web Implementation Guide

This is a reference manual and configuration guide for the Netverify Web product. It illustrates how to embed Netverify into your web page and use Netverify redirect.

<br>

|⚠️ Netverify Web v3 is officially being deprecated and will not be supported after December 18, 2018.
|:----------|

<br>

### Revision history

Information about changes to features and improvements documented in each release is available in our [Revision history](/netverify/README.md).

---

## Table of Contents
- [Embedding Netverify into your page](#embedding-netverify-into-your-page)
    - [Initiating the transaction](#embedded-initiating-the-transaction)
    - [Displaying and configuring your Netverify client](#displaying-and-configuring-your-netverify-client)
    - [Redirecting the customer after the user journey](#embedded-redirecting-the-customer-after-the-user-journey)
- [Using Netverify redirect](#using-netverify-redirect)
    - [Customizing your landing page](#customizing-your-landing-page)
    - [Initiating the transaction](#redirect-initiating-the-transaction)
    - [Calling your landing page](#calling-your-landing-page)
    - [Redirecting the customer after the user journey](#redirect-redirecting-the-customer-after-the-user-journey)
- [Using performNetverify](#using-performnetverify)
- [Global Netverify APIs](#global-netverify-apis)
    - [Supported IDs](#supported-ids)
    - [Accepted IDs](#accepted-ids)
- [Netverify Delete API](#netverify-delete-api)
- [Global Netverify Settings](#global-netverify-settings)
- [Supported Cipher Suites](#supported-cipher-suites)
- [Supported Browsers](#supported-browsers)
- [Test IDs](#test-ids)



---
# Embedding Netverify into your page

## <a name="embedded-initiating-the-transaction"></a>Initiating the Transaction

Call the RESTful HTTP POST API **initiateNetverify** with the below JSON parameters to create a transaction for each user. You will receive a Jumio scan reference and an authorization token, which is valid for a certain amount of time, and to be specified in the embed code parameter "authorizationToken".

**HTTP Request Method:** `POST`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/initiateNetverify`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/initiateNetverify`<br>

**Authentication**:
Netverify API calls are protected using [HTTP Basic Authentication](https://tools.ietf.org/html/rfc7617). Your Basic Auth credentials are constructed using your API token as the user-id and your API secret as the password. You can view and manage your API token and secret in the Customer Portal under **Settings > API credentials**.
<br>

|⚠️ Never share your API token, API secret, or Basic Auth credentials with *anyone* — not even Jumio Support.
|:----------|

The [TLS Protocol](https://tools.ietf.org/html/rfc5246) is required to securely transmit your data, and we strongly recommend using the latest version. For information on cipher suites supported by Jumio during the TLS handshake see [Supported cipher suites](/netverify/supported-cipher-suites.md). <br>Note: Calls with missing or suspicious headers, suspicious parameter values, or without HTTP Basic Authentication will result in HTTP status code 403 Forbidden.

**Header:**
The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Content-Type: application/json`<br>
`Content-Length:`  (see [RFC-7230](https://tools.ietf.org/html/rfc7230#section-3.3.2))<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

<br>

### Request Parameters

|ℹ️ Values set in your API request will override the corresponding settings configured in the Customer Portal.
|:----------|

**Required items appear in bold type.**  


|Parameter    |Type    |Max. length    |Description    |
|:------------|:-------|:--------------|:--------------|
|**merchantIdScanReference**<sup>1</sup>|String|100|Your internal reference for the transaction.|
|**successUrl**<sup>2</sup>|String|2048|Redirects to this URL after a successful transaction.<br><b>Mandatory if not specified in your Jumio portal.</b>|
|**errorUrl**<sup>2</sup>|String|255|Redirects to this URL after a successful transaction.<br><b>Mandatory if not specifed in your Jumio portal.</b>|
|enabledFields|String|100|Defines fields which will be extracted during the ID verification. If a field is not listed in this parameter, it will not be processed for this transaction, regardless of customer portal settings.<br> **Note:** Face match and Address extraction will not be processed unless enabled for your account. If you want to enable them, please contact your Customer Success Manager, or reach out to Jumio Support.<br><br>Possible values:<br>`"idNumber,idFirstName,idLastName,idDob,idExpiry,idUsState,<br>idPersonalNumber,idFaceMatch,idAddress"`|
|authorizationTokenLifetime|Number|Max value:<br>5184000|Time in seconds until the authorization token expires.<br>• Default: 1800 seconds <br>• 0 = 60 days|
|merchantReportingCriteria|String|100|Your reporting criteria for each scan|
|callbackUrl|String|255|Callback URL for the confirmation after the verification is completed (for constraints see [Callback URL](/netverify/portal-settings.md#callback-url))|
|locale|String|100|Locale of the Netverify client:<br>• "bg" Bulgarian<br>• "cs" Czech<br>• "da" Danish<br>• "de" German<br>• "el" Greek<br>• "en" American English (default)<br>• "en\_GB" British English<br>• "es" Spanish<br>• "es\_MX" Spanish Mexico<br>• "et" Estonian<br>• "fi" Finnish<br>• "fr" French<br>• "hu" Hungarian<br>• "it" Italian<br>• "ja" Japanese<br>• "ko" Korean<br>• "lt" Lithuanian<br>• "nl" Dutch<br>• "no" Norwegian<br>• "pl" Polish<br>• "pt" Portuguese<br>• "pt\_BR" Brazilian Portuguese<br>• "ro" Romanian<br>• "ru" Russian<br>• "sk" Slovak<br>• "sv" Swedish<br>• "tr" Turkish<br>• "vi" Vietnamese<br>• "zh\_CN" Chinese (China)<br>• "zh\_HK" Chinese (Hong Kong)|
|clientIp|String|100|IP address of the client|
|customerId<sup>1</sup>|String|100| Your internal reference for the user.|
|firstName|String|100|First name of the customer|
|lastName|String|100|Last name of the customer|
|country|String|3|Possible countries:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>• `XKX` (Kosovo)|
|usState|String|100|Possible values if idType = `PASSPORT` or `ID_CARD`:<br>• Last two characters of [ISO 3166-2: US](http://en.wikipedia.org/wiki/ISO_3166-2:US) state code<br>• [ISO 3166-1](http://en.wikipedia.org/wiki/ISO_3166-1) country name<br>• Kosovo<br>• [ISO 3166-1 alpha-2](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) country code<br>• `XK` (Kosovo)<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>• `XKX` (Kosovo)<br><br>If idType = DRIVING\_LICENSE:<br>• Last two characters of [ISO 3166-2: US state code](http://en.wikipedia.org/wiki/ISO_3166-2:US)|
|expiry|String||Date of expiry in the format *YYYY-MM-DD*|
|number|String|100|Identification number of the document|
|dob|String||Date of birth in the format *YYYY-MM-DD*|
|idType|String|255|Possible values:<br>• `PASSPORT`<br>• `DRIVING_LICENSE`<br>• `ID_CARD`|
|personalNumber|String|14|Personal number of the document|
|captureMethod|String||Capture method for this scan. Possible values: <br>• `CAM`<br>• `UPLOAD`<br>• `ALL`|
|presetCountry|String|3|Preset country and ID type to skip the "Select country and ID type" screen.<br>Note: Both parameters must be present.<br>Possible countries:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>• `XKX` (Kosovo)|
|presetIdType|String|255|Preset country and ID type to skip the "Select country and ID type" screen.<br>Note: Both parameters must be present.<br>Possible ID types: <br>• `PASSPORT`<br>• `DRIVING_LICENSE`<br>• `ID_CARD`|

<sup>1</sup> Values **must not** contain Personally Identifiable Information (PII) or other sensitive data such as email addresses.<br>
<sup>2</sup> See URL constraints for [Success and error URLs](/netverify/portal-settings.md#success-and-error-urls).

#### Supported documents for address extraction

|Country    |ID card    |Driving license    |Passport    |Callback format |
|:------------|:-------|:--------------|:--------------|:-------|
|Australia|No|Yes|No|US|
|Canada|No|Yes|No|US|
|France|Yes|Yes|Yes|Raw|
|Germany|Yes|No|No|EU|
|Ireland|No|Yes|No|Raw|
|Mexico|Yes|No|No|US|
|Romania|Yes|No|No|Raw|
|Singapore|Yes|No|No|Raw|
|Spain|Yes|No|No|EU|
|United Kingdom|No|Yes|No|Raw|
|United States|No|Yes|No|US|

### Response Parameters

|Parameter|Type|Max. length|Description|
|:----|:----|:----|:----|
|timestamp|String||Timestamp (UTC) of the response.<br> Format *YYYY-MM-DDThh:mm:ss.SSSZ*|
|authorizationToken|String|36|Authorization token, which is valid for the authorization token lifetime|
|jumioIdScanReference|String|36|Jumio's reference number for each scan|

### Sample Request

```
POST https://netverify.com/api/netverify/v2/initiateNetverify HTTP/1.1Accept: application/jsonContent-Type: application/jsonContent-Length: 1234User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
{
  "merchantIdScanReference": "YOURSCANREFERENCE","successUrl": "https://www.your-site.com/sucess","errorUrl": "https://www.your-site.com/error"
}
```
|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|

### Sample Response

```
{
  "timestamp": "2017-08-16T10:27:29.494Z",
  "authorizationToken": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "jumioIdScanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```


## Displaying and configuring your Netverify client

Copy and paste the following script into your page. If your customer account is in the EU data center, use `lon.netverify.com` instead of `netverify.com`.

If you are using jQuery, include the jQuery JavaScript library beforehand.

```
<script type="text/javascript" src="https://netverify.com/widget/jumio-verify/2.0/iframe-script.js"></script><script type="text/javascript">
  /*<![CDATA[*/
  JumioClient.setVars({
    authorizationToken: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  }).initVerify("JUMIOIFRAME");
  /*]]>*/
</script>
```

To display Netverify, paste

`<div id="JUMIOIFRAME"></div>`

in your HTML code where you want the client to appear.

### Embed Code Parameters

**Required items appear in bold type.**  

|Parameter|Type |Description|
|:--------|:----|:----------|
|**authorizationToken**|String|Your transaction-specific authorization token|
|clientHeight|String|For a fixed size choose from `small` to `big`, for a responsive size use `responsive`. <br><br>Possible values:<br>• `responsive` (default)<br>• `small` (600px)<br>• `medium` (620px)<br>• `big` (640px)|
|clientWidth|String|For a fixed size choose from `small` to `big`, for a responsive size use `responsive`. <br><br>Possible values:<br>• `responsive` (default)<br>• `small` (800px)<br>• `medium` (860px)<br>• `big` (900px)|
|locale|String|Locale of the Netverify client. This parameter overrides the initiateNetverify parameter `locale`.<br>• "bg" Bulgarian<br>• "cs" Czech<br>• "da" Danish<br>• "de" German<br>• "el" Greek<br>• "en" American English (default)<br>• "en\_GB" British English<br>• "es" Spanish<br>• "es\_MX" Spanish Mexico<br>• "et" Estonian<br>• "fi" Finnish<br>• "fr" French<br>• "hu" Hungarian<br>• "it" Italian<br>• "ja" Japanese<br>• "ko" Korean<br>• "lt" Lithuanian<br>• "nl" Dutch<br>• "no" Norwegian<br>• "pl" Polish<br>• "pt" Portuguese<br>• "pt\_BR" Brazilian Portuguese<br>• "ro" Romanian<br>• "ru" Russian<br>• "sk" Slovak<br>• "sv" Swedish<br>• "tr" Turkish<br>• "vi" Vietnamese<br>• "zh\_CN" Chinese (China)<br>• "zh\_HK" Chinese (Hong Kong)|

### Sample: Responsive Layout

```
<script type="text/javascript" src="https://netverify.com/widget/jumio-verify/2.0/iframe-script.js"></script>
<script type="text/javascript">  /*
<![CDATA[*/     
  JumioClient.setVars({
    authorizationToken: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    clientHeight: "responsive",
    clientWidth: "responsive"
    }).initVerify("JUMIOIFRAME");  
/*]]>*/
</script>
```

### iFrame Logging (optional)

Each Netverify Web embedded instance notifies the customer’s embedding page that it loaded correctly using the JavaScript `window.postMessage()` method. This notification includes data that allow the embedding page to identify which instance of Netverify Web was loaded. Note: The notification is not sent if the Netverify Web embedded instance did not start properly, for example if the user's authorization token expired.

**Note**: Please keep in mind that some older browsers (most notably IE 7 and below) do not support this method of communication and no notification will be sent for instances of Netverify Web embedded loaded in one of these browsers. For a full list of supported browsers, please consult the following page: http://caniuse.com/#search=postmessage (click "Show all" to view all browsers)

Using JavaScript, the embedding page can receive the notification and consume the data it contains by listening for the "message" event on the "window" object and reacting to it as needed. The data passed by Netverify Web in this notification is represented in the "data" JSON string property of the listener method's "event" argument. Parsing this JSON string results in an object with the following properties:

|Property|Type|Required|Description|
|:-------|:---|:-------|:----------|
|authorizationToken|String|yes|Authorization token, which is valid for the authorization token lifetime|
|scanReference|String|yes|Jumio's reference number for each scan|
|timestamp|String|yes|Timestamp when Netverify instance was rendered. <br>Format: *YYYY-MM-DDThh:mm:ss.SSSZ*|

All data passed is encoded with UTF-8.

### Example of JavaScript Code

```
function receiveMessage(event) {  var data = window.JSON.parse(event.data);  console.log('Netverify Web embedded was loaded.');  console.log('authorization token:', data.authorizationToken);  console.log('scan reference:', data.scanReference);  console.log('timestamp:', data.timestamp);}window.addEventListener("message", receiveMessage, false);
```

### <a name="embedded-redirecting-the-customer-after-the-user-journey"></a>Redirecting the customer after the user journey

After finishing the Netverify user journey, the iFrame will be redirected to your specified success page, if the ID was able to be processed.

If the ID is declined as fraud immediately, or if the customer does not provide an ID which can be processed after three attempts (see [Reject reasons](/netverify/callback.md#reject-reason)), the user will be sent to your specified error page.

To display information on your redirect pages, you can use the following parameters, which are attached to your URL as HTTP GET query string parameters:

|Parameter|Description|
|:---|:---|
|idScanStatus|`SUCCESS` - if document uplaoded succesfully <br> or `ERROR`|
|merchantIdScanReference|Your reference for each scan|
|jumioIdScanReference|Jumio's reference number for each scan|
|errorCode|Possible codes: <br>• `110` (network communication problem)<br>• `210` (authorization token invalidates or expires during user journey)<br>• `310` (denied as fraud immediately)<br>• `320` (ID could not be processed, e.g. bad image quality)<br>• `340` (capture method restricted to camera and camera not available)|

**Note:** Because HTTP GET parameters can be manipulated on the client side, they may be used for display purposes only. It is also possible to set success and error URLs to the same address, because you can get the status from the URL's query parameter `idScanStatus`.

### Sample Redirect URL: Success

```
https://www.your-successurl.com/success?idScanStatus=SUCCESS&merchantIdScanReference=YOURIDSCANREFERENCE&jumioIdScanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

### Sample Redirect URL: Error

```
https://www.your-errorurl.com/error?idScanStatus=ERROR&merchantIdScanReference=YOURIDSCANREFERENCE&jumioIdScanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&errorCode=320
```

### Sample Redirect URL: Expired Authorization Token
In case the user is trying to go through the user journey after their authorization token has expired. They will be redirected to the error URL with different parameters.

```
https://www.your-errorurl.com/?idScanStatus=ERROR&errorCode=xxx&authorizationToken=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

## Callback

The callback is the authoritative answer from Jumio. Specify a callback URL (for constraints see [Global Netverify settings](/netverify/portal-settings.md#callback-url)</b>) to recieve the ID verification result for each scan (see [Callback](/netverify/callback.md)).


---

# Using Netverify Redirect

## Customizing your landing page

In your Jumio portal, you can customize and brand your Netverify redirect page.

### Application Settings

#### Domain Name Prefix

Define the domain name prefix for the URL of your landing page.
- Allowed characters are: a-z, 0-9, "-"
- Must not start or end with: "-"
- Max. 63 characters


#### Default Locale

Set your default language for Netverify redirect.<br>
Possible locales are: Bulgarian, Czech, Chinese (China), Chinese (Hong Kong), Danish, Dutch, English, English (United Kingdom), Estonian, Finnish, French, German, Greek, Hungarian, Italian, Japanese, Korean, Lithuanian, Norwegian, Polish, Portuguese, Portuguese (Brazil), Romanian, Russian, Slovak, Spanish, Spanish Mexico, Swedish, Turkish or Vietnamese.


### Customize Client

#### Select locale to configure

Images and labels can be defined per locale. Any configuration which is not set will first default to the language (e.g. EN\_GB to EN), then to your default setup, and finally to the Jumio standard.

#### Images

The file size for header, intro, success, or error image must not exceed 5 MB.

**Note:** Success and error images will be applied for the Jumio default redirect pages, if you are not specifying your own success and error URLs.

#### Label for customer ID

Specify the label for the customer identification field on your landing page. This field must not contain sensitive data like PII (Personally Identifiable Information) or account login.

## <a name="redirect-initiating-the-transaction"></a>Initiating the Transaction

Use the manual setup in your Jumio customer portal or call the **initiateNetverifyRedirect** API to create a transaction for each user. You will receive a Jumio scan reference and your Netverify Redirect URL including an authorization token, which is valid for the token lifetime.

### Using the Jumio Customer Portal

You can create a transaction manually in your Jumio customer portal under the "Create verfication" tab. The field values you can specify are equal to the related API request parameters described in the previous section of the guide.

### Using the initiateNetverifyRedirect API

Call the RESTful HTTP POST API <b>initiateNetverifyRedirect</b> with the JSON parameters below.

**HTTP Request Method:** `POST`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/initiateNetverifyRedirect`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/initiateNetverifyRedirect`<br>

**Authentication**:
Netverify API calls are protected using [HTTP Basic Authentication](https://tools.ietf.org/html/rfc7617). Your Basic Auth credentials are constructed using your API token as the user-id and your API secret as the password. You can view and manage your API token and secret in the Customer Portal under **Settings > API credentials**.
<br>

|⚠️ Never share your API token, API secret, or Basic Auth credentials with *anyone* — not even Jumio Support.
|:----------|

The [TLS Protocol](https://tools.ietf.org/html/rfc5246) is required to securely transmit your data, and we strongly recommend using the latest version. For information on cipher suites supported by Jumio during the TLS handshake see [Supported cipher suites](/netverify/supported-cipher-suites.md). <br>Note: Calls with missing or suspicious headers, suspicious parameter values, or without HTTP Basic Authentication will result in HTTP status code 403 Forbidden.

**Header:**
The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Content-Type: application/json`<br>
`Content-Length:`  (see [RFC-7230](https://tools.ietf.org/html/rfc7230#section-3.3.2))<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

<br>

### Request Parameters

|ℹ️ Values set in your API request will override the corresponding settings configured in the Customer Portal.
|:----------|

**Required items appear in bold type.**  

|Parameter|Type|Max. length|Description |
|:--------|:---|:----------|:-----------|
|**merchantIdScanReference**<sup>1</sup>|String|100|Your internal reference for the transaction.|
|**customerId**<sup>1</sup>|String|100| Your internal reference for the user.|
|successUrl<sup>2</sup>|String|2048|Redirect URL in case of success|
|errorUrl<sup>2</sup>|String|255|Redirect URL in case of error|
|enabledFields|String|100|Defines fields which will be extracted during the ID verification. If a field is not listed in this parameter, it will not be processed for this transaction, regardless of customer portal settings.<br> **Note:** Face match and Address extraction will not be processed unless enabled for your account. If you want to enable them, please contact your Customer Success Manager, or reach out to Jumio Support.<br><br>Possible values:<br>`idNumber,idFirstName,idLastName,idDob,idExpiry,idUsState,<br>idPersonalNumber,idFaceMatch,idAddress`|
|authorizationTokenLifetime|Number|Max. value:<br>5184000|Time in seconds until the authorization token expires<br>• Default: 1800 seconds<br>• 0: 60 days|
|merchantReportingCriteria|String|100|Your reporting criteria for each scan|
|callbackUrl|String|255|Callback URL for the confirmation after the verification is completed (for constraints see [Callback URL](/netverify/portal-settings.md#callback-url))|
|locale|String|100|Locale of the Netverify client:<br>• "bg" Bulgarian<br>• "cs" Czech<br>• "da" Danish<br>• "de" German<br>• "el" Greek<br>• "en" American English (default)<br>• "en\_GB" British English<br>• "es" Spanish<br>• "es\_MX" Spanish Mexico<br>• "et" Estonian<br>• "fi" Finnish<br>• "fr" French<br>• "hu" Hungarian<br>• "it" Italian<br>• "ja" Japanese<br>• "ko" Korean<br>• "lt" Lithuanian<br>• "nl" Dutch<br>• "no" Norwegian<br>• "pl" Polish<br>• "pt" Portuguese<br>• "pt\_BR" Brazilian Portuguese<br>• "ro" Romanian<br>• "ru" Russian<br>• "sk" Slovak<br>• "sv" Swedish<br>• "tr" Turkish<br>• "vi" Vietnamese<br>• "zh\_CN" Chinese (China)<br>• "zh\_HK" Chinese (Hong Kong)|
|clientIp|String|100|IP address of the client|
|firstName|String|100|First name of the customer|
|lastName|String|100|Last name of the customer|
|country|String|3|Possible countries:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>• `XKX` (Kosovo)|
|usState|String|100|Possible values if idType = `PASSPORT` or `ID_CARD`:<br>• Last two characters of [ISO 3166-2: US](http://en.wikipedia.org/wiki/ISO_3166-2:US) state code<br>• [ISO 3166-1](http://en.wikipedia.org/wiki/ISO_3166-1) country name<br>• Kosovo<br>• [ISO 3166-1 alpha-2](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) country code<br>• `XK` (Kosovo)<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>• `XKX` (Kosovo)<br><br>If idType = `DRIVING_LICENSE`:<br>• Last two characters of [ISO 3166-2: US state code](http://en.wikipedia.org/wiki/ISO_3166-2:US)|
|expiry|String||Date of expiry in the format *YYYY-MM-DD*|
|number|String|100|Identification number of the document|
|dob|String||Date of birth in the format *YYYY-MM-DD*|
|idType|String|255|Possible values: <br>• `PASSPORT`<br>• `DRIVING_LICENSE`<br>• `ID_CARD`|
|personalNumber|String|14|Personal number of the document|
|captureMethod|String||Capture method for this scan. Possible values: <br>• `CAM`<br>• `UPLOAD`<br>• `ALL`|
|presetCountry|String|3|Preset country and ID type to skip the "Select country and ID type" screen.<br>Note: Both parameters must be present.<br><br>Possible countries:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>• `XKX` (Kosovo)|
|presetIdType|String|255|Preset country and ID type to skip the "Select country and ID type" screen.<br>Note: Both parameters must be present.<br><br>Possible ID types: <br>• `PASSPORT`<br>• `DRIVING_LICENSE`<br>• `ID_CARD`|

<sup>1</sup> Values **must not** contain Personally Identifiable Information (PII) or other sensitive data such as email addresses.<br>
<sup>2</sup> See URL constraints for [Success and error URLs](/netverify/portal-settings.md#success-and-error-urls).

#### Supported documents for address extraction

|Country    |ID card    |Driving license    |Passport    |Callback format |
|:------------|:-------|:--------------|:--------------|:-------|
|Australia|No|Yes|No|US|
|Canada|No|Yes|No|US|
|France|Yes|Yes|Yes|Raw|
|Germany|Yes|No|No|EU|
|Ireland|No|Yes|No|Raw|
|Mexico|Yes|No|No|US|
|Singapore|Yes|No|No|Raw|
|Spain|Yes|No|No|EU|
|United Kingdom|No|Yes|No|Raw|
|United States|No|Yes|No|US|


### Response Parameters

|Parameter|Type|Max. length|Description |
|:--------|:---|:---------|:-------|
|timestamp|String||Timestamp of the response in the format YYYY-MM-DD|
|authorizationToken|String|36|Authorization token, which is valid for the authorization token lifetime|
|clientRedirectUrl|String|255|The redirect URL for the customer|
|jumioIdScanReference|String|36|Jumio's reference number for each scan|

### Sample Request

```
POST https://netverify.com/api/netverify/v2/initiateNetverifyRedirect HTTP/1.1
Accept: application/json  
Content-Type: application/json
Content-Length: 1234
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
{
  "merchantIdScanReference": "transaction_1234",
  "customerId": "user_1234",
  "successUrl": "https://www.your-site.com/success",
  "errorUrl": "https://www.your-site.com/error"
}
```
|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|

### Sample Response
```
{
  "timestamp": "2012-08-16T10:27:29.494Z",
  "authorizationToken": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "clientRedirectUrl": "https://[your-domain-prefix].netverify.com/v2?authorizationToken=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "jumioIdScanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

## Calling your landing page

The obtained `clientRedirectUrl`, which redirects to your customized landing page, can be placed as a link on your web page or included in an email to your customer.

**Note**: The URL including its HTTP `GET` parameter(s) may not be modified.


## <a name="redirect-redirecting-the-customer-after-the-user-journey"></a>Redirecting the customer after the user journey

After finishing the Netverify user journey, the iFrame will be redirected to your specified success page, if the ID was able to be processed.

If the ID is declined as fraud immediately, or if the customer does not provide an ID which can be processed after three attempts (see [Reject reason](/netverify/callback.md#reject-reason) chapter), the user will be sent to your specified error page.

To display information on your redirect pages, you can use the following parameters, which are attached to your URL as HTTP GET query string parameters:

|Parameter|Description|
|:---------|:----------|
|idScanStatus|`SUCCESS` - if document uploaded successfully <br>or `ERROR`|
|merchantIdScanReference|Your reference for each scan|
|jumioIdScanReference|Jumio's reference number for each scan|
|errorCode|Possible codes: <br>• `110` (network communication problem)<br>• `210` (authorization token invalidates or expires during user journey)<br>• `310` (denied as fraud immediately)<br>• `320` (ID could not be processed, e.g. bad image quality)<br>• `340` (capture method restricted to camera and camera not available)|

**Note:** Because HTTP GET parameters can be manipulated on the client side, tey may be used for display purposes only. It is also possible to set success and error URLs to the same address, because you can get the status from the URL's query parameter "idScanStatus".

### Sample Redirect URL: Success

```
https://www.your-successurl.com/success?idScanStatus=SUCCESS&merchantIdScanReference=YOURIDSCANREFERENCE&jumioIdScanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

### Sample Redirect URL: Error

```
https://www.your-errorurl.com/error?idScanStatus=ERROR&merchantIdScanReference=YOURIDSCANREFERENCE&jumioIdScanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&errorCode=320
```

### Sample Redirect URL: Expired Authorization Token
In case the user is trying to go through the user journey after their authorization token has expired. They will be redirected to the error URL with different parameters.

```
https://www.your-errorurl.com/?idScanStatus=ERROR&errorCode=xxx&authorizationToken=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

### Callback
The callback is the authoritative answer from Jumio. Specify a callback URL (for constaints see <b>[Global Netverify settings](/netverify/portal-settings.md)</b>) to recieve the ID verification result for each scan (see [Callback](/netverify/callback.md)).

---
# Using performNetverify

See [performNetverify](/netverify/performNetverify.md)

---

# Global Netverify APIs

## Supported IDs

Call the RESTful HTTP GET API **supportedIdTypes** to receive a JSON response including all Jumio supported IDs.

**HTTP Request Method:** `GET`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/supportedIdTypes`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/supportedIdTypes`<br>

**Authentication**:
Netverify API calls are protected using [HTTP Basic Authentication](https://tools.ietf.org/html/rfc7617). Your Basic Auth credentials are constructed using your API token as the user-id and your API secret as the password. You can view and manage your API token and secret in the Customer Portal under **Settings > API credentials**.
<br>

|⚠️ Never share your API token, API secret, or Basic Auth credentials with *anyone* — not even Jumio Support.
|:----------|

The [TLS Protocol](https://tools.ietf.org/html/rfc5246) is required to securely transmit your data, and we strongly recommend using the latest version. For information on cipher suites supported by Jumio during the TLS handshake see [Supported cipher suites](/netverify/supported-cipher-suites.md). <br>Note: Calls with missing or suspicious headers, suspicious parameter values, or without HTTP Basic Authentication will result in HTTP status code 403 Forbidden.

**Header:**
The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

<br>

### Response Parameters

|Parameter|Type|Description|
|:----|:----|:----|
|supportedIdTypes|Array|Array of supported IDs|
|timestamp|String|Timestamp of the response. <br> Format: *YYYY-MM-DDThh:mm:ss.SSSZ*|

|Parameter "supportedIdTypes"|Type|Max. length|Description|
|:----|:----|:----|:----|
|countryName|String|100|Possible names:<br>•	[ISO 3166-1](http://en.wikipedia.org/wiki/ISO_3166-1) country name<br>•	Kosovo|
|countryCode|String|3|Possible codes:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>•	`XKX` (Kosovo)|
|idTypes|Array||Array of supported ID types for this country|

|Parameter "idTypes"|Type|Max. length|Description|
|:----|:----|:----|:----|
|idType|String|255|Possible values:<br>•	`PASSPORT`<br>•	`DRIVING_LICENSE`<br>•	`ID_CARD`|
|acquisitionConfig|Object||Object containing acquisition configuration for this ID type|

|Parameter "acquisitionConfig"|Type|Max. length|Description|
|:----|:----|:----|:----|
|backSide|Boolean||True if back side required, otherwise false|

### Sample Request

```
GET https://netverify.com/api/netverify/v2/supportedIdTypes HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic
```

### Sample Response
```
{
"supportedIdTypes": [
  {
  "countryName": "Austria",
  "countryCode": "AUT",
  "idTypes": [
    {
    "idType": "ID_CARD",
    "acquisitionConfig": {
      "backSide": true
      }
    },
    {
    "idType": "DRIVER_LICENSE",
    "acquisitionConfig": {
      "backSide": false
      }
    },
    {
    "idType":"PASSPORT",
    "acquisitionConfig": {
      "backSide": false
      }
    }
    ]
  },
  {
  "countryName": "United states",
  "countryCode": "USA",
  "idTypes": [
    {
    "idType": "ID_CARD",
    "acquisitionConfig": {
      "backSide": false
      }
    },
    {
    "idType": "DRIVER_LICENSE",
    "acquisitionConfig": {
      "backSide": false
      }
    },
    {
    "idType": "PASSPORT",
    "acquisitionConfig": {
      "backSide": false
      }
    }
    ]
  }
  ],
"timestamp": "2013-04-10T07:38:25.332Z"
}
```

## Accepted IDs

Call the RESTful HTTP GET API **acceptedIdTypes** to receive a JSON response including all accepted IDs, as specified in your Jumio portal settings.

**HTTP Request Method:** `GET`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/acceptedIdTypes`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/acceptedIdTypes`<br>

**Authentication**:
Netverify API calls are protected using [HTTP Basic Authentication](https://tools.ietf.org/html/rfc7617). Your Basic Auth credentials are constructed using your API token as the user-id and your API secret as the password. You can view and manage your API token and secret in the Customer Portal under **Settings > API credentials**.
<br>

|⚠️ Never share your API token, API secret, or Basic Auth credentials with *anyone* — not even Jumio Support.
|:----------|

The [TLS Protocol](https://tools.ietf.org/html/rfc5246) is required to securely transmit your data, and we strongly recommend using the latest version. For information on cipher suites supported by Jumio during the TLS handshake see [Supported cipher suites](/netverify/supported-cipher-suites.md). <br>Note: Calls with missing or suspicious headers, suspicious parameter values, or without HTTP Basic Authentication will result in HTTP status code 403 Forbidden.

**Header:**
The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

<br>

### Response Parameters

|Parameter|Type|Description|
|:--------|:---|:----------|
|accepteddIdTypes|Array|Array of accepted IDs, as specified in your Jumio portal settings|
|timestamp|String|Timestamp of the response. <br>Format: *YYYY-MM-DDThh:mm:ss.SSSZ*|

|Parameter "accepteddIdTypes"|Type|Max. length|Description|
|:----|:----|:----|:----|
|countryName|String|100|Possible names:<br>•	[ISO 3166-1](http://en.wikipedia.org/wiki/ISO_3166-1) country name<br>•	Kosovo|
|countryCode|String|3|Possible codes:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>• `XKX (Kosovo)`|
|idTypes|Array||Array of your accepted ID types for this country|

|Parameter "idTypes"|Type|Max. length|Description|
|:----|:----|:----|:----|
|idType|String|255|Possible values: <br>• `PASSPORT`<br>• `DRIVING_LICENSE`<br>• `ID_CARD`|
|acquisitionConfig|Object||Object containing acquisition configuration for this ID type|

|Parameter "acquisitionConfig"|Type|Max. length|Description|
|:----|:----|:----|:----|
|backSide|Boolean||`true` if back side required, otherwise `false`|


### Sample Request

```
GET https://netverify.com/api/netverify/v2/acceptedIdTypes HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic
```


### Sample Response

```
{
"acceptedIdTypes": [
  {
  "countryName": "Austria",
  "countryCode": "AUT",
  "idTypes": [
    {
    "idType": "ID_CARD",
    "acquisitionConfig": {
      "backSide": true
      }
    },
    {
    "idType": "DRIVER_LICENSE",
    "acquisitionConfig": {
      "backSide": false
      }
    },
    {
    "idType":"PASSPORT",
    "acquisitionConfig": {
      "backSide": false
      }
    }
    ]
  },
  {
  "countryName": "United states",
  "countryCode": "USA",
  "idTypes": [
    {
    "idType": "ID_CARD",
    "acquisitionConfig": {
      "backSide": false
      }
    },
    {
    "idType": "DRIVER_LICENSE",
    "acquisitionConfig": {
      "backSide": false
      }
    },
    {
    "idType": "PASSPORT",
    "acquisitionConfig": {
      "backSide": false
      }
    }
    ]
  }
  ],
"timestamp": "2013-04-10T07:38:25.332Z"
}
```

---
# Netverify Delete API

You can implement the RESTful DELETE API to remove sensitive data (e.g. name, address, date of birth, document number, etc.) and image(s) of a finished scan. Find the Implementation Guide at the link below.

[View Delete API Guide](/netverify/netverify-delete-api.md)

---
# Global Netverify Settings

In your Jumio customer portal you can configure your settings. Find the description of the settings at the link below.

[View portal settings](/netverify/portal-settings.md)


---
# Supported Cipher Suites
Jumio supported cipher suites during the TLS handshake.<p>
[View supported cipher suites](/netverify/supported-cipher-suites.md)


---
# Supported Browsers

Jumio offers guaranteed support for Netverify Web and Document Verification on the following browsers and the latest major version of each operating system.

Capture methods:
- Image upload
- HTML5 video stream
- Flash video stream

## Desktop

|Browser name|Major browser version|Operating system |Image upload |HTML5<br>video stream |Flash  <sup>1</sup><br> video stream|
|:---|:---|:---|:---|:---|:---|
|Google Chrome|current +<br> 1 previous|Windows + Mac|X|X|X|
|Mozilla Firefox|current +<br>1 previous|Windows + Mac|X|X|X|
|Apple Safari|current|Mac|X|X<sup>2</sup>|O|
|Microsoft Internet Explorer|current|Windows|X| |X|
|Microsoft Edge|current|Windows|X|X|X|

## Mobile

|Browser name|Major browser version|Operating system |Image upload |HTML<br>video stream |Flash  <sup>1</sup><br> video stream|
|:---|:---|:---|:---|:---|:---|
|Google Chrome |current |Android|X|X| |
|Apple Safari |current |iOS|X|X<sup>2</sup>| |

X - Supported<br>
O - Support turned off by default but can be enabled by the end-user<br>
<br>
<sup>1</sup> Fallback method for HTML5 video streaming<br>
<sup>2</sup> Netverify redirect only

---
# Test IDs

See [Netverify Test IDs](/netverify/netverify-test-ids.md)


---
&copy; Jumio Corp. 268 Lambert Avenue, Palo Alto, CA 94306
