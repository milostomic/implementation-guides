![Jumio](/images/netverify.png)

# Netverify Document Verification Implementation Guide

This is a reference manual and configuration guide for the Netverify Document Verification (formerly Multi Document) product. It illustrates how to embed Netverify Document Verification into your web page, as well as implement the Netverify Document Verification APIs.


## Table of Contents
- [Release Notes](#release-notes)
- [Using Document Verification](#using-document-verification)
    - [Initiating the transaction](#initiating-the-transaction)
    - [Displaying your Document Verification client](#displaying-your-document-verification-client)
    - [Redirecting the customer after the user journey](#redirecting-the-customer-after-the-user-journey)
- [Using the Document Verification API](#using-the-document-verification-api)
    - [Multi page upload: Initiating the transaction](#multi-page-upload-initiating-the-transaction)
    - [Multi-page Upload: Adding a Page](#multi-page-upload-adding-a-page)
    - [Multi-page Upload: Adding a Document](#multi-page-upload-adding-a-document)
    - [Multi page upload: Finalizing the transaction](#multi-page-upload-finalizing-the-transaction)
    - [Single page upload](#single-page-upload)
- [Global Netverify APIs](#global-netverify-apis)
    - [Supported documents](#supported-documents)
- [Netverify Delete API](#netverify-delete-api)
- [Global Netverify Settings](#global-netverify-settings)
- [Supported Cipher Suites](#supported-cipher-suites)


---
# Release Notes

The release information for Document Verification can be found via the link below.<p>
[View Release Notes](/netverify/README.md)

----

# Using Document Verification

Netverify Document Verification offers document upload and extraction (see [Supported documents for data extraction](#supported-documents)) with a Jumio-hosted user interface. Simply use the following RESTful API and you will receive a callback after completion (see [Callback](/netverify/callback.md)).

## Initiating the Transaction

Call the RESTful HTTP POST API **acquisitions** with the below JSON parameters, in order to create a transaction for each user. You will receive a Jumio scan reference and your client redirect URL, which will be valid for a certain amount of time. The length of time is specified in the request parameter, `authorizationTokenLifetime`.

HTTP request method: **POST**<br>
**REST URL:** `https://upload.netverify.com/api/netverify/v2/acquisitions`<br>
Note: If your customer account is in the EU data center, use `lon.netverify.com` instead of `netverify.com`.

**Authentication:** The acquisitions API call is protected. To access it, use HTTP Basic Authentication with your API token as the "userid" and your API secret as the "password". You can find your API token and API secret by logging into your Jumio customer portal and navigating to the "Settings" page and clicking on the "API credentials" tab.

**Header:** The following parameters are mandatory in the "header" section of your request.
-	`Accept: application/json`
-	`Content-Type: application/json`
- `Content-Length: xxx` (RFC-2616)
- `User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`<br>
(e.g. MyCompany MyApp/1.0.0, change this to reflect your company)

**TLS handshake:** The TLS protocol is required (see [Supported Cipher Suites](/netverify/supported-cipher-suites.md)) and we strongly recommend using the latest version.

**Note:** Calls with missing or suspicious headers, suspicious parameter values, or without HTTP Basic Authentication will result in the HTTP status code, **403 Forbidden**.

### Request Parameters

**Note:** Mandatory JSON parameters are highlighted bold.

|Parameter       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|**type**|String||Possible codes:<br>See [Supported documents](#supported-documents) chapter|
|**country**<br>Not mandatory for type=CC|String|3|Possible countries:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>•	XKX (Kosovo)|
|**merchantScanReference**|String|100|Your reference for each scan must not contain sensitive data like PII (Personally Identifiable Information) or account login|
|**customerId**|String|100|Identification of the customer should not contain sensitive data like PII (Personally Identifiable Information) or account login|
|callbackUrl|String|255|Callback URL for the confirmation after the user journey is completed (constraints see [Callback URL](/netverify/portal-settings.md#callback-url) chapter). This setting overrides your Jumio portal settings.|
|successUrl|String|255|Redirect URL in case of success (constraints see [Success and error URLs](/netverify/portal-settings.md#success-and-error-urls) chapter).|
|errorUrl|String|255|Redirect URL in case of error (constraints see [Success and error URLs](/netverify/portal-settings.md#success-and-error-urls) chapter).|
|authorizationTokenLifetime|Number|Max. value: 5184000|Time in seconds until the authorization token expires<br>Default: 1800 seconds<br>0: 60 days |
|merchantReportingCriteria|String|255|Your reporting criteria for each scan|
|locale|String|100|Locale of the Document verification client:<br>•	"bg" Bulgarian<br>•	"cs" Czech<br>•	"da" Danish<br>•	"de" German<br>•	"el" Greek<br>•	"en" American English (default)<br>•	"en\_GB" British English<br>•	"es" Spanish<br>•	"es\_MX" Spanish Mexico<br>•	"et" Estonian<br>•	"fi" Finnish<br>•	"fr" French<br>•	"hu" Hungarian<br>•	"it" Italian<br>•	"ja" Japanese<br>•	"ko" Korean<br>•	"lt" Lithuanian<br>•	"nl" Dutch<br>•	"no" Norwegian<br>•	"pl" Polish<br>•	"pt" Portuguese<br>•	"pt_BR" Brazilian Portuguese<br>•	"ro" Romanian<br>•	"ru" Russian<br>•	"sv" Swedish<br>•	"tr" Turkish<br>•	"sk" Slovak<br>•	"zh\_CN" Chinese (China)<br>•	"zh\_HK" Chinese (Hong Kong)|
|baseColor|String|6|Client's main web color as hexadecimal triplet <br>Note: Custom colors are only used if both, baseColor and bgColor, are present|
|bgColor|String|6|Client's background web color as hexadecimal triplet<br>Note: Custom colors are only used if both, baseColor and bgColor, are present|
|headerImageUrl|String|255|Defines your own header logo, which should be landscape (16:9 or 4:3) with a min. height of 192 pixels and a size between 8 and 64 KB.<br>URL constraints:<br>•	HTTPS using the TLS protocol<br>•	Valid URL (RFC-2396) using ASCII characters or IDNA Punycode<br>•	IP addresses, ports, query parameters and fragment identifiers are not allowed |
|customDocumentCode|String|100|Your custom document code (see [Multi documents](/netverify/portal-settings.md#multi-documents) chapter)<br>Needs to be added if type = CUSTOM|

### Response Parameters

|Parameter       | Type    | Max. Length| Description|
|:---------------|:--------|:-----------|:------------|
|scanReference|String|36|Jumio’s reference number for each scan|
|clientRedirectUrl|String|2000|The redirect URL for the customer|

### Sample Request
```
POST https://upload.netverify.com/api/netverify/v2/acquisitions HTTP/1.1
Accept: application/json
Content-Type: application/json
Content-Length: xxx
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
{
"type": "SSC",
"country": "USA",
"merchantScanReference": "YOURSCANREFERENCE",
"customerId": "CUSTOMERID"
}
```

### Sample Request: Custom Document Code

```
POST https://upload.netverify.com/api/netverify/v2/acquisitions HTTP/1.1
Accept: application/json
Content-Type: application/json
Content-Length: xxx
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
{
"type": "CUSTOM",
"customDocumentCode": "XXX",
"country": "USA",
"merchantScanReference": "YOURSCANREFERENCE",
"customerId": "CUSTOMERID"
}
```

### Sample Response
```
{
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
"clientRedirectUrl": "https://upload.netverify.com/api/netverify/v2/acquisitions?initiationToken=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```


## Displaying your Document Verification Client

To display Document Verification, you can add the obtained "clientRedirectUrl" into the "src" attribute of an "iFrame" tag by specifying width and height, or as a link on your web page.

We recommend considering the following minimum/maximum dimensions:
-	Min. size: 320 x 480 pixels (or vice versa if landscape on mobile)
-	Max. width of 900 pixels (for iFrame only)

**Note:** The URL including its HTTP GET parameter(s) may not be modified.

## Redirecting the customer after the user journey

After successfully completing the Document Verification user journey, the user will be redirected to your specified success page. In case of an error, the user will be sent to your specified error page.

You can use the following parameters to display information on your redirect pages. These are attached to your URL as HTTP GET query string parameters:

|Parameter  | Description|
|:----------|:------------|
|idScanStatus|"SUCCESS" or "ERROR"|
|jumioIdScanReference|Jumio’s reference number for each scan|
|errorCode|Possible codes:<br>•	211 (authorization token invalidates)<br>•	310 (upload session has expired)<br>•	311 (document upload was already completed for provided scan reference)<br>•	320 (browser session has been started multiple times)<br>•	338 (browser is in private mode (not supported)|

**Note:** Since HTTP GET parameters can be manipulated on the client side, they may be used for display purposes only. It is also possible to configure success and error URLs with the same address; as you can get the status from the URL’s query parameter "idScanStatus".

### Sample Redirect URL: Success
```
https://www.your-site.com/success?idScanStatus=SUCCESS&jumioIdScanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

### Sample Redirect URL: Error
```
https://www.your-site.com/error?idScanStatus=ERROR&jumioIdScanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&errorCode=320
```

## Callback

The callback is the authoritative answer from Jumio. Specify a callback URL (see [Global Netverify settings](/netverify/portal-settings.md)) to receive the detailed results for each scan (see [Callback](/netverify/callback.md)).

----
# Using the Document Verification API

The Document Verification API offers document upload and extraction (see [Supported documents for data extraction](/netverify/callback.md#supported-documents-for-data-extraction)) without a Jumio-hosted user interface. Simply use the following RESTful APIs to submit the user's document. You will receive the results via a callback after completion (see [Callback](/netverify/callback.md#callback-for-document-verification) page).

**Authentication:** The below API calls are protected. To access them, use HTTP Basic Authentication with your API token as the "userid" and your API secret as the "password". Log into your Jumio customer portal and you can find your API token and API secret on the "Settings" page under "API credentials".

**TLS handshake:** The TLS protocol is required (see [Supported Cipher Suites](/netverify/supported-cipher-suites.md)) and we strongly recommend using the latest version.

**Note:** Calls with missing or suspicious headers, suspicious parameter values, or without HTTP Basic Authentication will result in an HTTP status code, **403 Forbidden**.



## Multi Page Upload: Initiating the Transaction

Call the RESTful HTTP POST API **acquisitions** with the below JSON parameters to create a transaction for each user.

HTTP request method: **POST**<br>
**REST URL:** `https://acquisition.netverify.com/api/netverify/v2/acquisitions`
If your customer account is in the EU data center, use `acquisition.lon.netverify.com` instead of `acquisition.netverify.com`.

**Header:** The following parameters are mandatory in the "header" section of your request.
-	`Accept: application/json`
-	`Content-Type: application/json`
-	`Content-Length: xxx` (RFC-2616)
-	`User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`<br>
(e.g. MyCompany MyApp/1.0.0, change this to reflect your company)

### Request Parameters

**Note:** Mandatory JSON parameters are highlighted bold.

|Parameter       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|**type**<br>Not mandatory for type=CC|String||Possible codes:<br>See [Supported documents](#supported-documents) chapter |
|**country**|String|3|Possible countries:<br>•	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>•	XKX (Kosovo)|
|**merchantScanReference**|String|100|Your reference for each scan must not contain sensitive data like PII (Personally Identifiable Information) or account login|
|**customerId**|String|100|Identification of the customer should not contain sensitive data like PII (Personally Identifiable Information) or account login|
|callbackUrl|String|255|Callback URL for the confirmation after the user journey is completed (constraints see [Callback URL](/netverify/portal-settings.md#callback-url) settings). This setting overrides your Jumio portal settings.|
|merchantReportingCriteria|String|100|Your reporting criteria for each scan|
|clientIp|String|100|IP address of the client|
|customDocumentCode|String|100|Your custom document code (see [Multi documents](/netverify/portal-settings.md#multi-documents))<br>Needs to be added if type = CUSTOM|


### Response Parameters

|Parameter       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|timestamp|String||Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|scanReference|String|36|Jumio’s reference number for each scan|

### Sample Request
```
POST https://acquisition.netverify.com/api/netverify/v2/acquisitions HTTP/1.1
Accept: application/json
Content-Type: application/json
Content-Length: xxx
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
{
"type": "SSC",
"country": "USA",
"merchantScanReference": "YOURSCANREFERENCE",
"customerId": "CUSTOMERID"
}
```

### Sample Response
```
{
"timestamp": "2017-10-17T06:37:51.969Z",
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```



## Multi-page Upload: Adding a Page

Call the RESTful HTTP POST API **pages** for a specified scan reference and page number to add a JPEG, PNG or PDF file with a maximum size of 10 MB.

HTTP request method: **POST**<br>
REST URL: `https://acquisition.netverify.com/api/netverify/v2/acquisitions/<scanReference>/document/pages/<page>`<br>
If your customer account is in the EU data center, use `acquisition.lon.netverify.com` instead of `acquisition.netverify.com`.

**Header:** The following parameters are mandatory in the "header" section of your request.
-	`Accept: application/json`
-	`Content-Type: multipart/form-data`
-	`Content-Length: xxx` (RFC-2616)
-	`User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`<br>
(e.g. MyCompany MyApp/1.0.0, change this to reflect your company)

### Request Parameters

|Parameter       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|scanReference <br>(path parameter)|String|36|Jumio’s reference number of a scan created/updated less than 5 minutes ago|
|page <br>(path parameter) |String|Max. value: 30|Page number|

Send your JPEG, PNG or PDF file with a maximum size of 10 MB as `multipart/form-data` in the body of your request.

### Response Parameter

You receive a JSON response in case of success, or HTTP status code **404 Not Found** if the scan is not available.

|Parameter       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|timestamp|String||Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|

### Sample Request
```
POST https://acquisition.netverify.com/api/netverify/v2/acquisitions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/document/pages/1 HTTP/1.1
Accept: application/json
Content-Type: multipart/form-data;boundary=----xxxx
Content-Length: xxx
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
----xxxx
// Your JPEG, PNG or PDF page
----xxxx
```

### Sample Response
```
{
"timestamp": "2017-10-17T07:56:58.613Z"
}
```

## Multi-page Upload: Adding a Document

Call the RESTful HTTP POST API **document** for a specified scan reference to add a PDF file with a maximum size of 10 MB and maximum of 30 pages.

HTTP request method: **POST**<br>
**REST URL:** `https://acquisition.netverify.com/api/netverify/v2/acquisitions/<scanReference>/document`<br>
If your customer account is in the EU data center, use `acquisition.lon.netverify.com`  instead of `acquisition.netverify.com`.

**Header:** The following parameters are mandatory in the "header" section of your request.
- `Accept: application/json`
-	`Content-Type: multipart/form-data`
-	`Content-Length: xxx` (RFC-2616)
-	`User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`<br>
(e.g. MyCompany MyApp/1.0.0, change this to reflect your company)

### Request Parameter

|Parameter       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|scanReference<br>(path parameter)|String|36|Jumio’s reference number of a scan created/updated less than 5 minutes ago|

Send your PDF file with a maximum size of 10 MB and maximum of 30 pages as `multipart/form-data` in the body of your request.

### Response Parameters

You receive a JSON response in case of success, or HTTP status code **404 Not Found** if the scan is not available.

|Parameter       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|timestamp|String||Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|pages|Number|Max. value: 30|Number of extracted pages|

### Sample Request

```
POST https://acquisition.netverify.com/api/netverify/v2/acquisitions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/document HTTP/1.1
Accept: application/json
Content-Type: multipart/form-data;boundary=----xxxx
Content-Length: xxx
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
----xxxx
// Your PDF document
----xxxx
```

### Sample Response
```
{
"timestamp": "2017-10-17T07:56:58.613Z",
"pages": 6
}
```

## Multi-page Upload: Finalizing the Transaction

Call the RESTful HTTP PUT API **acquisitions** for a specified scan reference to finalize the transaction.

HTTP request method: **PUT**<br>
**REST URL:** `https://acquisition.netverify.com/api/netverify/v2/acquisitions/<scanReference>`<br>
If your customer account is in the EU data center, use `acquisition.lon.netverify.com` instead of `acquisition.netverify.com`.

**Header:** The following parameter is mandatory in the "header" section of your request.
-	`Accept: application/json`
-	`User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`<br>
(e.g. MyCompany MyApp/1.0.0, change this to reflect your company)

### Request Parameter

|Parameter       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|scanReference (path parameter)|String|36|Jumio’s reference number of a scan created/updated less than 5 minutes ago |

### Response Parameter

You receive a JSON response in case of success, or HTTP status code **404 Not Found** if the scan is not available.

|Parameter       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|timestamp|String||Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|

### Sample Request
```
PUT https://acquisition.netverify.com/api/netverify/v2/acquisitions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```
### Sample Response
```
{
"timestamp": "2017-10-17T07:56:58.613Z"
}
```

## Single Page Upload

Call the RESTful HTTP POST API **complete** with the below parameters to initialize, upload and finalize a single page document in one API call. You can add a JPEG, PNG or a single page PDF file with a maximum size of 10 MB.

HTTP request method: **POST**<br>
**REST URL:** `https://acquisition.netverify.com/api/netverify/v2/acquisitions/complete`<br>
If your customer account is in the EU data center, use `acquisition.lon.netverify.com` instead of `acquisition.netverify.com`.

**Header:** The following parameters are mandatory in the "header" section of your request.
-	`Accept: application/json`
-	`Content-Type: multipart/form-data`
-	`Content-Length: xxx` (RFC-2616)
-	`User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`<br>
(e.g. MyCompany MyApp/1.0.0, change this to reflect your company)

### Request Parameters

**Note:** Mandatory JSON parameters are highlighted bold.

|Parameter       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|**image** |PDF, JPEG or PNG|Max. 10 MB|Image of the customer's document should be always listed before the metadata parameter|
|**metadata**|JSON||Metadata of the customer's document as JSON objects, see table below|


|Parameter "metadata"  | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|**type**<br>Not mandatory for type=CC|String||Possible codes:<br>See [Supported documents](#supported-documents) chapter |
|**country**|String|3|Possible countries:<br>•	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>•	XKX (Kosovo)|
|**merchantScanReference**|String|100|Your reference for each scan must not contain sensitive data like PII (Personally Identifiable Information) or account login|
|**customerId**|String|100|Identification of the customer should not contain sensitive data like PII (Personally Identifiable Information) or account login|
|callbackUrl|String|255|Callback URL for the confirmation after the user journey is completed (constraints see [Callback URL](/netverify/portal-settings.md#callback-url) settings). This setting overrides your Jumio portal settings.|
|merchantReportingCriteria|String|100|Your reporting criteria for each scan|
|clientIp|String|100|IP address of the client|
|customDocumentCode|String|100|Your custom document code (see [Multi documents](/netverify/portal-settings.md#multi-documents))<br>Needs to be added if type = CUSTOM|

Send your JPEG, PNG or PDF file with a maximum size of 10 MB as `multipart/form-data` in the body of your request.


### Response Parameters

|Parameter       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|timestamp|String||Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|scanReference|String|36|Jumio’s reference number for each scan|

### Sample Request
```
POST https://acquisition.netverify.com/api/netverify/v2/acquisitions/complete HTTP/1.1
Accept: application/json
Content-Type: multipart/form-data;boundary=----xxxx
Content-Length: xxx
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
----xxxx
Content-Disposition: form-data; name="image"; filename="xxx.png"
Content-Type: image/png
// Your JPEG, PNG or PDF page
----xxxx
Content-Disposition: form-data; name="metadata"
{
"type": "SSC",
"country": "USA",
"merchantScanReference":"YOURSCANREFERENCE",
"customerId": "CUSTOMERID",
}
----xxxx
```
### Sample Response
```
{
"timestamp": "2017-10-17T07:56:58.613Z",
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

---
# Global Netverify APIs

## Supported Documents

Call the RESTful HTTP GET API **supportedDocumentTypes** to receive a JSON response including the code and names of all supported documents.

HTTP request method: **GET**<br>
**REST URL:** `https://netverify.com/api/netverify/v2/supportedDocumentTypes`<br>
If your customer account is in the EU data center, use `lon.netverify.com` instead of `netverify.com`.

**Authentication:** The supportedDocumentTypes API call is protected. To access it, use HTTP Basic Authentication with your API token as the "userid" and your API secret as the "password".

**Header:** The following parameters are mandatory in the "header" section of your request.
-	`Accept: application/json`
-	`User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`<br>
(e.g. MyCompany MyApp/1.0.0, change this to reflect your company)

**TLS handshake:** The TLS protocol is required (see [Supported cipher suites](/netverify/supported-cipher-suites.md)) and we strongly recommend using the latest version.

**Note:** Calls with missing or suspicious headers, suspicious parameter values, or without HTTP Basic Authentication will result in HTTP status code, **403 Forbidden**.

### Response Parameters

|Parameter       | Type        | Description|
|:---------------|:------------|:------------|
|timestamp|String|Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|supportedDocumentTypes|Array|Array of supported documents|

|Parameter "supportedDocumentTypes" | Type |Max. Length       | Description|
|:---------------|:------------|:------------|:------------|
|code|String|6|Unique identifier of the document type|
|name|String|255|Full name of the document type|

### Sample Request
```
GET https://netverify.com/api/netverify/v2/supportedDocumentTypes HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```

### Sample Response
```
{
"timestamp":"2017-10-17T14:25:59.779Z",
"supportedDocumentTypes":[
{"code":"BS","name":"Bank statement"},
{"code":"CC","name":"Credit card"},
{"code":"IC","name":"Insurance card"},
{"code":"UB","name":"Utility bill"},
{"code":"CAAP","name":"Cash advance application"},
{"code":"CRC","name":"Corporate resolution certificate"},
{"code":"CCS","name":"Credit card statement"},
{"code":"LAG","name":"Lease agreement"},
{"code":"LOAP","name":"Loan application"},
{"code":"MOAP","name":"Mortgage application"},
{"code":"TR","name":"Tax return"},
{"code":"VT","name":"Vehicle title"},
{"code":"VC","name":"Voided check"},
{"code":"STUC","name":"Student card"},
{"code":"HCC","name":"Health care card"},
{"code":"CB","name":"Council bill"},
{"code":"SENC","name":"Seniors card"},
{"code":"MEDC","name":"Medicare card"},
{"code":"BC","name":"Birth certificate"},
{"code":"WWCC","name":"Working with children check"},
{"code":"SS","name":"Superannuation statement"},
{"code":"TAC","name":"Trade association card"},
{"code":"SEL","name":"School enrolment letter"},
{"code":"PB","name":"Phone bill"},
{"code":"USSS","name":"US social security card"},
{"code":"SSC","name":"Social security card"},
{"code":"CUSTOM","name":"DOCUMENT"}]
}
```

---
# Netverify Delete API

You can implement the RESTful DELETE API to remove sensitive data (e.g. name, address, date of birth, document number, etc.) and image(s) of a finished scan. The implementation guide can be viewed via the link below.

[View Guide](/netverify/netverify-delete-api.md)

---
# Global Netverify Settings

The configuration settings are located in your Jumio customer portal. The description of each of the settings are available via the link below.

[View Portal Settings](/netverify/portal-settings.md)


----
# Supported Cipher Suites
Jumio supported cipher suites during the TLS handshake.<p>
[View Supported Cipher Suites](/netverify/supported-cipher-suites.md)

----
&copy; Jumio Corp. 268 Lambert Avenue, Palo Alto, CA 94306
