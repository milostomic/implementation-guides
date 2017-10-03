![Jumio](/images/netverify.png)

# Netverify Retrieval API Implementation Guide

This guide illustrates how to implement the Netverify Retrieval API.

## Contact

If you have any questions regarding our implementation guide please contact Jumio Support at support@jumio.com or https://support.jumio.com. The Jumio online helpdesk contains a wealth of information regarding our service including demo videos, product descriptions, FAQs and other things that may help to get you started with Jumio. Check it out at: https://support.jumio.com.

## Table of Content

- [Release notes](#release-notes)
- [Usage](#usage)
- [Authentication and header](#authentication-and-header)
- [Netverify Retrieval](#netverify-retrieval)
    - [Retrieving scan status](#retrieving-scan-status)
    - [Retrieving scan details](#retrieving-scan-details)
    - [Retrieving document data only](#retrieving-document-data-only)
    - [Retrieving transaction data only](#retrieving-transaction-data-only)
    - [Retrieving verification data only](#retrieving-verification-data-only)
    - [Retrieving available images](#retrieving-available-images)
    - [Retrieving specific image](#retrieving-specific-image)
- [Netverify Multi Document Retrieval](#netverify-multi-document-retrieval)
    - [Retrieving scan status ](#multi-retrieving-scan-status)
    - [Retrieving scan details](#multi-retrieving-scan-details)
    - [Retrieving document data only](#multi-retrieving-document-data-only)
    - [Retrieving transaction data only](#multi-retrieving-transaction-data-only)
    - [Retrieving available images](#multi-retrieving-available-images)
    - [Retrieving specific image](#multi-retrieving-specific-image)
- [Supported cipher suites](#supported-cipher-suites)

## Release notes

|Version       | Date    | Description|
|:-------------|:--------|:------------|
|1.3.1  | 2017-09-21   |Removed response parameter "additionalInformation"|
|1.3.0  | 2017-08-24   |Added response parameter "identityVerification"|
|1.2.9  | 2017-05-23   |Added response parameter "issuingDate" for retrieving scan details and<br />document data only|
|1.2.8  | 2017-01-31   |Added value "PASSPORT_CARD_ID" and "MILITARY_ID" to parameter "idSubType" |
|1.2.7  | 2016-11-15   |Added reject reason detail code 10011 for PLACE_OF_BIRTH |
|1.2.6  | 2016-06-21   |Removed Content-Length header for HTTP GET APIs |
|1.2.5  | 2016-05-18   |Added value "STUDENT_ID" to parameter "idSubType"<br />Removed TLS_DHE ciphers |
|1.2.4  | 2016-02-09   |Added reject reason code 111 for MISMATCH_PRINTED_BARCODE_DATA |
|1.2.3  | 2015-11-17   |Added parameter "idSubtype" to Netverify Retrieval |
|1.2.2  | 2015-10-21   |Added ECDHE ciphers to supported cipher suites |
|1.2.1  | 2015-09-23   |Added parameter "country" and "customDocumentCode" for Netverify Multi Document<br/>Retrieval |
|1.2.0  | 2015-09-09   |Added Netverify Multi Document Retrieval<br />Added second set of API credentials for retrieving transaction data |
|1.1.2  | 2015-08-11   |Added new classifier "face" for retrieving scan images |
|1.1.1  | 2015-07-14   |Added value "XKX" for Kosovo to parameters "country" and "issuingCountry"<br />Added value "Kosovo" to parameter "usState" |
|1.1.0  | 2015-06-30   |Introduced EU data center |
|1.0.4  | 2015-04-08   |Added reject reason code 109 for PUNCHED_DOCUMENT |
|1.0.3  | 2015-03-24   |Removed cipher TLS_RSA_WITH_RC4_128_SHA due to RC4 deprecation |
|1.0.2  | 2014-03-05   |Added sources WEB and REDIRECT<br />Document type not mandatory anymore |
|1.0.1  | 2014-09-25   |Changed parameter "maskhint" to apply for credit cards only |
|1.0.0  | 2014-08-19   |Initial release |

# Usage

The Netverify Retrieval API should be used as a fallback, if your server was not able to process the callback which is the authoritative answer from Jumio.

Permanent, scheduled calls or bulk requests should be avoided, otherwise Jumio can occasionally restrict API access.

#### Best practice:
If your server was not able to process the callback for a scan, call the status retrieval API.
If the scan status is done or failed (not pending), retrieve scan details and image(s) once.

# Authentication and header

**Authentication:** Each API call is protected. To access it, use HTTP Basic Authentication with your API token as the "userid" and your API secret as the "password". Log into your Jumio customer portal, and you can find your API token and API secret on the "Settings" page under "API credentials".

**Note:** You have the opportunity to generate a second set of API credentials for retrieving transaction data in your customer portal under "API credentials - Transaction administration APIs".

**Header:** The following parameters are mandatory in the "header" section of your request.
-	`Accept: application/json`<br />
(Except for "Retrieving specific image")
-	`User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`<br />
(e.g. MyCompany MyApp/1.0.0, change this to reflect your company)

**TLS handshake:** The TLS protocol is required (see [Supported cipher suites](#supported-cipher-suites) chapter) and we strongly recommend using the latest version.

**Note:** Calls with missing or suspicious headers, suspicious parameter values, or without HTTP Basic Authentication result in HTTP status code 403 Forbidden.

---

# Netverify Retrieval

## Retrieving scan status

Call the RESTful HTTP GET API below to receive the status of a scan by specifying the Jumio scan reference as a path parameter.

HTTP request method: **GET<br>
REST URL:** `https://netverify.com/api/netverify/v2/scans/<scanReference>`<br>
If your customer account is in the EU data center, use `lon.netverify.com` instead of `netverify.com`.

### Request parameter

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference**<br>(path parameter)| String|36|Jumio’s reference number of an existing scan from your account|

### Response

You receive a JSON response in case of success, or HTTP status code **404 Not Found** if the scan is not available or deleted.

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|timestamp| String| |Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|scanReference| String|36|Jumio’s reference number for each scan|
|status| String| |Possible states:<br />•	PENDING<br />•	DONE<br />•	FAILED|

### Sample request

```
GET https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```

### Sample response

```
{
"timestamp": "2014-08-13T12:08:02.068Z",
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
"status": "DONE"
}

```

## Retrieving scan details

Call the RESTful HTTP GET API below to receive document, transaction and verification details of a scan by specifying the Jumio scan reference as a path parameter.

HTTP request method: **GET<br>
REST URL:** `https://netverify.com/api/netverify/v2/scans/<scanReference>/data`<br>
If your customer account is in the EU data center, use `lon.netverify.com` instead of `netverify.com`.

### Request parameter

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference** <br>(path parameter)| String|36|Jumio’s reference number of an existing scan from your account|

### Response

You receive a JSON response in case of success, or HTTP status code **404 Not Found** if the scan is not available or deleted.

**Note:** Mandatory JSON parameters are highlighted bold.

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**timestamp**| String| |Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference**| String|36|Jumio’s reference number for each scan|
|**document**| Object| |Same parameters as listed in the section [Retrieving document data only](#retrieving-document-data-only) but without timestamp and scanReference|
|**transaction**| Object| |Same parameters as listed in the section [Retrieving transaction data only](#retrieving-transaction-data-only) but without timestamp and scanReference|
|verification| Object| |Same parameters as listed in the section [Retrieving verification data only](#retrieving-verification-data-only) but without timestamp and scanReference|

### Sample request

```
GET https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/data HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```

### Sample response

```
{
"timestamp": "2014-08-14T08:16:20.845Z",
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
 "document": {
"type": "PASSPORT",
"dob": "1990-01-01",
"expiry": "2022-12-31",
"firstName": "FIRSTNAME",
"issuingCountry": "USA",
"lastName": "LASTNAME",
"number": "P1234",
"status": "APPROVED_VERIFIED"
},
 "transaction": {
"clientIp": "xxx.xxx.xxx.xxx",
"customerId": "CUSTOMERID",
"date": "2014-08-10T10:27:29.494Z",
"source": "WEB_UPLOAD",
"status": "DONE"
},
 "verification": {
"mrzCheck": "OK"
}
}
```

## Retrieving document data only

Call the RESTful HTTP GET API below to receive document related data of a scan by specifying the Jumio scan reference as a path parameter.

HTTP request method: **GET**<br>
**REST URL:** `https://netverify.com/api/netverify/v2/scans/<scanReference>/data/document`<br>
If your customer account is in the EU data center, use `lon.netverify.com` instead of `netverify.com`.

### Request parameter

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference** <br>(path parameter)| String|36|Jumio’s reference number of an existing scan from your account|

### Response

You receive a JSON response in case of success, or HTTP status code **404 Not Found** if the scan is not available, pending or deleted.

**Note:** Mandatory JSON parameters are highlighted bold.

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**timestamp**| String| |Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference**| String|36|Jumio’s reference number for each scan|
|**status**| String| |Netverify:<br>• APPROVED_VERIFED<br>• DENIED_FRAUD<br>• DENIED_UNSUPPORTED_ID_TYPE<br>• DENIED_UNSUPPORTED_ID_COUNTRY<br>• DENIED_NAME_MISMATCH<br>• ERROR_NOT_READABLE_ID<br>• NO_ID_UPLOADED |
|type| String| |Netverify:<br>•	PASSPORT<br>• DRIVING_LICENSE<br>• ID_CARD<br>• UNSUPPORTED |
|idSubtype| String|255|Possible subtypes if idType = ID_CARD<br>•	NATIONAL_ID<br>• CONSULAR_ID<br>• ELECTORAL_ID<br>- RESIDENT_PERMIT_ID<br>• TAX_ID (only supported for PHL)<br>• STUDENT_ID (only supported for POL)<br>• PASSPORT_CARD_ID (only supported for IRL and USA)<br>• MILITARY_ID (only supported for GRC)<br>• OTHER_ID<br>• VISA (only supported for USA)<br>• UNKNOWN<br><br>Possible subtypes if idType = DRIVING_LICENSE<br>• LEARNING_DRIVING_LICENSE (only supported for GBR, IRL, BEL and CAN)<br><br>Possible subtypes if idType = PASSPORT<br>- E_PASSPORT (only for mobile)|
|issuingCountry|String|3|Possible countries:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>- XKX (Kosovo)|
|firstName|String|255|First name of the customer|
|lastName|String|255|Last name of the customer|
|dob|String||Date of birth in the format YYYY-MM-DD|
|expiry|String||Date of expiry in the format YYYY-MM-DD|
|issuingDate|String|255|Issue date in the format YYYY-MM-DD|
|number|String|255|Identification number of the document|
|usState|String|255|Possible values if idType = PASSPORT or ID_CARD:<br>• Last two characters of [ISO 3166-2:US](http://en.wikipedia.org/wiki/ISO_3166-2:US) state code<br>• [ISO 3166-1](http://en.wikipedia.org/wiki/ISO_3166-1) country name<br>• Kosovo<br><br>If idType = DRIVING_LICENSE:<br>• Last two characters of [ISO 3166-2:US](http://en.wikipedia.org/wiki/ISO_3166-2:US) state code |
|personalNumber|String|255|Personal number of the document|
|optionalData1|String|255|Optional field of MRZ line 1|
|optionalData2|String|255|Optional field of MRZ line 2|
|address|Object||Address in US, EU or Raw format, see tables below|

#### US address format

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|city| String|255 |City|
|stateCode| String|6 |[ISO 3166-2](http://en.wikipedia.org/wiki/ISO_3166-2) state code|
|streetName| String|255 |Street name|
|streetSuffix|String|255 |Street suffix abbreviation<br>Examples: [US](http://www.gis.co.clay.mn.us/USPS.htm#suffix), [Canada](http://www.canadapost.ca/tools/pg/manual/PGaddress-e.asp#1423617), [Australia](https://auspost.com.au/media/documents/australia-post-addressing-standards-1999.pdf)|
|streetDirection| String|255 |Street direction abbreviation<br>Examples: US (E=EAST, W=WEST, N=NORTH, S=SOUTH), [Canada](http://www.canadapost.ca/tools/pg/manual/PGaddress-e.asp#1403220), [Australia](https://auspost.com.au/media/documents/australia-post-addressing-standards-1999.pdf)|
|streetNumber| String|255 |Street number|
|unitDesignator| String|255 |Unit designator abbreviation<br>Examples: [US](http://www.gis.co.clay.mn.us/USPS.htm#secunitdesig), [Canada](http://www.canadapost.ca/tools/pg/manual/PGaddress-e.asp#1380473), [Australia](https://auspost.com.au/media/documents/australia-post-addressing-standards-1999.pdf)|
|unitNumber| String|255 |Unit number|
|zip| String|255 |Zip code|
|zipExtension| String|255 |Zip extension|
|country| String|3 |Possible countries:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>• XKX (Kosovo)|

#### EU address format

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|city| String|255 |City|
|province| String|255 |Province|
|streetName| String|255 |Street name|
|streetNumber| String|255 |Street number|
|unitDetails| String|255 |Unit details|
|postalCode| String|255 |Postal code|
|country| String|3 |Possible countries:<br>- [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>- XKX (Kosovo)|

#### Raw address format

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|line1| String|255 |Line item1|
|line2| String|255 |Line item2|
|line3| String|255 |Line item3|
|line4| String|255 |Line item4|
|line5| String|255 |Line item5|
|country|String|3|Possible countries:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>• XKX (Kosovo)|
|postalCode|String|255|Postal code|
|city|String|255|City|


### Sample request

```
GET https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/data/document HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```

### Sample response

```
{
"type": "PASSPORT",
"dob": "1990-01-01",
"expiry": "2022-12-31",
"firstName": "FIRSTNAME",
"issuingCountry": "USA",
"lastName": "LASTNAME",
"number": "P1234",
"issuingDate": "2015-11-01",
"status": "APPROVED_VERIFIED"
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
"timestamp": "2014-08-14T09:05:47.394Z"
}
```

## Retrieving transaction data only

Call the RESTful HTTP GET API below to receive transaction related data of a scan by specifying the Jumio scan reference as a path parameter.

HTTP request method: **GET**<br>
**REST URL:** `https://netverify.com/api/netverify/v2/scans/<scanReference>/data/transaction`<br>
If your customer account is in the EU data center, use `lon.netverify.com` instead of `netverify.com`.

### Request parameter


|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference** <br>(path parameter)| String|36|Jumio’s reference number of an existing scan from your account|

### Response

You receive a JSON response in case of success, or HTTP status code **404 Not Found** if the scan is not available, pending or deleted.

**Note:** Mandatory JSON parameters are highlighted bold.

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**timestamp**| String| |Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference**| String|36 |Jumio’s reference number for each scan|
|**status**| String| |Possible states:<br>• PENDING<br>• DONE<br>- FAILED|
|**source**| String| |Netverify Web embedded:<br>• WEB<br>• WEB-CAM<br>- WEB-UPLOAD<br><br>Netverify Web redirect:<br>• REDIRECT<br>• REDIRECT-CAM<br>• REDIRECT-UPLOAD<br><br>performNetverify:<br>- API<br><br>Netverify Mobile:<br>• SDK |
|**date**| String| |Timestamp of the scan creation in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|clientIp| String| |IP address of the client in the format xxx.xxx.xxx.xxx|
|customerId| String|255 |ID of the customer|
|merchantScanReference| String|255 |Your reference for each scan|
|merchantReportingCriteria| String|255 |Your reporting criteria for each scan|

### Sample request

```
GET https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/data/transaction HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```

### Sample response

```
"clientIp": "xxx.xxx.xxx.xxx",
"customerId": "CUSTOMERID",
"date": "2014-08-10T10:27:29.494Z",
"source": "WEB_UPLOAD",
"status": "DONE"
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
"timestamp": "2014-08-14T10:06:13.610Z"
```


## Retrieving verification data only

Call the RESTful HTTP GET API below to receive verification related data of a scan by specifying the Jumio scan reference as a path parameter.

HTTP request method: **GET**<br>
**REST URL:** `https://netverify.com/api/netverify/v2/scans/<scanReference>/data/verification`<br>
If your customer account is in the EU data center, use `lon.netverify.com` instead of `netverify.com`.

### Request parameter

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference** <br>(path parameter)| String|36|Jumio’s reference number of an existing scan from your account|

### Response

You receive a JSON response in case of success, or HTTP status code **404 Not Found** if the scan is not available, pending, deleted or of source Netverify Multi Document Legacy.

**Note:** Mandatory JSON parameters are highlighted bold.

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**timestamp**| String| |Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference**| String|36 |Jumio’s reference number for each scan|
|mrzCheck|String||Possible values:<br>• OK<br>• NOT_OK<br>• NOT_AVAILABLE|
|faceMatch|String||Face match percentage 0-100|
|rejectReason|Object||Reject reason, see tables below|
|identityVerification|Object||Identity verification, see table below|

### Reject reason

|Parameter "rejectReason" | Type   | Max. length    | Description|
|:------------------------|:--------|:--------|:------------|
|rejectReasonCode |String| 5  |see below |
|rejectReasonDescription |String |255  |Possible codes and descriptions for verification status DENIED_FRAUD:<br>100	MANIPULATED\_DOCUMENT<br/>105	FRAUDSTER<br/>106	FAKE<br/>107	PHOTO\_MISMATCH<br/>108	MRZ\_CHECK\_FAILED<br/>109	PUNCHED\_DOCUMENT<br/>110	CHIP\_DATA\_MANIPULATED (only available for ePassport)<br/>111	MISMATCH\_PRINTED\_BARCODE_DATA<br><br>Possible codes and descriptions for verificationStatus = ERROR\_NOT\_READABLE\_ID:<br/>102	PHOTOCOPY\_BLACK\_WHITE<br/>103	PHOTOCOPY\_COLOR<br/>104	DIGITAL\_COPY<br/>200	NOT\_READABLE\_DOCUMENT<br/>201	NO\_DOCUMENT<br/>202	SAMPLE\_DOCUMENT<br/>206	MISSING\_BACK<br/>207	WRONG\_DOCUMENT\_PAGE<br/>209	MISSING\_SIGNATURE<br/>210	CAMERA\_BLACK\_WHITE<br/>211	DIFFERENT\_PERSONS\_SHOWN<br/>300	MANUAL\_REJECTION|
|rejectReasonDetails |Object  |   |Reject reason details as JSON array containing JSON objects if rejectReasonCode = 100 or 200, see table below |


### Reject reason details

|Parameter "rejectReasonDetails" |  Type    | Max. length    | Description|
|:-------------------------------|:---------|:---------------|:------------|
|detailsCode   |String | 5 | see below |
|detailsDescription|String|255| Possible codes and description details for rejectReasonCode = 100:<br/>1001	PHOTO<br/>1002	DOCUMENT\_NUMBER<br>1003	EXPIRY<br/>1004	DOB<br/>1005	NAME<br/>1006	ADDRESS<br/>1007	SECURITY\_CHECKS<br/>1008	SIGNATURE<br>1009	PERSONAL\_NUMBER<br><br>Possible codes and description details for rejectReasonCode = 200:<br/>2001	BLURRED<br/>2002	BAD\_QUALITY<br/>2003	MISSING\_PART\_DOCUMENT<br/>2004	HIDDEN\_PART\_DOCUMENT<br/>2005	DAMAGED\_DOCUMENT |

### Identity verification

|Parameter "identityVerification"       | Max. length    | Description|
|:---------------|:--------|:------------|
|similarity |  |Possible values:<br/>• MATCH<br />• NO_MATCH<br />• NOT_POSSIBLE|
|validity  |  |Possible values:<br/>• TRUE<br />• FALSE |
|reason   |  |Provided if validity = FALSE<br/>Possible values:<br />• SELFIE\_CROPPED\_FROM\_ID<br />•	ENTIRE\_ID\_USED\_AS\_SELFIE<br />•	MULTIPLE\_PEOPLE<br />•	SELFIE\_IS\_SCREEN\_PAPER\_VIDEO<br />•	SELFIE\_MANIPULATED<br />• AGE\_DIFFERENCE\_TOO\_BIG<br />•	NO\_FACE\_PRESENT<br />•	FACE\_NOT\_FULLY\_VISIBLE<br />• BAD\_QUALITY|


### Sample request

```
GET https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/data/verification HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```

### Sample response

```
{
"mrzCheck": "OK",
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
"timestamp": "2014-08-14T10:56:19.088Z"
}
```

## Retrieving available images

Call the RESTful HTTP GET API below to receive available images of a scan by specifying the Jumio scan reference as a path parameter.

HTTP request method: **GET<br>
REST URL:** `https://netverify.com/api/netverify/v2/scans/<scanReference>/images`<br>
If your customer account is in the EU data center, use `lon.netverify.com` instead of `netverify.com`.

### Request parameter

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference** <br>(path parameter)| String|36|Jumio’s reference number of an existing scan from your account|


### Response

You receive a JSON response in case of success, or HTTP status code **404 Not Found** if the scan is not available, pending or deleted.

**Note:** Mandatory JSON parameters are highlighted bold.

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**timestamp**| String| |Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference**| String|36 |Jumio’s reference number for each scan|
|images|JSON Array/Object||Available image/s see table below|

|Parameter "images"       | Type    | Description|
|:---------------|:--------|:------------|
|**classifier**| String| Netverify:<br/>•	front<br/>•	face<br/>•	back|
|**href**| String |REST URL to retrieve specific image (see [Retrieving specific image](#retrieving-specific-image) section)|
|maskhint| String |For credit cards:<br />•	masked<br>•	unmasked<br/>|

### Sample request

```
GET https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic

```

### Sample response

```
{
"timestamp": "2014-08-14T11:22:20.182Z",
 "images": [
 {
"classifier": "back",
"href": "https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/back"
},
 {
"classifier": "front",
"href": "https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/front"
},
 {
"classifier": "face",
"href": "https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/face"
 }
],
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

## Retrieving specific image

Call the RESTful HTTP GET API below to receive a specific image of a scan by specifying the Jumio scan reference as a path parameter.

To receive the unmasked credit card image, append the query parameter `maskhint=unmasked`. By default, retrieval of unmasked credit card images is disabled (HTTP status code **403 Forbidden**). If you want to enable it please contact Jumio Support. Retrieving unmasked credit card images might impose additional security requirements on your systems depending if you already store/transmit/process credit card data on your systems.

In case you are unsure about the ramifications of retrieving unmasked images regarding PCI DSS please refer to "[Information Supplement: PCI DSS E-commerce Guidelines, version 2.0, January 2013](https://www.pcisecuritystandards.org/pdfs/PCI_DSS_v2_eCommerce_Guidelines.pdf)" and/or contact your acquirer and/or contact a PCI DSS QSA (Qualified Security Assessor).

HTTP request method: **GET<br>
REST URL:** `https://netverify.com/api/netverify/v2/scans/<scanReference>/images/<classifier>`<br>
If your customer account is in the EU data center, use `lon.netverify.com` instead of `netverify.com`.

### Request parameter

**Note:** Mandatory JSON parameters are highlighted bold.

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference** <br>(path parameter)| String|36|Jumio’s reference number of an existing scan from your account|
|**classifier** (path parameter)| String| |Netverify:<br/>•	front<br/>•	face<br/>•	back|
|maskhint (path parameter)| String| |For credit cards:<br/>•	masked (default)<br/>•	unmasked|

### Response

You receive a JPG or PNG image in case of success with the according header (e.g. `Content-Type: image/jpeg`), or HTTP status code **404 Not Found** if the scan is not available, pending, deleted or not containing the specified image.

### Sample request

```
GET https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/front HTTP/1.1
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```

---


# Netverify Multi Document Retrieval

## <a name="multi-retrieving-scan-status"></a>Retrieving scan status

Call the RESTful HTTP GET API below to receive the status of a scan by specifying the Jumio scan reference as a path parameter.

HTTP request method: **GET**<br>
**REST URL:** `https://retrieval.netverify.com/api/netverify/v2/documents/<scanReference>`<br>
If your customer account is in the EU data center, use `retrieval.lon.netverify.com` instead of `retrieval.netverify.com`.

### Request parameter

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference** <br>(path parameter)| String|36|Jumio’s reference number of an existing scan from your account|

### Response

You receive a JSON response in case of success, or HTTP status code **404 Not Found** if the scan is not available or deleted.

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|timestamp| String||Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|scanReference|String|36|Jumio’s reference number for each scan|
|status|String||Possible states:<br>• DONE<br>• FAILED|

### Sample Request
```
GET https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```

### Sample Response
```
{
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
"timestamp": "2015-08-13T12:08:02.068Z",
"status": "DONE"
}
```



## <a name="multi-retrieving-scan-details"></a>Retrieving scan details

Call the RESTful HTTP GET API below to receive document and transaction details of a scan by specifying the Jumio scan reference as a path parameter.

HTTP request method: **GET**<br>
**REST URL:** `https://retrieval.netverify.com/api/netverify/v2/documents/<scanReference>/data`<br>
If your customer account is in the EU data center, use `retrieval.lon.netverify.com` instead of `retrieval.netverify.com`.

### Request parameter

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference** <br>(path parameter)| String|36|Jumio’s reference number of an existing scan from your account|

### Response

You receive a JSON response in case of success, or HTTP status code **404 Not Found** if the scan is not available, pending or deleted.

**Note:** Mandatory JSON parameters are highlighted bold.

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|timestamp|String||Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|scanReference|String|36|Jumio’s reference number for each scan|
|document|Object||Same parameters as listed in the section [Retrieving document data only](#multi-retrieving-document-data-only) but without timestamp and scanReference|
|transaction|Object||Same parameters as listed in the section [Retrieving transaction data only](#multi-retrieving-transaction-data-only) but without timestamp and scanReference|

### Sample request
```
GET https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/data HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```


### Sample response
```
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
"timestamp": "2015-08-14T08:16:20.845Z",
"document": {
"status": "EXTRACTED",
"type": "SSC",
"extractedData": {
"firstName": "FIRSTNAME",
"lastName": "LASTNAME",
"ssn": "12341234",
"signatureAvailable": true
}
},
"transaction": {
"status": "DONE",
"merchantReportingCriteria": "YOURMERCHANTREPORTINGCRITERIA",
"merchantScanReference": "YOURSCANREFERENCE",
"customerId": "CUSTOMERID",
"source": "DOC_UPLOAD"
}
}
```


## <a name="multi-retrieving-document-data-only"></a>Retrieving document data only

Call the RESTful HTTP GET API below to receive document related data of a scan by specifying the Jumio scan reference as a path parameter.

HTTP request method: **GET**<br>
**REST URL:** `https://retrieval.netverify.com/api/netverify/v2/documents/<scanReference>/data/document`<br>
If your customer account is in the EU data center, use `retrieval.lon.netverify.com` instead of `retrieval.netverify.com`.

### Request parameter

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference** <br>(path parameter)| String|36|Jumio’s reference number of an existing scan from your account|

### Response

You receive a JSON response in case of success, or HTTP status code **404 Not Found** if the scan is not available, pending or deleted.

**Note:** Mandatory JSON parameters are highlighted bold.

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**timestamp**|String||Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference**| String|36|Jumio’s reference number for each scan|
|**status**|String||Possible states:<br>•	UPLOADED (default)<br>•	EXTRACTED if type = SSC, country = USA and US social security card provided<br>•	DISCARDED if type = SSC, country = USA and no US social security card provided |
|type|String||Possible types:<br>• BS (Bank statement, front side)<br>•	IC (Insurance card, front side)<br>•	UB (Utility bill, front side)<br>•	CAAP (Cash advance application, front and back side)<br>•	CRC (Corporate resolution certificate, front and back side)<br>•	CCS (Credit card statement, front and back side)<br>•	LAG (Lease agreement, front and back side)<br>•	LOAP (Loan application, front and back side)<br>•	MOAP (Mortgage application, front and back side)<br>•	TR (Tax return, front and back side)<br>•	VT (Vehicle title, front side)<br>•	VC (Voided check, front side)<br>•	STUC (Student card, front side)<br>•	HCC (Health care card, front side)<br>•	CB (Council bill, front side)<br>•	SENC (Seniors card, front side)<br>•	MEDC (Medicare card, front side)<br>•	BC (Birth certificate, front side)<br>•	WWCC (Working with children check, front side)<br>•	SS (Superannuation statement, front side)<br>•	TAC (Trade association card, front side)<br>•	SEL (School enrolment letter, front side)<br>•	PB (Phone bill, front side)<br>•	USSS (US social security card, front side)<br>•	SSC (Social security card, front side)<br>•	CUSTOM (Custom document type) |
|country|String|3|Possible countries:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>•	XKX (Kosovo)|
|customDocumentCode|String|100|Your custom document code (maintained in your Jumio customer portal) if type = CUSTOM|
|extractedData|Object||Extracted data if type = SSC, country = USA and status = EXTRACTED, see table below|

|Parameter "extractedData" | Type    | Max. length| Description|
|:-------------------------|:--------|:------------|:------------|
|signatureAvailable|String||"true" if signature available, otherwise "false"|
|ssn|String|255|Social security number if readable|
|firstName|String|255|First name if readable|
|lastName|String|255|Last name if readable|

### Sample request
```
GET https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/data/document HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```

### Sample Response
```
{
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
"timestamp": "2015-08-14T09:05:47.394Z",
"status": "EXTRACTED",
"type": "SSC",
"country": "USA",
"extractedData": {
"firstName": "FIRSTNAME",
"lastName": "LASTNAME",
"ssn": "12341234",
"signatureAvailable": true
}
```



## <a name="multi-retrieving-transaction-data-only"></a>Retrieving transaction data only

Call the RESTful HTTP GET API below to receive transaction related data of a scan by specifying the Jumio scan reference as a path parameter.

HTTP request method: **GET**<br>
**REST URL:** `https://retrieval.netverify.com/api/netverify/v2/documents/<scanReference>/data/transaction`<br>
If your customer account is in the EU data center, use `retrieval.lon.netverify.com` instead of `retrieval.netverify.com`.

### Request parameter

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference** (path parameter)| String|36|Jumio’s reference number of an existing scan from your account|

### Response

You receive a JSON response in case of success, or HTTP status code **404 Not Found** if the scan is not available, pending or deleted.

**Note:** Mandatory JSON parameters are highlighted bold.

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**timestamp**| String||Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference**|String|36|Jumio’s reference number for each scan|
|**status**|String||Possible states:<br>•	DONE<br>•	FAILED |
|**source**|String||Possible status:<br>•	DOC_API<br>• DOC_UPLOAD |
|merchantReportingCriteria|String|255|Your reporting criteria for each scan|
|merchantScanReference|String|255|Your reference for each scan|
|customerId|String|255|ID of the customer|

### Sample request
```
GET https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/data/transaction HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```

### Sample response
```
{
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
"timestamp": "2015-08-14T10:06:13.610Z",
"status": "DONE",
"merchantReportingCriteria": "YOURMERCHANTREPORTINGCRITERIA",
"merchantScanReference": "YOURSCANREFERENCE",
"customerId": "CUSTOMERID",
"source": "DOC_UPLOAD"
}
```

## <a name="multi-retrieving-available-images"></a>Retrieving available images

Call the RESTful HTTP GET API below to receive available images of a scan by specifying the Jumio scan reference as a path parameter.

HTTP request method: **GET<br>
REST URL:** `https://retrieval.netverify.com/api/netverify/v2/documents/<scanReference>/pages`<br>
If your customer account is in the EU data center, use `retrieval.lon.netverify.com` instead of `retrieval.netverify.com`.

### Request parameter

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference** <br>(path parameter)| String|36|Jumio’s reference number of an existing scan from your account|

### Response

You receive a JSON response in case of success, or HTTP status code **404 Not Found** if the scan is not available, pending, deleted or of source Netverify Multi Document Legacy.

**Note:** Mandatory JSON parameters are highlighted bold.

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**timestamp**| String| |Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**scanReference**| String|36 |Jumio’s reference number for each scan|
|images|JSON Array/Object||Available image/s see table below|

|Parameter "images"       | Type    | Description|
|:---------------|:--------|:------------|
|**classifier**| Integer| Page number specified when uploading the page|
|**href**| String |REST URL to retrieve specific image (see [Retrieving specific image section](#multi-retrieving-specific-image))|

### Sample request

```
GET https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/pages HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```

### Sample request

```
{
"timestamp": "2015-08-14T11:22:20.182Z",
"images": [
{
"classifier": 1,
"href": "https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/pages/1"
},
{
"classifier": 2,
"href": "https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/pages/2"
}
],
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```



## <a name="multi-retrieving-specific-image"></a>Retrieving specific image

Call the RESTful HTTP GET API below to receive a specific image of a scan by specifying the Jumio scan reference as a path parameter.

HTTP request method: **GET**<br>
**REST URL:**  `https://retrieval.netverify.com/api/netverify/v2/documents/<scanReference>/pages/<page_number>`<br>
If your customer account is in the EU data center, use `lon.netverify.com` instead of `netverify.com`.

### Request parameter

**Note:** Mandatory JSON parameters are highlighted bold.

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference** <br>(path parameter)| String|36|Jumio’s reference number of an existing scan from your account|
|**page_number**| String|36|Page number specified when uploading the page|

### Response

You receive a JPG or PNG image in case of success with the according header (e.g. `Content-Type: image/jpeg`), or HTTP status code **404 Not Found** if the scan is not available, pending, deleted or not containing the specified image.

### Sample request

```
GET https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/pages/1 HTTP/1.1
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```

---

## Supported cipher suites
The following cipher suites (listed in server-preferred order) are supported by Jumio during the TLS handshake:

* TLS\_ECDHE\_RSA\_WITH\_AES\_128\_GCM\_SHA256
* TLS\_ECDHE\_RSA\_WITH\_AES\_128\_CBC\_SHA256
* TLS\_ECDHE\_RSA\_WITH\_AES\_256\_GCM\_SHA384
* TLS\_ECDHE\_RSA\_WITH\_AES\_256\_CBC\_SHA384
* TLS\_RSA\_WITH\_AES\_256\_GCM\_SHA384
* TLS\_RSA\_WITH\_AES\_256\_CBC\_SHA256
* TLS\_RSA\_WITH\_AES\_128\_GCM\_SHA256
* TLS\_RSA\_WITH\_AES\_128\_CBC\_SHA256
* TLS\_ECDHE\_RSA\_WITH\_AES\_128\_CBC\_SHA
* TLS\_ECDHE\_RSA\_WITH\_AES\_256\_CBC\_SHA
* TLS\_RSA\_WITH\_AES\_128\_CBC\_SHA
* TLS\_RSA\_WITH\_AES\_256\_CBC\_SHA

## Copyright
&copy; Jumio Corp. 268 Lambert Avenue, Palo Alto, CA 94306
