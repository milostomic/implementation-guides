![Jumio](/images/netverify.png)

# Netverify Delete API Implementation Guide

This guide illustrates how to implement the Netverify Delete API.

### Revision history

| Date    | Description|
|:--------|:------------|
| 2016-05-18  |Removed TLS\_DHE ciphers|
| 2015-10-20  |Added ECDHE ciphers to supported cipher suites|
| 2015-09-23  |Initial release|

---

## Table of Contents

- [Deleting a Netverify scan](#deleting-a-netverify-scan)
- [Deleting a Document Verification scan](#deleting-a-document-verification-scan)
- [Supported Cipher Suites](#supported-cipher-suites)


---
# Deleting a Netverify Scan

Call the RESTful HTTP **DELETE** API below to remove sensitive data and image(s) of a scan by specifying the Jumio scan reference as a path parameter.

HTTP request method: **DELETE**<br>
**REST URL:** `https://netverify.com/api/netverify/v2/scans/<scanReference>`
If your customer account is in the EU data center, use `lon.netverify.com` instead of `netverify.com`.

**Authentication:** Each API call is protected. To access it, use HTTP Basic Authentication with your API token as the "userid" and your API secret as the "password". You can find your API token and API secret by logging into your Jumio customer portal and navigating to the "Settings" page and clicking on the "API credentials" tab.

**Note:** You have the opportunity to generate a second set of API credentials for deleting scans. To do so, log into your Jumio customer portal and navigate to "API credentials - Transaction administration APIs".

**Header:** The following parameters are mandatory in the "header" section of your request.<br>
-	`Accept: application/json`<br />
-	`User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`<br /><br>
The value for **User-Agent** must contain a reference to your business or entity for Jumio to be able to identify your requests. (e.g. YourCompanyName YourAppName/1.0.0). Without a proper User-Agent header, Jumio will take longer to diagnose API issues.

**TLS handshake:** The TLS protocol is required (see [Supported cipher suites](/netverify/supported-cipher-suites.md)) and we strongly recommend using the latest version.

**Note:** Calls with missing or suspicious headers, suspicious parameter values, or without HTTP Basic Authentication result in HTTP status code, **403 Forbidden**.

### Request Parameter

|Parameter       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference (path parameter)** *| String|36|Jumio’s reference number of an existing scan from your account|

### Response

You will receive the HTTP status code, **200 OK**, to confirm success.

### Sample Request

```
DELETE https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```

---

# Deleting a Document Verification Scan

Call the RESTful HTTP **DELETE** API below to remove sensitive data and image(s) of a Document Verification scan by specifying the Jumio scan reference as a path parameter.

HTTP request method: **DELETE**
**REST URL:** `https://retrieval.netverify.com/api/netverify/v2/documents/<scanReference>`
If your customer account is in the EU data center, use `retrieval.lon.netverify.com` instead of `retrieval.netverify.com`.

**Authentication:** Each API call is protected. To access it, use HTTP Basic Authentication with your API token as the "userid" and your API secret as the "password". You can find your API token and API secret by logging into your Jumio customer portal and navigating to the "Settings" page and clicking on the "API credentials" tab.

**Note:** You have the opportunity to generate a second set of API credentials for deleting scans. To do so, log into your Jumio customer portal and navigate to "API credentials - Transaction administration APIs".

**Header:** The following parameters are mandatory in the "header" section of your request.<br/>
-	`Accept: application/json`<br/>
-	`User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`<br/><br/>
The value for **User-Agent** must contain a reference to your business or entity for Jumio to be able to identify your requests. (e.g. YourCompanyName YourAppName/1.0.0). Without a proper User-Agent header, Jumio will take longer to diagnose API issues.

**TLS handshake:** The TLS protocol is required (see [Supported cipher suites](/netverify/supported-cipher-suites.md)) and we strongly recommend using the latest version.

**Note:** Calls with missing or suspicious headers, suspicious parameter values, or without HTTP Basic Authentication result in HTTP status code **403 Forbidden**.

### Request Parameter

|Parameter       | Type    | Max. Length| Description|
|:--------|:--------|:--------|:---------|
|**scanReference (path parameter)** *| String|36|Jumio’s reference number of an existing scan from your account|

### Response

You receive HTTP status code **200 OK** in case of success.

### Sample Request

```
DELETE https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```

---

# Supported Cipher Suites
Jumio supported cipher suites during the TLS handshake.<p>
[View Supported Cipher Suites](/netverify/supported-cipher-suites.md)

---
&copy; Jumio Corp. 268 Lambert Avenue, Palo Alto, CA 94306
