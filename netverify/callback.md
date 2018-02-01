![Jumio](/images/jumio_feature_graphic.png)

# Callback

The callback is the authoritative answer from Jumio. Specify a callback URL (for constaints see [Global Netverify settings](/netverify/portal-settings.md#callback-url)) to recieve the ID verification result for each scan.


## Table of Contents

- [Callback for Netverify](#callback-for-netverify)
- [Callback for Document Verification](#callback-for-document-verification)
    - [Supported Documents for Data Extraction](#supported-documents-for-data-extraction)
- [Netverify Retrieval API](#netverify-retrieval-api)
- [Supported cipher suites](#supported-cipher-suites)


---
## Callback for Netverify

An HTTP POST is sent to your specified callback URL (see [Global Netverify settings](/netverify/portal-settings.md#callback-url)) containing an `application/x-www-form-urlencoded` formatted string with the result.

For mobile: ID verification must be enabled to receive the callback.

#### Netverify Web

|User journey state       | Scan state    | Callback|
|:---------------|:--------|:------------|
|Not started or drop off before/during first attempt|Pending => Failed|Transaction will be cleaned-up from pending to failed after the authorization token lifetime<br>Callback: Verification status NO\_ID\_UPLOADED |
|Drop off during second or third attempt  | Done  |Transaction will be finished 15 minutes after the last update<br>Callback: Verification status ERROR\_NOT\_READABLE\_ID with previous reject reason |
|Finished       | Done  |Callback: Verification status depends on the result |

#### Netverify Mobile

|User journey state       | Scan state    | Callback|
|:---------------|:--------|:------------|
|Drop off   | Pending => Failed  |Transaction will be cleaned-up from pending to failed 15 minutes after last update<br/>Callback: verification status NO\_ID\_UPLOADED|
|Finished   | Done  |ID verification will be performed<br/>Callback: verification status depends on the result (see table below)|


### Parameters

The following parameters are posted to your callback URL for Netverify Web embedded, redirect, performNetverify and Netverify Mobile iOS/Android.

**Note:** Mandatory parameters are highlighted bold.

|Parameter       | Max. Length    | Description| Activation needed |
|:---------------|:--------|:------------| ------ |
|**callBackType**   |   |NETVERIFYID | |
|**jumioIdScanReference**   |36  |Jumio’s reference number for each scan | |
|**verificationStatus**   |   |Possible states:<br/>•	APPROVED\_VERIFIED<br/> •	DENIED\_FRAUD<br/>•	DENIED\_UNSUPPORTED\_ID\_TYPE (`*1`)<br/> •	DENIED\_UNSUPPORTED\_ID\_COUNTRY (`*1`)<br/>•	ERROR\_NOT\_READABLE\_ID<br/> •	NO\_ID\_UPLOADED | |
|**idScanStatus**   |   |SUCCESS if verificationStatus = APPROVED\_VERIFIED, otherwise ERROR | |
|**idScanSource**   |   |Possible values:<br>•	WEB (if Netverify Web embedded and no camera or upload started)<br>•	WEB\_CAM (if Netverify Web embedded via camera)<br>•	WEB\_UPLOAD (if Netverify Web embedded via upload)<br>•	REDIRECT (if Netverify Web redirect and no camera or upload started)<br>•	REDIRECT\_CAM (if Netverify Web redirect via camera)<br>•	REDIRECT\_UPLOAD (if Netverify Web redirect via upload)<br>•	API (if performNetverify)<br>• SDK (if mobile) | |
|**idCheckDataPositions**   |   |"OK" if verificationStatus = APPROVED\_VERIFIED, otherwise "N/A" | |
|**idCheckDocumentValidation**   |   |"OK" if verificationStatus = APPROVED\_VERIFIED, otherwise "N/A" | |
|**idCheckHologram**   |   |"OK" if verificationStatus = APPROVED\_VERIFIED, otherwise "N/A" | |
|**idCheckMRZcode**   |   |"OK" for passports and supported ID cards if verificationStatus = APPROVED\_VERIFIED and MRZ check is enabled, otherwise "N/A" | |
|**idCheckMicroprint**   |   |"OK" if verificationStatus = APPROVED\_VERIFIED, otherwise "N/A" | |
|**idCheckSecurityFeatures**   |   |"OK" if verificationStatus = APPROVED\_VERIFIED, otherwise "N/A" | |
|**idCheckSignature**   |   |"OK" if verificationStatus = APPROVED\_VERIFIED, otherwise "N/A" | |
|**transactionDate**   |   |Timestamp of the scan creation in the format YYYY-MM-DDThh:mm:ss.SSSZ | |
|**callbackDate**   |   |Timestamp of the callback creation in the format YYYY-MM-DDThh:mm:ss.SSSZ | |
|idType   |   |Possible types:<br/>•	PASSPORT<br/>•	DRIVING\_LICENSE<br/>•	ID\_CARD<br>•	VISA (only supported for country = CHN or USA) | |
|idSubtype   |255  |Possible subtypes if idType = ID\_CARD<br/>•	NATIONAL\_ID<br/>•	CONSULAR\_ID<br/>•	ELECTORAL\_ID<br/>•	RESIDENT\_PERMIT\_ID<br/>•	TAX\_ID (only supported for PHL)<br/>•	STUDENT\_ID (only supported for POL)<br/>•	PASSPORT\_CARD\_ID (only supported for IRL, RUS and USA)<br/>•	MILITARY\_ID (only supported for GRC)<br/>•	OTHER\_ID<br/>•	VISA (only supported for USA)<br/>•	UNKNOWN<br/><br/>Possible subtypes if idType = DRIVING\_LICENSE<br/>•	LEARNING\_DRIVING\_LICENSE (only supported for USA, CAN, AUS, GBR, IRL, DEU)<br/><br/>Possible subtypes if idType = PASSPORT<br/>•	E\_PASSPORT (only for mobile) | |
|idCountry   |3  |Possible countries:<br/>•	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br/>•	XKX (Kosovo)| |
|rejectReason   |   |Reject reason as JSON object if verificationStatus = DENIED\_FRAUD or ERROR\_NOT\_READABLE\_ID, see tables below  | |
|idFaceMatch **(deprecated)**   |   |Face match percentage 0-100 if verificationStatus = APPROVED\_VERIFIED (`*2`) |X |
|idScanImage   |255  |URL to the image of the scan (JPEG or PNG) if available (`*3`) | |
|idScanImageFace   |255  |URL to the face image of the scan (JPEG or PNG) if available (`*3`)| |
|idScanImageBackside   |255  |URL to the back side image of the scan (JPEG or PNG) if available (`*3`)| |
|idNumber   |200  |Identification number of the document as available on the ID if verificationStatus = APPROVED\_VERIFIED and enabled, otherwise if provided | |
|idFirstName   |200  |•	First name of the customer as available on the ID if verificationStatus = APPROVED\_VERIFIED and enabled, otherwise if provided<br/><br>•	N/A (for non-Latin characters)<br />o	if idCountry = CHN and idType = DRIVING\_LICENSE or ID\_CARD<br>o	if idCountry = KOR and idType = DRIVING\_LICENSE or ID\_CARD<br>o	if idCountry = JPN and idType = DRIVING\_LICENSE<br>o	if idCountry = RUS and idType = ID\_CARD| |
|idLastName   |200  |•	Last name of the customer as available on the ID if verificationStatus = APPROVED\_VERIFIED and enabled, otherwise if provided<br/><br>•	Only if full name is printed in Latin characters<br />o if idCountry = KOR and idType = DRIVING\_LICENSE (first name and last name)<br /><br> •	N/A (for non-Latin characters)<br />o	if idCountry = CHN and idType = DRIVING\_LICENSE or ID\_CARD<br>o	if idCountry = KOR and idType = DRIVING\_LICENSE or ID\_CARD<br>o	if idCountry = JPN and idType = DRIVING\_LICENSE<br>o	if idCountry = RUS and idType = ID\_CARD | |
|idDob   |10  |Date of birth in the format YYYY-MM-DD as available on the ID if verificationStatus = APPROVED\_VERIFIED and enabled, otherwise if provided | |
|idExpiry   |10  |Date of expiry in the format YYYY-MM-DD as available on the ID if verificationStatus = APPROVED\_VERIFIED and enabled, otherwise if provided | |
|idUsState   |3  |Possible values if idType = PASSPORT or ID\_CARD:<br/>•	Last two characters of [ISO 3166-2:US](http://en.wikipedia.org/wiki/ISO_3166-2:US) state code<br/>•	Last 2-3 characters of [ISO 3166-2:AU](http://en.wikipedia.org/wiki/ISO_3166-2:AU) state code<br/>•	Last two characters of [ISO 3166-2:CA](http://en.wikipedia.org/wiki/ISO_3166-2:CA) state code<br/>• [ISO 3166-1](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country name<br/>• XKX (Kosovo)<br/><br/>If idType = DRIVING\_LICENSE:<br/>•	Last two characters of [ISO 3166-2:US](http://en.wikipedia.org/wiki/ISO_3166-2:US) state code<br/>•	Last 2-3 characters of [ISO 3166-2:AU](http://en.wikipedia.org/wiki/ISO_3166-2:AU) state code<br/>•	Last two characters of [ISO 3166-2:CA](http://en.wikipedia.org/wiki/ISO_3166-2:CA) state code| |
|personalNumber   |14  |Personal number of the document <br />• if verificationStatus = APPROVED\_VERIFIED and <br />• if idType = PASSPORT and if data available on the document | |
|idAddress   |   |Address as JSON object in US, EU or raw format if verificationStatus = APPROVED, see tables below (`*4`) |X |
|merchantIdScanReference   |100  |Your reference for each scan | |
|merchantReportingCriteria   |100  |Your reporting criteria for each scan | |
|customerId   |100  |ID of the customer as provided | |
|clientIp   |   |IP address of the client in the format xxx.xxx.xxx.xxx | |
|firstAttemptDate   |  |Timestamp of the first scan attempt in the format YYYY-MM-DDThh:mm:ss.SSSZ | |
|optionalData1   |255  |Optional field of MRZ line 1 | |
|optionalData2   |255  |Optional field of MRZ line 2 | |
|dni   |255  |DNI as available on the ID if idCountry = ESP and idSubtype = NATIONAL\_ID  | |
|gender   |2  |Possible values: M, F<br>• if idCountry = FRA and idSubtype = NATIONAL\_ID (MRZ type CNIS)<br/>•	if idType = VISA and additional extraction for Visa enabled | |
|idFaceLiveness **(deprecated)**  |   |only available for SDK<br/>Possible values:<br/>•	TRUE (if face match enabled for ID verification and liveness detected successfully during scanning)<br/>•	FALSE |X |
|identityVerification   |   |Identity verification as JSON object if verificationStatus = APPROVED\_VERIFIED, see table below|X |
|presetCountry   | 3  |Possible countries:<br />•	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code <br /> • XKX (Kosovo)| |
|presetIdType   |    |Possible ID types: PASSPORT, DRIVING_LICENSE, ID_CARD| |
|dlCategories   | JSON object  |Driver license categories as JSON object if verificationStatus = APPROVED\_VERIFIED, see table below<br />(only supported for country FRA and BEL)|X |
|nationality |3|Nationality if idType = VISA and additional extraction for Visa enabled. Possible countries: <br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code <br />• XKX (Kosovo)|X |
|passportNumber|255|Passport number if idType = VISA and additional extraction for Visa enabled|X |
|durationOfStay|255|Duration of stay if idType = VISA and additional extraction for Visa enabled|X |
|numberOfEntries|255|Number of entries if idType = VISA and additional extraction for Visa enabled|X |
|visaCategory|255|Visa category if idType = VISA and additional extraction for Visa enabled|X |



(`*1`) Scan is declined as unsupported if the provided ID is not supported by Jumio, or not accepted in your Netverify settings.<br/>
(`*2`) Face match is performed if enabled.<br/>
(`*3`) For ID types that are configured to support a separate scan of the front side and back side, there is a separate image of the front side (idScanImage) and the back side (idScanImageBackside). If face match is enabled, there is also a picture of the face (idScanImageFace).
To access the image, use the HTTP GET method and HTTP Basic Authentication with your API token as the "userid" and your API secret as the "password". Set "User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION" (e.g. MyCompany MyApp/1.0.0) in the "header" section of your request. The TLS protocol is required during the TLS handshake (see [Supported cipher suites](/netverify/supported-cipher-suites.md)) and we strongly recommend using the latest version.<br/>
(`*4`) Address recognition is performed for supported IDs, if enabled. Please note, there are three different address formats. You are able to view which format applies to specific IDs by logging into your Jumio customer portal and navigating to the "Data settings" section. The different address parameters are a part of the JSON object, if they are available on the ID.

### US Address Format

|Parameter "idAddress"       | Max. Length    | Description|
|:---------------|:--------|:------------|
|city   |64  |City |
|stateCode   |6  |[ISO 3166-2](http://en.wikipedia.org/wiki/ISO_3166-2) state code |
|streetName   |64  |Street name |
|streetSuffix   |14  |Street suffix abbreviation<br/>Examples: [US](http://www.gis.co.clay.mn.us/USPS.htm#suffix), [Canada](http://www.canadapost.ca/tools/pg/manual/PGaddress-e.asp#1423617), [Australia](https://auspost.com.au/media/documents/australia-post-addressing-standards-1999.pdf) |
|streetDirection   |4  |Street direction abbreviation<br/>Examples: US (E=EAST, W=WEST, N=NORTH, S=SOUTH), [Canada](http://www.canadapost.ca/tools/pg/manual/PGaddress-e.asp#1403220), [Australia](https://auspost.com.au/media/documents/australia-post-addressing-standards-1999.pdf) |
|streetNumber   |14  |Street number |
|unitDesignator   |14  |Unit designator abbreviation<br/>Examples: [US](http://www.gis.co.clay.mn.us/USPS.htm#secunitdesig), [Canada](http://www.canadapost.ca/tools/pg/manual/PGaddress-e.asp#1380473), [Australia](https://auspost.com.au/media/documents/australia-post-addressing-standards-1999.pdf) |
|unitNumber   |14  |Unit number |
|zip   |14  |Zip code |
|zipExtension   |20  |Zip extension |
|country   |3  |Possible countries:<br/>•	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br/>•	XKX (Kosovo) |

### EU Address Format

|Parameter "idAddress"       | Max. Length    | Description|
|:---------------|:--------|:------------|
|city   |64  |City |
|province   |64  |Province |
|streetName   |64  |Street name |
|streetNumber   |15  |Street number |
|unitDetails   |64  |Unit details |
|postalCode   |15  |Postal code |
|country   |3  |Possible countries:<br/>•	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br/>•	XKX (Kosovo) |

### Raw Address Format

|Parameter "idAddress"       | Max. Length    | Description|
|:---------------|:--------|:------------|
|line1   |100  |Line item 1 |
|line2   |100  |Line item 2 |
|line3   |100  |Line item 3 |
|line4   |100  |Line item 4 |
|line5   |100  |Line item 5 |
|country   |3  |Possible countries:<br/>•	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br/>•	XKX (Kosovo)  |
|postalCode   |15  |Postal code |
|city   |64  |City |

### Reject Reason

|Parameter "rejectReason" | Type   | Max. Length    | Description|
|:------------------------|:--------|:--------|:------------|
|rejectReasonCode |String| 5  |see below |
|rejectReasonDescription |String |64  |Possible codes and descriptions for verification status DENIED\_FRAUD:<br>100	MANIPULATED\_DOCUMENT<br/>105	FRAUDSTER<br/>106	FAKE<br/>107	PHOTO\_MISMATCH<br/>108	MRZ\_CHECK\_FAILED<br/>109	PUNCHED\_DOCUMENT<br/>110	CHIP\_DATA\_MANIPULATED (only available for ePassport)<br/>111	MISMATCH\_PRINTED\_BARCODE_DATA<br><br>Possible codes and descriptions for verificationStatus = ERROR\_NOT\_READABLE\_ID:<br/>102	PHOTOCOPY\_BLACK\_WHITE<br/>103	PHOTOCOPY\_COLOR (for sources WEB\_CAM and REDIRECT\_CAM)<br/>104	DIGITAL\_COPY<br/>200	NOT\_READABLE\_DOCUMENT<br/>201	NO\_DOCUMENT<br/>202	SAMPLE\_DOCUMENT<br/>206	MISSING\_BACK<br/>207	WRONG\_DOCUMENT\_PAGE<br/>209	MISSING\_SIGNATURE<br/>210	CAMERA\_BLACK\_WHITE<br/>211	DIFFERENT\_PERSONS\_SHOWN (documents of multiple people in one image)<br/>300	MANUAL\_REJECTION|
|rejectReasonDetails |Object  |   |Reject reason details as JSON array containing JSON objects if rejectReasonCode = 100 or 200, see table below |


### Reject Reason Details

|Parameter "rejectReasonDetails" |  Type    | Max. Length    | Description|
|:-------------------------------|:---------|:---------------|:------------|
|detailsCode   |String | 5 | see below |
|detailsDescription|String|32| Possible codes and descriptions for rejectReasonCode = 100:<br/>1001	PHOTO<br/>1002	DOCUMENT\_NUMBER<br>1003	EXPIRY<br/>1004	DOB<br/>1005	NAME<br/>1006	ADDRESS<br/>1007	SECURITY\_CHECKS<br/>1008	SIGNATURE<br>1009	PERSONAL\_NUMBER<br>10011 PLACE_OF_BIRTH<br><br>Possible codes and descriptions for rejectReasonCode = 200:<br/>2001	BLURRED<br/>2002	BAD\_QUALITY<br/>2003	MISSING\_PART\_DOCUMENT<br/>2004	HIDDEN\_PART\_DOCUMENT<br/>2005	DAMAGED\_DOCUMENT |


### Identity Verification

|Parameter "identityVerification"       | Max. Length    | Description|
|:---------------|:--------|:------------|
|**similarity**  |  |Possible values:<br/> •	MATCH<br />•	NO\_MATCH<br />•	NOT\_POSSIBLE|
|**validity**  |  |Possible values:<br/> •	TRUE<br />•	FALSE |
|reason   |  |Provided if validity = FALSE<br/>Possible values:<br />• SELFIE\_CROPPED\_FROM\_ID<br />•	ENTIRE\_ID\_USED\_AS\_SELFIE<br />•	MULTIPLE\_PEOPLE<br />•	SELFIE\_IS\_SCREEN\_PAPER\_VIDEO<br />•	SELFIE\_MANIPULATED<br />• AGE\_DIFFERENCE\_TOO\_BIG<br />•	NO\_FACE\_PRESENT<br />•	FACE\_NOT\_FULLY\_VISIBLE<br />•	BAD\_QUALITY<br />•	BLACK\_AND\_WHITE|

### Driver License Categories

|Parameter "dlCategories"       | Max. Length    | Description|
|:---------------|:--------|:------------|
|**category** | 10 |Possible values:<br/> •	B1<br />•	B<br />•	BE|
|issueDate   |  |Issue date in the format YYYY-MM-DD|
|expiryDate   |  |Date of expiry in the format YYYY-MM-DD as available on the driver license|
|isReadable   |  |Possible value:<br>• FALSE |


### Sample Callbacks

#### Sample callback (URL-encoded POST): Approved and verified

```
idExpiry=2022-12-31&idType=PASSPORT&idDob=1990-01-01&idCheckSignature=OK&idCheckDataPositions=OK&idCheckHologram=OK&idCheckMicroprint=OK&idCheckDocumentValidation=OK&idCountry=USA&idScanSource=SDK&idFirstName=FIRSTNAME&verificationStatus=APPROVED_VERIFIED&jumioIdScanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&personalNumber=N%2FA&merchantIdScanReference=YOURIDSCANREFERENCE&idCheckSecurityFeatures=OK&idCheckMRZcode=OK&idScanImage=https%3A%2F%2Fnetverify.com%2Frecognition%2Fv1%2Fidscan%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Ffront&callBackType=NETVERIFYID&clientIp=xxx.xxx.xxx.xxx&idLastName=LASTNAME&idAddress=%7B%22country%22%3A%22USA%22%2C%22stateCode%22%3A%22US%2DOH%22%7D&idScanStatus=SUCCESS&identityVerification=%7B%22similarity%22%3A%22MATCH%22%2C%22validity%22%3Atrue%7D&idNumber=P1234
```

#### Sample callback (URL-encoded POST): Fraud

```
idType=PASSPORT&idCheckSignature=N%2FA&rejectReason=%7B%20%22rejectReasonCode%22%3A%22100%22%2C%20%22rejectReasonDescription%22%3A%22MANIPULATED_DOCUMENT%22%2C%20%22rejectReasonDetails%22%3A%20%5B%7B%20%22detailsCode%22%3A%20%221001%22%2C%20%22detailsDescription%22%3A%20%22PHOTO%22%20%7D%2C%7B%20%22detailsCode%22%3A%20%221004%22%2C%20%22detailsDescription%22%3A%20%22DOB%22%20%7D%5D%7D&idCheckDataPositions=N%2FA&idCheckHologram=N%2FA&idCheckMicroprint=N%2FA&idCheckDocumentValidation=N%2FA&idCountry=USA&idScanSource=SDK&verificationStatus=DENIED_FRAUD&jumioIdScanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&merchantIdScanReference=YOURSCANREFERENCE&idCheckSecurityFeatures=N%2FA&idCheckMRZcode=N%2FA&idScanImage=https%3A%2F%2Fnetverify.com%2Frecognition%2Fv1%2Fidscan%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Ffront&callBackType=NETVERIFYID&clientIp=xxx.xxx.xxx.xxx&idScanStatus=ERROR
```

---

## Callback for Document Verification
(formerly Netverify Multi Document)

### Parameters

The following parameters are posted to your callback URL for Document Verification and Document Verification API.

**Note:** Mandatory parameters are highlighted in bold.

|Parameter       | Type    | Max. Length|  Description|
|:---------------|:--------|:----------: |:------------|
|**scanReference**   | String  |36 |Jumio's reference number for each scan|
|**timestamp** | String  |  |Timestamp of the response in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**transaction** | JSON object  |  |Transaction related data, see table below|
|document   | JSON object  |       |Document related data if transaction status = DONE, see table |

|Parameter "transaction"       | Type    | Max. Length|  Description|
|:---------------|:--------|:----------:|:------------|
|**date**   					        | String  |    |Timestamp of the scan creation in the format YYYY-MM-DDThh:mm:ss.SSSZ|
|**status**  | String  |    |Possible states:<br>•	DONE<br>•	FAILED (if initialized acquisition is not successfully finalized within 5 minutes after creation/last update)|
|**source**     							| String  |    |Possible values: <br>• DOC\_UPLOAD (Document Verification)<br>• DOC\_API (Document Verification API)<br>• DOC\_SDK (Document Verification Mobile)|
|**merchantScanReference**		| String  |255 |Your reference for each scan |
|**customerId**       				| String  |255 |ID of the customer|
|merchantReportingCriteria    | String  |255 |Your reporting criteria for each scan|
|clientIp       	| String  |100  |IP address of the client if provided for the Document Verification API |

|Parameter "document"      | Type    | Max. Length|  Description|
|:-------------------------|:--------|:----------:|:------------|
|**status**   					| String  |    |Possible states: <br/> ⦁ UPLOADED (default) <br/> ⦁	EXTRACTED if supported document for data extraction provided <br/> ⦁	DISCARDED if no supported document for data extraction provided |
|**country**     				| String  |3   |Possible countries: <br/> ⦁ [ISO 3166-1 alpha-3](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code <br/> ⦁	XKX (Kosovo) |
|**type**     					| String  |    |Possible types: <br>⦁ CC (Credit card, front and back side)<br/>⦁	BS (Bank statement, front side) <br/>⦁	IC (Insurance card, front side) <br/>⦁	UB (Utility bill, front side) <br/>⦁	CAAP (Cash advance application, front and back side) <br/>⦁	CRC (Corporate resolution certificate, front and back side) <br/>⦁	CCS (Credit card statement, front and back side) <br/>⦁	LAG (Lease agreement, front and back side) <br/>⦁	LOAP (Loan application, front and back side) <br/>⦁	MOAP (Mortgage application, front and back side) <br/>⦁	TR (Tax return, front and back side) <br/>⦁	VT (Vehicle title, front side) <br/>⦁	VC (Voided check, front side) <br/>⦁	STUC (Student card, front side) <br/>⦁	HCC (Health care card, front side) <br/>⦁	CB (Council bill, front side) <br/>⦁	SENC (Seniors card, front side) <br/>⦁	MEDC (Medicare card, front side) <br/>⦁	BC (Birth certificate, front side) <br/>⦁	WWCC (Working with children check, front side) <br/>⦁	SS (Superannuation statement, front side) <br/>⦁	TAC (Trade association card, front side) <br/>⦁	SEL (School enrolment letter, front side) <br/>⦁	PB (Phone bill, front side) <br/>⦁	USSS (US social security card, front side) <br/>⦁	SSC (Social security card, front side) <br/>⦁	CUSTOM (Custom document type)|
|**images**	| JSON array  |  |URLs to the images of the scan (JPEG or PNG) (`*1`) |
|originalDocument |String | | URL to the originally submitted document of the scan (PDF) if available (`*1`)  |
|customDocumentCode | String  |100 |Your custom document code (maintained in your Jumio customer portal) if type = CUSTOM |
|extractedData | JSON object  | |Extracted data if status = EXTRACTED, see [Supported documents for data extraction](#supported-documents-for-data-extraction)|


(`*1`) To access the images or the originalDocument, use the HTTP GET method and HTTP Basic Authentication with your API token as the "userid" and your API secret as the "password". <br/>Set "User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION" (e.g. MyCompany MyApp/1.0.0) in the "header" section of your request. <br/>The TLS protocol is required during the TLS handshake (see [Supported cipher suites](/netverify/supported-cipher-suites.md)) and we strongly recommend using the latest version.<br/>



|Parameter "extractedData"      | Type    | Max. Length|  Description|
|:---------------|:--------|:----------:|:------------|
|firstName  | String  |255    |First name if readable|
|lastName |String |255 |Last name if readable|
|name |String | 100 | Full name if readable|
|accountNumber |String |28|Bank account number of the customer from a bank statement|
|pan |String |20 |Personal account number of credit card|
|issueDate  | String  |  |Issue date in the format YYYY-MM-DD|
|expiryDate |String | |Date of expiry in the format YYYY-MM-DD|
|ssn   | String  |255 |Social security number if readable|
|signatureAvailable  | String  |   |"true" if signature available, otherwise "false"|
|address	| JSON object  |  |Address as JSON object in raw format if status = EXTRACTED, see table below |

#### Raw Address Format

|Parameter "address"      | Max. Length|  Description|
|:---------------|:----------:|:------------|
|line1 |100 |Line item 1 |
|line2 |100 |Line item 2 |
|line3 |100 |Line item 3 |
|line4 |100 |Line item 4 |
|line5 |100 |Line item 5 |
|countryCode |3|Possible countries of residence: <br />• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br />• XKX (Kosovo)|
|postalCode |15 |Postal code|
|city |64 |City |
|subdivision |50 |Name of subdivision |

#### Supported Documents for Data Extraction

|Country      | Type | Extracted data |
|:------------|:-----|:---------------|
|AUS |UB |name, issueDate, address, dueDate |
|AUS |BS |name, issueDate, address, accountNumber |
|AUS |CCS |name, issueDate, address, cardNumberLastFourDigits |
|CAN |UB |name, issueDate, address, dueDate |
|CAN |BS |name, issueDate, address, accountNumber |
|CAN |CCS |name, issueDate, address, cardNumberLastFourDigits |
|FRA |BS |name, issueDate, address, accountNumber |
|GBR |UB |name, issueDate, address, dueDate |
|GBR |BS |name, issueDate, address, accountNumber |
|GBR |CCS |name, issueDate, address, cardNumberLastFourDigits |
|USA |UB |name, issueDate, address, dueDate |
|USA |BS |name, issueDate, address, accountNumber |
|USA |CCS |name, issueDate, address, cardNumberLastFourDigits |
|USA |SSC |firstName, lastName, ssn, signatureAvailable  |
|all |CC |name, pan, expiryDate |

<!-- Prepared table  for Release 146
|Country ([ISO 3166-1 alpha-3](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code ) | Type | Extracted data |
|:---------------|:----------|:------------|
|ALB, AND, AGO, ATG, ARG, AUS, AUT, AZE, BHS, BRB, BEL, BLZ, BEN, BOL, BIH, BWA, BRA, BFA, BDI, CMR, CAN, CPV, CAF, TCD, CHL, COL, COM, CRI, CIV, HRV, CUB, CYP, CZE, DNK, DJI, DMA, DOM, ECU, SLV, GNQ, ERI, EST, FJI, FIN, FRA, GAB, GMB, DEU, GHA, GRD, GTM, GIN, GNB, GUY, HTI, HND, HUN, ISL, IND, IDN, IRL, ITA, JAM, JPN, KEN, KIR, LVA, LSO, LBR, LIE, LTU, LUX, MDG, MWI, MYS, MLI, MLT, MHL, MRT, MUS, MEX, MDA, MCO, MNE, MOZ, NAM, NRU, NLD, NZL, NIC, NER, NGA, NOR, PAK, PLW, PAN, PNG, PRY, PER, PHL, POL, PRT, ROU, RWA, KNA, LCA, VCT, WSM, SMR, STP, SEN, SRB, SYC, SLE, SGP, SVK, SVN, SLB, SOM, ZAF, SSD, ESP, SDN, SUR, SWZ, SWE, CHE, TGO, TON, TTO, TUR, TKM, TUV, UGA, ARE, GBR, USA, URY, UZB, VUT, VAT, VEN, VNM, ZMB, ZWE |BS (Bank statement) |name, issueDate, address, accountNumber |
|all |CC (Credit card) |name, pan, expiryDate |
|ALB, AND, AGO, ATG, ARG, AUS, AUT, AZE, BHS, BRB, BEL, BLZ, BEN, BOL, BIH, BWA, BRA, BFA, BDI, CMR, CAN, CPV, CAF, TCD, CHL, COL, COM, CRI, CIV, HRV, CUB, CYP, CZE, DNK, DJI, DMA, DOM, ECU, SLV, GNQ, ERI, EST, FJI, FIN, FRA, GAB, GMB, DEU, GHA, GRD, GTM, GIN, GNB, GUY, HTI, HND, HUN, ISL, IND, IDN, IRL, ITA, JAM, JPN, KEN, KIR, LVA, LSO, LBR, LIE, LTU, LUX, MDG, MWI, MYS, MLI, MLT, MHL, MRT, MUS, MEX, MDA, MCO, MNE, MOZ, NAM, NRU, NLD, NZL, NIC, NER, NGA, NOR, PAK, PLW, PAN, PNG, PRY, PER, PHL, POL, PRT, ROU, RWA, KNA, LCA, VCT, WSM, SMR, STP, SEN, SRB, SYC, SLE, SGP, SVK, SVN, SLB, SOM, ZAF, SSD, ESP, SDN, SUR, SWZ, SWE, CHE, TGO, TON, TTO, TUR, TKM, TUV, UGA, ARE, GBR, USA, URY, UZB, VUT, VAT, VEN, VNM, ZMB, ZWE |UB (Utility Bill) |name, issueDate, address, dueDate |
|AUS, CAN, GBR, USA |CCS (Credit card statement) |name, issueDate, address, cardNumberLastFourDigits |
|USA |SSC (Social security card) |firstName, lastName, ssn, signatureAvailable  |
-->


### Sample Callbacks

#### Sample callback (URL-encoded POST): UPLOADED
```
timestamp=2017-06-06T12%3A06%3A49.016Z&scanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&document=%7B%22type%22%3A%22SSC%22%2C%22country%22%3A%22AUT%22%2C%22images%22%3A%5B%22https%3A%2F%2Fretrieval.netverify.com%2Fapi%2Fnetverify%2Fv2%2Fdocuments%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fpages%2F1%22%5D%2C%22status%22%3A%22UPLOADED%22%7D&transaction=%7B%22customerId%22%3A%22CUSTOMERID%22%2C%22date%22%3A%222014-10-17T06%3A37%3A51.969Z%22%2C%22merchantScanReference%22%3A%22YOURSCANREFERENCE%22%2C%22source%22%3A%22DOC_SDK%22%2C%22status%22%3A%22DONE%22%7D
```

#### Sample callback (URL-encoded POST): EXTRACTED
```
timestamp=2017-06-06T12%3A06%3A49.016Z&scanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&document=%7B%22type%22%3A%22SSC%22%2C%22country%22%3A%22USA%22%2C%22extractedData%22%3A%7B%22firstName%22%3A%22FIRSTNAME%22%2C%22lastName%22%3A%22LASTNAME%22%2C%22signatureAvailable%22%3Atrue%2C%22ssn%22%3A%22xxxxxxxxx%22%7D%2C%22images%22%3A%5B%22https%3A%2F%2Fretrieval.netverify.com%2Fapi%2Fnetverify%2Fv2%2Fdocuments%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fpages%2F1%22%5D%2C%22status%22%3A%22EXTRACTED%22%7D&transaction=%7B%22customerId%22%3A%22CUSTOMERID%22%2C%22date%22%3A%222014-10-17T06%3A37%3A51.969Z%22%2C%22merchantScanReference%22%3A%22YOURSCANREFERENCE%22%2C%22source%22%3A%22DOC_SDK%22%2C%22status%22%3A%22DONE%22%7D
```

#### Sample callback (URL-encoded POST): DISCARDED
```
timestamp=2017-06-06T12%3A06%3A49.016Z&scanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&document=%7B%22type%22%3A%22SSC%22%2C%22country%22%3A%22USA%22%2C%22images%22%3A%5B%22https%3A%2F%2Fretrieval.netverify.com%2Fapi%2Fnetverify%2Fv2%2Fdocuments%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fpages%2F1%22%5D%2C%22status%22%3A%22DISCARDED%22%7D&transaction=%7B%22customerId%22%3A%22CUSTOMERID%22%2C%22date%22%3A%222014-10-17T06%3A37%3A51.969Z%22%2C%22merchantScanReference%22%3A%22YOURSCANREFERENCE%22%2C%22source%22%3A%22DOC_SDK%22%2C%22status%22%3A%22DONE%22%7D
```
---
## Netverify Retrieval API
If your server was not able to process the callback, which is the authoritative answer from Jumio, you can implement RESTful HTTP GET APIs to retrieve the status, details and image(s) for a specific scan. The implementation guide can be viewed via the link below.

[View Guide](/netverify/netverify-retrieval-api.md)

---

# Supported Cipher Suites
Jumio supported cipher suites during the TLS handshake.<p>
[View supported cipher suites](/netverify/supported-cipher-suites.md)

---
&copy; Jumio Corp. 268 Lambert Avenue, Palo Alto, CA 94306
