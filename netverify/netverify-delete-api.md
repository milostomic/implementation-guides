![Jumio](/images/netverify.png)

# Netverify Delete API Implementation Guide

This guide illustrates how to implement the Netverify Delete API.

## Contact

If you have any questions regarding our implementation guide please contact Jumio Customer Service at support@jumio.com or https://support.jumio.com. The Jumio online helpdesk contains a wealth of information regarding our service including demo videos, product descriptions, FAQs and other things that may help to get you started with Jumio. Check it out at: https://support.jumio.com.

## Table of Content

- [Deleting a Netverify scan](#deleting-a-netverify-scan)
- [Deleting a Netverify Multi Document scan](#deleting-a-netverify-multi-document-scan)
- [Supported cipher suites](#supported-cipher-suites)

## Release notes

|Version       | Date    | Description|
|:-------------|:--------|:------------|
|1.0.2  | 2016-05-18  |Removed TLS_DHE ciphers|
|1.0.1  | 2015-10-20  |Added ECDHE ciphers to supported cipher suites|
|1.0.0  | 2015-09-23  |Initial release|

# Deleting a Netverify scan

Call the RESTful HTTP **DELETE** API below to remove sensitive data and image(s) of a scan by specifying the Jumio scan reference as a path parameter.

HTTP request method: **DELETE**<br>
**REST URL:** `https://netverify.com/api/netverify/v2/scans/<scanReference>`
If your customer account is in the EU data center, use `lon.netverify.com` instead of `netverify.com`.

**Authentication:** Each API call is protected. To access it, use HTTP Basic Authentication with your API token as the "userid" and your API secret as the "password". Log into your Jumio customer portal, and you can find your API token and API secret on the "Settings" page under "API credentials".

**Note:** You have the opportunity to generate a second set of API credentials for deleting scans in your customer portal under "API credentials - Transaction administration APIs".

**Header:** The following parameters are mandatory in the "header" section of your request.<br>
-	`Accept: application/json`
-	`User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`
(e.g. MyCompany MyApp/1.0.0, change this to reflect your company)

**TLS handshake:** The TLS protocol is required (see [Supported cipher suites](#supported-cipher-suites) chapter) and we strongly recommend using the latest version.

**Note:** Calls with missing or suspicious headers, suspicious parameter values, or without HTTP Basic Authentication result in HTTP status code **403 Forbidden**.

### Request parameter

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|scanReference (path parameter)| String|36|Jumio’s reference number of an existing scan from your account|

### Response

You receive HTTP status code **200 OK** in case of success.

### Sample request

```
DELETE https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```

---

# Deleting a Netverify Multi Document scan

Call the RESTful HTTP **DELETE** API below to remove sensitive data and image(s) of a Netverify Multi Document scan by specifying the Jumio scan reference as a path parameter.

HTTP request method: **DELETE**
**REST URL:** `https://retrieval.netverify.com/api/netverify/v2/documents/<scanReference>`
If your customer account is in the EU data center, use `retrieval.lon.netverify.com` instead of `retrieval.netverify.com`.

**Authentication:** Each API call is protected. To access it, use HTTP Basic Authentication with your API token as the "userid" and your API secret as the "password". Log into your Jumio customer portal, and you can find your API token and API secret on the "Settings" page under "API credentials".

**Note:** You have the opportunity to generate a second set of API credentials for deleting scans in your customer portal under "API credentials - Transaction administration APIs".

**Header:** The following parameters are mandatory in the "header" section of your request.
-	`Accept: application/json`
-	`User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`
(e.g. MyCompany MyApp/1.0.0, change this to reflect your company)

**TLS handshake:** The TLS protocol is required (see [Supported cipher suites](#supported-cipher-suites) chapter) and we strongly recommend using the latest version.

**Note:** Calls with missing or suspicious headers, suspicious parameter values, or without HTTP Basic Authentication result in HTTP status code **403 Forbidden**.

### Request parameter

|Parameter       | Type    | Max. length| Description|
|:--------|:--------|:--------|:---------|
|scanReference (path parameter)| String|36|Jumio’s reference number of an existing scan from your account|

### Response

You receive HTTP status code **200 OK** in case of success.

### Sample request

```
DELETE https://netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx HTTP/1.1
Accept: application/json
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
