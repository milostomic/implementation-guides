
![Jumio](/images/bam_checkout.png)

# Credit Card Retrieval API Implementation Guide

This guide illustrates how to implement the Credit Card Retrieval API.


## Table of Contents

- [Retrieving credit card image](#retrieving-credit-card-image)
- [Retrieving credit card details](#retrieving-credit-card-details)
- [Supported cipher suites](#supported-cipher-suites)

---
# Release Notes

Date    | Description
:-------|:------------
2017-05-23|Added response parameter "merchantReportingCriteria" for retrieving <br>credit card details  
2016-07-19|Updated endpoint URLs  
2016-05-18|Removed TLS_DHE ciphers  
2015-10-21|Added ECDHE ciphers to supported cipher suites  
2015-06-30|Introduced EU data center  
2015-03-24|Removed cipher TLS_RSA_WITH_RC4_128_SHA due to RC4 deprecation
2014-10-07|Added unmasked retrieval of cardAccountNumber  
2014-06-10|Updated supported cipher suites during SSL handshake  
2014-04-23|Added supported cipher suites during SSL handshake (TLS required)  
2014-01-14|Changed location of API credentials due to redesigned customer portal  
2013-12-17| Initial release |

---
# Retrieving Credit Card Image

By calling the RESTful HTTP GET API below you receive the masked credit card image of a successful scan by specifying the Jumio scan reference as a path parameter.

To receive the unmasked image, append the query parameter `maskhint=unmasked`. By default, retrieval of unmasked images is disabled (HTTP status code 403 Forbidden). If you want to enable it, please contact <support@jumio.com>. Retrieving unmasked images might impose additional security requirements on your systems depending if you already store/transmit/process credit card data on your systems.

In case you are unsure about the ramifications of retrieving unmasked images regarding PCI DSS please refer to "[Information Supplement: PCI DSS E-commerce Guidelines, version 2.0, January 2013](https://www.pcisecuritystandards.org/pdfs/PCI_DSS_v2_eCommerce_Guidelines.pdf)" and/or contact your acquirer and/or contact a PCI DSS QSA (Qualified Security Assessor).

HTTP request method: **GET**<br>
**REST URL:** `https://bam-retrieval.jumio.com/api/netswipe/v1/scans/<scanReference>/images/front`<br>
If your customer account is in the EU data center, use `bam-retrieval.lon.jumio.com` instead of `bam-retrieval.jumio.com`.

**Authentication:** The API call is protected. To access it, use HTTP Basic Authentication with your API token as the "userid" and your API secret as the "password". Log into your Jumio customer portal, and you can find your API token and API secret on the "Settings" page under "API credentials".

**Header:** The following parameter is mandatory in the "header" section of your request.<br>
- `User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`<br>
(e.g. MyCompany MyApp/1.0.0, change this to reflect your company)

**TLS handshake:** The TLS protocol is required (see [Supported Cipher Suites](/netverify/supported-cipher-suites.md) chapter) and we strongly recommend using the latest version.

**Note:** Calls with missing or suspicious headers, suspicious parameter values, or without HTTP Basic Authentication will result in HTTP status code **403 Forbidden**.


### Request Parameters

**Note:** Mandatory parameters are marked with an asterisk * and highlighted bold.

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference (path parameter)** *| String|36|Jumio’s reference number of an existing scan from your account|
|maskint (query parameter) |String||Possible values:<br>• masked (default)<br>• unmasked|

### Response

You receive a JPEG image in case of success, or HTTP status code **404 Not Found** if the scan or the image is not available, which may take up to 5 minutes.

### Sample Request

```
GET https://bam-retrieval.jumio.com/api/netswipe/v1/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/images/front HTTP/1.1
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```
---

# Retrieving Credit Card Details

By calling the RESTful HTTP GET API below you receive the credit card data of successful scans by specifying the Jumio scan reference as a path parameter.

To receive unmasked card details, append the query parameter `maskhint=unmasked`. By default, retrieval of unmasked credit card details is disabled (HTTP status code **403 Forbidden**). If you want to enable it please contact <support@jumio.com>. Retrieving unmasked credit card details might impose additional security requirements on your systems depending if you already store/transmit/process credit card data on your systems.

*In case you are unsure about the ramifications of retrieving unmasked images regarding PCI DSS please refer to "[Information Supplement: PCI DSS E-commerce Guidelines, version 2.0, January 2013](https://www.pcisecuritystandards.org/pdfs/PCI_DSS_v2_eCommerce_Guidelines.pdf)" and/or contact your acquirer and/or contact a PCI DSS QSA (Qualified Security Assessor).*

HTTP request method: **GET**<br>
**REST URL:** `https://bam-retrieval.jumio.com/api/netswipe/v1/scans/<scanReference>/creditCard`<br>
If your customer account is in the EU data center, use `bam-retrieval.lon.jumio.com` instead of `bam-retrieval.jumio.com`.

**Authentication:** The API call is protected. To access it, use HTTP Basic Authentication with your API token as the "userid" and your API secret as the "password". Log into your Jumio customer portal, and you can find your API token and API secret on the "Settings" page under "API credentials".

**Header:** The following parameters are mandatory in the "header" section of your request.<br>
-	`Accept: application/json`
-	`User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`
(e.g. MyCompany MyApp/1.0.0, change this to reflect your company)

**TLS handshake:** The TLS protocol is required (see [Supported Cipher Suites](/netverify/supported-cipher-suites.md)) and we strongly recommend using the latest version.

**Note:** Calls with missing or suspicious headers, suspicious parameter values, or without HTTP Basic Authentication result in HTTP status code **403 Forbidden**.

## Request Parameters

**Note:** Mandatory parameters are marked with an asterisk * and highlighted bold.

|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference (path parameter)** *| String|36|Jumio’s reference number of an existing scan from your account|
|maskint (query parameter) |String||Possible values:<br>• masked (default)<br>• unmasked|

## Response Parameters

You receive a JSON response in case of success, or HTTP status code **404 Not Found** if the scan or the credit card data is not available, which may take up to 5 minutes. <br>

**Note:** Mandatory parameters are marked with an asterisk * and highlighted bold.


|Parameter       | Type    | Max. length| Description|
|:---------------|:--------|:------------|:----------|
|**jumioRequestReference** *|String |36|Jumio’s reference number for each scan|
|**cardNumber** * |String|19|If maskhint = masked (default):<br>•	First 6 and last 4 digits of the credit card number, other digits are masked with "x"<br>If maskhint = unmasked:<br>•	Full credit card number|
|cardExpiryMonth|Number|Min. value: 1<br>Max. value: 12|Month card expires|
|cardExpiryYear|Number|4|Year card expires in the format YY|
|cardHolderName|String|100|Name of the credit card holder in capital letters|
|cardSortCode|String|8|Sort code in the format xx-xx-xx or xxxxxx|
|cardAccountNumber|String|8|If maskhint = masked (default):<br>•	Last two digits of the account number, other digits masked with "x"<br>If maskhint = unmasked:<br>•	Full account number|
|merchantReportingCriteria|String|100|Your reporting criteria for each scan|

### Sample Request

```
GET https://bam-retrieval.jumio.com/api/netswipe/v1/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/creditCard HTTP/1.1
Accept: application/json
User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/x.x.x
Authorization: Basic
```

### Sample Response
```
{
"cardExpiryMonth":"1",
"cardExpiryYear":"2022",
"cardNumber":"123456xxxxxx1234",
"jumioRequestReference":"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

---

# Supported Cipher Suites
Jumio supported cipher suites during the TLS handshake.<p>
[View supported cipher suites](/netverify/supported-cipher-suites.md)

---
&copy; Jumio Corp. 268 Lambert Avenue, Palo Alto, CA 94306
