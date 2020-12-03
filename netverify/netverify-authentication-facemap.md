![Jumio](/images/Jumio-Authentication-Banner.png)

# Authentication with Facemap Storage on Customer Premise

This is a reference manual and configuration guide for Jumio Authentication with facemap storage on customer premise. It describes the workflow and how to initiate an Authentication transaction.

This feature allows you to use Authentication with needing to store the ID Verification PII data in the Jumio infrastructure.

You will retrieve the facemap (which represents the face identity of the user) using the ID Verification Retrieval API or ID Verification Callback. This facemap needs to be stored in your system. When you want to authenticate this user, you will send us the facemap using our server to server API to create a new Authentication transaction. You can start the user journey in your mobile app or web journey. Once the user completes the face capture process we will perform our 3D liveness checks on the face and compare it with the facemap identity you provide in the API call. Results of this transaction will let you know if the person is live and is the same person that was onboarded on your system.
<br>

### Revision history

Information about changes to features and improvements documented in each release is available in our [Revision history](/netverify/README.md).
<br>

## Table of contents

- [Preconditions](#preconditions)
- [Preparation](#preparation)
	- [Enable functionality](#enable-functionality)
	- [Store facemap](#store-facemap)
- [Initiating a transaction](#initiating-a-transaction)
	- [ID Verification Mobile](#id-verification-mobile)
		- [Authentication and encryption](#authentication-and-encryption)
		- [Request headers](#request-headers)
		- [Request body](#request-body)
		- [Response](#response)
		- [Examples](#examples)
	- [ID Verification Web](#id-verification-web)
		- [Authentication and encryption](#authentication-and-encryption-1)
		- [Request headers](#request-headers-1)
		- [Request body](#request-body-1)
		- [Response](#response-1)
		- [Examples](#examples-1)
- [Start Authentication](#start-authentication)
	- [Authentication SDK for Android](#authentication-sdk-for-android)
	- [Authentication SDK for iOS](#authentication-sdk-for-ios)
	- [Authentication SDK for Web](#authentication-for-web)

<br>

---
# Preparation

## Preconditions

The following conditions must be met before using this feature:

* The original ID Verification transaction needs to be deleted
* That ID Verification transaction has to be
	* verificationStatus = APPROVED_VERIFIED
	* similarity = MATCH
	* validity = TRUE

<br>

## Enable functionality

If you need this function enabled or are unsure if it is enabled on your account, please reach out to your Account Manager directly or contact the Jumio Support team at support@jumio.com.

<br>

## Store facemap

You must use either the callback or Retrieval API to get the facemap of a ID Verification transaction and store this on your system.

|⚠️ You are responsible to store the facemap to the proper user in your system. Jumio does not check if the facemap of the person belongs to the onboarded user and will only compare the facemap which was sent in for authentication with the ID Verification transaction.
|:----------|

### Callback for ID Verification

The callback parameter `facemap` contains an URL to the facemap of the transaction.

See [Callback for ID Verification](/netverify/callback.md#callback-for-authentication) for further information.

### ID Verification Retrieval API

The response parameter `facemap` will be returned when retrieving the details of a transaction.

See the [ID Verification Retrieval API Implementation Guide](/netverify/netverify-retrieval-api.md#authentication-retrieval) for further details.

<br>

---

# Initiating a transaction

## ID Verification Mobile

Call the RESTful API POST endpoint **/initiateAuthentication** with a JSON object containing the properties described below to create a transaction for each user. You will receive a JSON object in the response containing a new authentication transaction reference to start authentication.

 **HTTP Request Method:** `POST`<br>
 **REST URL (US)**: `https://netverify.com/api/authentication/v1/facemap/initiate`<br>
 **REST URL (EU)**: `https://lon.netverify.com/api/authentication/v1/facemap/initiate`<br>
 **REST URL (SGP)**: `https://core-sgp.jumio.com/api/authentication/v1/facemap/initiate`<br>


|⚠️ A new transaction will be created and the facemap will be temporarily stored until the transaction reaches a final state (max. 15 minutes).
|:----------|

|⚠️ If multiple transactions has been created for the same `enrollmentTransactionReference` within 15 minutes only the last one will successfully work. For the first ones the user will not be able to finish them, they will get a final state of `EXPIRED`.
|:----------|

<br>


### Authentication and encryption
Authentication API calls are protected using [HTTP Basic Authentication](https://tools.ietf.org/html/rfc7617). Your Basic Auth credentials are constructed using your API token as the user-id and your API secret as the password. You can view and manage your API token and secret in the Customer Portal under **Settings > API credentials**.
<br>

|⚠️ Never share your API token, API secret, or Basic Auth credentials with *anyone* — not even Jumio Support.
|:----------|

The [TLS Protocol](https://tools.ietf.org/html/rfc5246) is required to securely transmit your data, and we strongly recommend using the latest version. For information on cipher suites supported by Jumio during the TLS handshake see [Supported cipher suites](/netverify/supported-cipher-suites.md).

<br>


### Request headers

The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Content-Type: multipart/form-data`<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

<br>


### Request body

**Required items appear in bold font.**  

|Name               |Type   |Content-Type |Description      |
|:---               |:---   |:---         |:---             |
|**enrollmentMetadata**|object |**application/json** | The metadata required to initiate an authentication transaction.<br>See [supported enrollmentMetadata values](#supported-enrollmentMetadata-values).
|**facemap**|byte array |**application/octet-stream**| The enrollment facemap bytes.		|
<br>

#### Supported `enrollmentMetadata` values

**Required items appear in bold font.**

|Value  |enrollmentMetadata|
|:--------------|:--------------|
|**enrollmentTransactionReference**|The transaction reference from the onboarding ID verification to be used for authentication. <br><br> This is the Jumio scan reference of the original ID Verification transaction.|
|userReference	|Your internal reference for the user.|
|callbackUrl<sup>1</sup> 	|Sends verification result to this URL upon completion. Overrides [Callback URL](/netverify/portal-settings.md#callback-error-and-success-urls) in the Customer Portal.|

<sup>1</sup> See URL constraints for [Callback, Error, and Success URLs](/netverify/portal-settings.md#callback-error-and-success-urls).<br>

<br>

### Response
Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold font.**

|Name|Type|Max. length|Description|
|:----|:----|:----|:----|
|**authenticationTransactionReference**|String|36|Jumio reference number for the Authentication transaction.|

<br>

### Examples

#### Sample request

~~~
POST https://netverify.com/api/v4/initiateAuthentication/ HTTP/1.1
Accept: application/json
Content-Type: multipart/form-data
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx

------WebKitFormBoundary7MA4YWxkTrZu0gW--,
Content-Disposition: form-data; name="facemap"; filename="facemap.bin"


------WebKitFormBoundary7MA4YWxkTrZu0gW--
Content-Disposition: form-data; name="enrollmentMetadata"

{
	"enrollmentMetadata": {
		"enrollmentTransactionReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  		"userReference": "user_1234",
		"callbackUrl": "https://www.yourcompany.com/callback"
	}
}
------WebKitFormBoundary7MA4YWxkTrZu0gW--

~~~

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

#### Sample response

~~~
{
  "authenticationTransactionReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
~~~

<br>

## ID Verification Web

Call the RESTful API POST endpoint **/initiateAuthentication** with a JSON object containing the properties described below to create a transaction for each user. You will receive a JSON object in the response containing a new authentication transaction reference to start authentication.

 **HTTP Request Method:** `POST`<br>
 **REST URL (US)**: `https://netverify.com/api/authentication/v1/facemap/initiate`<br>
 **REST URL (EU)**: `https://lon.netverify.com/api/authentication/v1/facemap/initiate`<br>
 **REST URL (SGP)**: `https://core-sgp.jumio.com/api/authentication/v1/facemap/initiate`<br>


|⚠️ A new transaction will be created and the facemap will be temporarily stored until the transaction reaches a final state (max. 15 minutes).
|:----------|

|⚠️ If multiple transactions has been created for the same `enrollmentTransactionReference` within 15 minutes only the last one will successfully work. For the first ones the user will not be able to finish them, they will get a final state of `EXPIRED`.
|:----------|

<br>


### Authentication and encryption
Authentication API calls are protected using [HTTP Basic Authentication](https://tools.ietf.org/html/rfc7617). Your Basic Auth credentials are constructed using your API token as the user-id and your API secret as the password. You can view and manage your API token and secret in the Customer Portal under **Settings > API credentials**.
<br>

|⚠️ Never share your API token, API secret, or Basic Auth credentials with *anyone* — not even Jumio Support.
|:----------|

The [TLS Protocol](https://tools.ietf.org/html/rfc5246) is required to securely transmit your data, and we strongly recommend using the latest version. For information on cipher suites supported by Jumio during the TLS handshake see [Supported cipher suites](/netverify/supported-cipher-suites.md).

<br>


### Request headers

The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Content-Type: multipart/form-data`<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

<br>


### Request body

**Required items appear in bold font.**  

|Name               |Type   |Content-Type |Description      |
|:---               |:---   |:---         |:---             |
|**enrollmentMetadata**|object |**application/json** | The metadata required to initiate an authentication transaction.<br>See [supported enrollmentMetadata values](#supported-enrollmentMetadata-values).
|**facemap**|byte array |**application/octet-stream**| The enrollment facemap bytes.		|

<br>

#### Supported `enrollmentMetadata` values

**Required items appear in bold font.**

|Value  |enrollmentMetadata|
|:--------------|:--------------|
|**enrollmentTransactionReference**|The transaction reference from the onboarding ID verification to be used for authentication. <br><br> This is the Jumio scan reference of the original ID Verification transaction.|
|userReference	|Your internal reference for the user.|
|callbackUrl<sup>1</sup> 	|Sends verification result to this URL upon completion. <br>Overrides [Callback URL](/netverify/portal-settings.md#callback-error-and-success-urls) in the Customer Portal.|
|successUrl<sup>1</sup>         |Redirects to this URL after a successful transaction.<br>Overrides [Success URL](/netverify/portal-settings.md#callback-error-and-success-urls) in the Customer Portal.		|
|errorUrl<sup>1</sup>                |Redirects to this URL after an unsuccessful transaction.<br>Overrides [Error URL](/netverify/portal-settings.md#callback-error-and-success-urls) in the Customer Portal.		|
|userReference<sup>2</sup> |Your internal reference for the user.|
|locale                 |Renders content in the specified language.<br>Overrides [Default locale](#default-locale) in the Customer Portal.<br>See [supported locale values](/netverify/netverify-authentication.md#supported-locale-values).		|

<sup>1</sup> See URL constraints for [Callback, Error, and Success URLs](/netverify/portal-settings.md#callback-error-and-success-urls).<br>
<sup>2</sup> Values **must not** contain Personally Identifiable Information (PII) or other sensitive data such as email addresses.


<br>

### Response
Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold font.**

|Name|Type|Max. length|Description|
|:----|:----|:----|:----|
|**authenticationTransactionReference**|String|36|Jumio reference number for the Authentication transaction.|
|**redirectUrl**|String|255|URL used to load Authentication client. TBD|

<br>

### Examples

#### Sample request

~~~
POST https://netverify.com/api/v4/initiateAuthentication/ HTTP/1.1
Accept: application/json
Content-Type: multipart/form-data
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx

------WebKitFormBoundary7MA4YWxkTrZu0gW--,
Content-Disposition: form-data; name="facemap"; filename="facemap.bin"


------WebKitFormBoundary7MA4YWxkTrZu0gW--
Content-Disposition: form-data; name="enrollmentMetadata"

{
	"enrollmentMetadata": {
		"enrollmentTransactionReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
		"userReference": "transaction_1234", "callbackUrl": "https://www.yourcompany.com/callback",
		"successUrl" : "https://www.yourcompany.com/success", "errorUrl" : "https://www.yourcompany.com/error",
		"tokenLifetimeInMinutes": 5,"locale": "de"
	}
}
------WebKitFormBoundary7MA4YWxkTrZu0gW--

~~~

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

#### Sample response

~~~
{
  "authenticationTransactionReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  "redirectUrl": "https://yourcompany.netverify.com/web/v4/app?authorizationToken=xxxxxxxxxxxxxxlocale=en-GB"
}
~~~

<br>

---

# Start Authentication

You can use the `authenticationTransactionReference` to start the Authentication workflow within the SDK, as long as the reference has not been finalized.

A transaction has been finalized once it has one of the final states:<br>
* PASSED<br>
* FAILED<br>
* INVALID<br>
* EXPIRED<br>

<br>

## Authentication SDK for Android

Follow the instructions in our [mobile guide](https://github.com/Jumio/Mobile-SDK-ANDROID/blob/master/docs/integration_authentication.md#starting-the-sdk) to start the SDK workflow.  

<br>

## Authentication SDK for iOS

Follow the instructions in our [mobile guide](https://github.com/Jumio/Mobile-SDK-IOS/blob/master/docs/integration_authentication.md#initialization) to start the SDK workflow.

<br>

## Authentication for Web

Follow the instructions in our [Web guide - Section "Displaying Authentication"](/netverify/netverify-authentication.md#displaying-authentication) to start the web workflow by using the `redirectUrl` you received before.

<br>


---
&copy; Jumio Corp. 395 Page Mill Road, Suite 150, Palo Alto, CA 94306
