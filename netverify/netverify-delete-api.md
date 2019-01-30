![Jumio](/images/netverify.png)

# Netverify Delete API Implementation Guide

This guide illustrates how to implement the Netverify Delete API.

### Revision history

| Date    | Description|
|:--------|:------------|
| 2019-01-30  |Document formatting|
| 2016-05-18  |Removed TLS\_DHE ciphers|
| 2015-10-20  |Added ECDHE ciphers to supported cipher suites|
| 2015-09-23  |Initial release|

---

## Table of Contents

- [About deleting transactions](#about-deleting-transactions)
- [Using the Delete API to delete a Netverify transaction](#using-the-delete-api-to-delete-a-netverify-transaction) 
- [Using the Delete API to delete a Document Verification transaction](#using-the-delete-api-to-delete-a-document-verification-transaction) 
	- [Authentication and encryption](#authentication-and-encryption)
	- [Request headers](#request-headers)
	- [Request path parameter](#request-path-parameter)
	- [Response](#response)
	- [Examples](#examples)
		- [Sample Netverify delete request](#sample-netverify-delete-request)
		- [Sample Document Verification delete request](#sample-document-verification-delete-request)




---
# About deleting transactions

If you do not wish to store sensitive customer data after a verification has been completed, you can delete the image(s) and any extracted data in a transaction record either directly in the Customer Portal or via the Delete API.

|⚠️ Deletion is immediate and irreversible. Jumio cannot perform any quality or technical support investigations on transactions that have been deleted.
|:----------|
<br>

---
# Using the Delete API to delete a Netverify transaction

Use the HTTP `DELETE` method to call the RESTful API endpoint below. Specify the Jumio transaction reference you want to delete as a path parameter.

**HTTP Request Method:** `DELETE`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/scans/<scanReference>`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/scans/<scanReference>`<br>
<br>

---
# Using the Delete API to delete a Document Verification transaction

Use the HTTP `DELETE` method to call the RESTful API endpoint below. Specify the Jumio transaction reference you want to delete as a path parameter.

**HTTP Request Method:** `DELETE`<br>
**REST URL (US)**: `https://retrieval.netverify.com/api/netverify/v2/documents/<scanReference>`<br>
**REST URL (EU)**: `https://retrieval.lon.netverify.com/api/netverify/v2/documents/<scanReference>`<br>
<br>

## Authentication and encryption
Netverify API calls are protected using [HTTP Basic Authentication](https://tools.ietf.org/html/rfc7617). Your Basic Auth credentials are constructed using your API token as the user-id and your API secret as the password. You can view and manage your API token and secret in the Customer Portal under **Settings > API credentials**. 

You can generate a separate set of API credentials in the Customer Portal to use when retrieving or deleting transaction data under **Settings > API credentials > Transaction administration APIs**.
<br>

|⚠️ Never share your API token, API secret, or Basic Auth credentials with *anyone* — not even Jumio Support.
|:----------|
<br> 
The [TLS Protocol](https://tools.ietf.org/html/rfc5246) is required to securely transmit your data, and we strongly recommend using the latest version. For information on cipher suites supported by Jumio during the TLS handshake see [Supported cipher suites](/netverify/supported-cipher-suites.md).<br>	
<br>

## Request headers

The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|
<br>

## Request path parameter

**Required items appear in bold type.**  

|Name       | Type    | Max. Length| Description|
|:---------------|:--------|:------------|:------------|
|**scanReference**| String|36|Jumio reference number for an existing transaction in your account.|
<br>

## Response

Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error.

Successful requests will return HTTP status code `200 OK` as confirmation that you have successfully deleted the image(s) and extracted data from the specified transaction record.<br>
<br>

## Examples
### Sample Netverify delete request

~~~
DELETE https://netverify.com/api/netverify/v2/scans/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/ HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
~~~


### Sample Document Verification delete request

~~~
DELETE https://retrieval.netverify.com/api/netverify/v2/documents/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/ HTTP/1.1
Accept: application/json
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
~~~
<br>

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>



---
&copy; Jumio Corp. 268 Lambert Avenue, Palo Alto, CA 94306
