![Jumio](/images/netverify.png)

# Global Netverify Settings

In your Jumio customer portal settings, you can configure the ID verification as follows.


## Table of Contents
- [Application settings](#application-settings)
    - [Callback URL](#callback-url)
    - [Success and Error URLs](#success-and-error-urls)
    - [ID Scan Capture Method](#id-scan-capture-method)
- [Customize client](#customize-client)
- [Accepted IDs](#accepted-ids)
- [Data settings](#data-settings)
    - [Mandatory Fields](#mandatory-fields)
    - [Optional Fields](#optional-fields)
    - [Data Retention Fields](#data-retention)
- [Multi Documents](#multi-documents)
    - [Custom](#custom)
- [Two-factor Authentication](#two-factor-authentication)

---
# Application Settings

## Callback URL

Provide a URL which meets the following constraints:
* HTTPS using the TLS protocol (we strongly recommend using the latest version)
* Valid URL (RFC-2396) using ASCII characters or IDNA Punycode
* IP addresses, ports, query parameters and fragment identifiers are not allowed

Whitelist the following IP addresses for callbacks, and use these to verify that the callback originated from Jumio:

**US data center:** </br>
184.106.91.66, 184.106.91.67, 104.130.61.196, 146.20.77.156, 34.202.241.227, 34.226.103.119, 34.226.254.127, 52.53.95.123, 52.52.51.178, 54.67.101.173.  </br>
You can look up the IP addresses with the host name "callback.jumio.com".<p>

**EU data center:**</br>
162.13.228.132, 162.13.228.134, 162.13.229.103, 162.13.229.104, 34.253.41.236, 52.209.180.134, 52.48.0.25, 35.157.27.193, 52.57.194.92, 52.58.113.86. <br/>
You can look up the IP addresses with the host name "callback.lon.jumio.com".

Jumio will post callbacks to your HTTPS URL if you are using a valid certificate to ensure a successful TLS handshake. If you are not receiving callbacks, please check the following [article](https://support.jumio.com/hc/en-us/articles/200275338-I-am-not-receiving-callbacks-What-can-I-do).

## Success and Error URLs

Provide redirect URLs which meet the following constraints:
- Valid URL (RFC-2396) using ASCII characters or IDNA Punycode
-	IP addresses, ports, query parameters and fragment identifiers are not allowed
-	HTTPS is strongly recommend, using the TLS protocol

## ID Scan Capture Method

You can restrict Netverify Web to capture IDs via camera or upload option only.

**Note:** Selecting "Webcam only" means some mobile browsers will not be supported.


---
# Customize Client
Specify primary and secondary colors for your own look and feel per locale for Netverify Web.

Any configuration which is not set will first default to the language (e.g. EN\_GB to EN), then to your default setup, and finally to the Jumio standard.


---
## Accepted IDs
You can configure accepted IDs per region or country. The default setup includes all countries and ID types supported by Jumio at the time when your account was created.

**Note:** You can disable the option to automatically accept newly supported IDs by Jumio.

---
## Data Settings
You can choose which fields should be processed during the ID verification.

### Mandatory Fields
Mandatory fields will be returned in the callback for all Jumio supported IDs, if enabled.
* Date of birth
* ID number
* First name
* Last name

### Optional Fields
Optional fields will be returned in the callback under certain conditions, if enabled.
* Expiry Date (if availble on the ID)
* US state (if US ID)
* Personal number (if available on the ID)
* MRZ check (if passport for MRZ check supported for the ID card)

**Note:** To perform the MRZ check, the fields date of birth, expiry and personal number will be processed during the ID verification and returned in the callback, if available on the ID.

**Supported countries for ID card MRZ check:** Albania, Argentina, Austria, Bosnia & Herzegovina, Bulgaria, Chile, Croatia, Czech Republic, Denmark, Dominican Republic, Ecuador, El Salvador, Estonia, Finland, France, Georgia, Germany, Guatemala, Hungary, Italy, Kazakhstan, Kenya, Latvia, Liechtenstein, Lithuania, Macedonia, Malta, Mexico , Moldova, Montenegro, Netherlands, Pakistan, Paraguay, Peru, Poland, Portugal, Romania, Serbia, Slovakia, Slovenia, Spain, Sweden, Switzerland, Turkey, United Arab Emirates

### Data Retention
Select a time interval for permanent purge of sensitive data (360 days by default) or keep your data forever. The deletion of fraud transactions can be enabled (excluded by default).

---
## Multi Documents

### Custom

In your Jumio customer portal settings, you can create your own custom document types. After submitting a new custom document type it takes up to five minutes until it is applied.

Specify:
* A unique document code (constraint: Blanks are not allowed),
* A name and
* A default label name in English.

It is possible to add translations for different languages.

**Note:** All fields can be updated after creation, except the document code. Created document types cannot be deleted, but they can be disabled (because of the unique document code which cannot be reused).


---

# Two-factor Authentication

If you want to enable two-factor authentication for your Jumio customer portal please contact support@jumio.com. Once enabled, users will be guided through the setup upon their first login to obtain a security code using the "Google Authenticator" app.


---
&copy; Jumio Corp. 268 Lambert Avenue, Palo Alto, CA 94306
