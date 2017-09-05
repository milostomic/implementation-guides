![Jumio](/images/fastfill.png)

# Fastfill API Implementation Guide

This is a reference manual and configuration guide for the Fastfill API. It illustrates how to retrieve information from an ID image immediately.

## Contact

If you have any questions regarding our implementation guide please contact Jumio Customer Service at support@jumio.com or https://support.jumio.com. The Jumio online helpdesk contains a wealth of information regarding our service including demo videos, product descriptions, FAQs and other things that may help to get you started with Jumio. Check it out at: https://support.jumio.com.

## Table of Content

- [Using the Fastfill API](#using-the-fastfill-api)
- [Supported IDs](#supported-ids)
- [Supported cipher suites](#supported-cipher-suites)

## Release notes

|Version       | Date    | Description|
|:-------------|:---------------|:------------|
|1.0.6  | 2016-11-29  |Updated Supported IDs|
|1.0.5  | 2016-05-18  |Removed TLS_DHE ciphers|
|1.0.4  | 2016-03-30  |Added parameter "issueDate"|
|1.0.3  | 2015-11-24 |Added parameter "gender"|
|1.0.2  | 2015-10-21  |Added driver license support<br>Added ECDHE ciphers to supported cipher suites|
|1.0.1  | 2015-07-14  |Added value "XKX" for Kosovo to parameters "country", "issuingCountry" and "originatingCountry"|
|1.0.0  | 2015-06-30  |Initial release|

# Using the Fastfill API

Call the RESTful HTTP POST API **fastfill** from your server with the below parameters to create a transaction for each user. Simply submit front and/or back side image(s) of the customer’s document with a maximum file size of 5 MB and you will receive the ID information as JSON response immediately.
**Note:** Requests originating from the client are not supported (CORS is disabled) and the API credentials must never be exposed on the client side.

HTTP request method: **POST**<br>
**REST URL:** `https://netverify.com/api/netverify/v2/fastfill`<br>
If your customer account is in the EU data center, use `lon.netverify.com` instead of `netverify.com`.

**Authentication:** The API call is protected. To access it, use HTTP Basic Authentication with your API token as the "userid" and your API secret as the "password". Log into your Jumio customer portal, and you can find your API token and API secret on the "Settings" page under "API credentials".

**Header:** The following parameters are mandatory in the "header" section of your request.<br>
-	`Accept: application/json`
- `Content-Type: multipart/form-data`
-	`Content-Length: xxx` (RFC-2616)
-	`User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION` <br>
(e.g. MyCompany MyApp/1.0.0, change this to reflect your company)

**TLS handshake:** The TLS protocol is required (see [Supported cipher suites](#supported-cipher-suites) chapter) and we strongly recommend using the latest version.

**Note:** Calls with missing or suspicious headers, suspicious parameter values, or without HTTP Basic Authentication will result in HTTP status code **403 Forbidden**.

### Request parameter

**Note:** Mandatory parameters are highlighted bold.

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**metadata**|JSON||Metadata of the customer's document as JSON objects, see table below|
|**frontsideImage**|JPEG or PNG|Max. 5 MB|Front side image of the customer's document<br> <strong>Mandatory<br>-	if type = PASSPORT<br>-	If type = ID_CARD or DRIVING_LICENSE and backsideImage not provided</strong> |
|**backsideImage**|JPEG or PNG|Max. 5 MB|Back side image of the customer's document if type = ID_CARD or DRIVING_LICENSE<br><strong>Mandatory if frontsideImage not provided</strong>|
|**type**|String||Possible types:<br>-	PASSPORT<br>-	ID_CARD<br>- DRIVING_LICENSE |
|**country**|String|3|Possible countries:<br>- [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>-	XKX (Kosovo) |

### Response parameter

**Note:** Mandatory parameters are highlighted bold.

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference**|String||Jumio’s reference number for each scan|
|type|String||Possible types:<br>-	PASSPORT<br>-	ID_CARD<br>- DRIVING_LICENSE|
|issueDate|String|255|Issue date in the format YYYY-MM-DD|
|issuingCountry|String|3|Possible countries:<br>- [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>-	XKX (Kosovo)|
|originatingCountry|||Possible countries:<br>- [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>-	XKX (Kosovo)|
|number|String|255|Identification number of the document|
|firstName|String|255|First name of the customer|
|lastName|String|255|Last name of the customer|
|dob|String|255|Date of birth in the format YYYY-MM-DD|
|expiry|String|255|Date of expiry in the format YYYY-MM-DD|
|optionalData1|String|255|Optional field of MRZ (TD1) line 1|
|optionalData2|String|255|Optional field of MRZ (TD1 and TD2) line 2|
|personalNumber|String|255|Personal number field of MRZ (MRP)|
|street|String|255|Street if type = DRIVING_LICENSE|
|city|String|255|City if type = DRIVING_LICENSE|
|state|String|255|State if type = DRIVING_LICENSE|
|postalCode|String|255|Postal code if type = DRIVING_LICENSE|
|gender|String|255|Possible values: <br>- M<br>-	F|

Optional parameters are returned
-	if available on the ID
-	if readable
-	as readable (can be invalid)

### Sample request

```
POST https://netverify.com/api/netverify/v2/fastfill HTTP/1.1
Accept: application/json
Content-Type: multipart/form-data;boundary=----xxxx
Content-Length: xxx
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
----xxxx
Content-Disposition: form-data; name="metadata"
Content-Type: application/json
{
"type": "PASSPORT",
"country": "USA"
}
----xxxx
Content-Disposition: form-data; name="frontsideImage"
Content-Type: image/jpeg
// Your image
----xxxx
```

### Sample response

```
{
"dob": "1990-01-01",
"expiry": "2022-12-31",
"firstName": "FIRSTNAME",
"issueDate": "2012-12-31",
"issuingCountry": "USA",
"lastName": "LASTNAME",
"number": "P1234",
"originatingCountry": "USA",
"personalNumber": "12345",
"scanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
"type": "PASSPORT"
}
```
---

# Supported IDs

The Fastfill API supports

- all Netverify supported passports.

- ID cards from the following countries: Albania, Argentina, Austria, Belgium, Bosnia & Herzegovina, Bulgaria, Cameroon, Chile, Czech Republic, Denmark, Dominican Republic, Ecuador, El Salvador, Estonia, Finland, France, Georgia, Germany, Guatemala, Hungary, Italy, Kazakhstan, Kenya, Latvia, Liechtenstein, Macedonia, Malta, Mexico, Moldova, Montenegro, Netherlands, Pakistan, Paraguay, Peru, Poland, Portugal, Romania, Serbia, Slovakia, Slovenia, Spain, Sweden, Switzerland, Turkey, United Arab Emirates, United Kingdom.

- Driver licenses from the following countries: Austria, Belgium, Czech Republic, Denmark, Finland, France, Germany, Ireland, Italy, Netherlands, Poland, Portugal, Romania, Slovakia, Spain, Sweden, United Kingdom, United States.<br>
PDF417 bar code extraction from driver licenses back side from USA and Canada.

---

# Supported cipher suites
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
