![Jumio](/images/Jumio-Main-Banner.png)

# 1 to Many (1:n) Feature

## Table of contents

- [Benefits](#benefits)
- [Usage](#usage)
- [Best Practice](#best-practice)

---
## Benefits
- 1:n detects if the face has been seen in a previous transaction within your account
- It detects fraudsters that are using the same face but different ID data
- Biggest value in combination with 3D Liveness as IDs can be manipulated whereas liveness cannot be spoofed

## Usage
- 1:n has to be activated for your account. Contact your Jumio Account Manager for activation.
- Use our [Retrieval API](netverify-retrieval-api.md) to retrieve the data of the affected transactions to consume the data

## Best Practice
- Do not allow multiple transactions within one user account (except for retries)
	- If your use case requires multiple submissions images with a face (ID document, Selfie), please take this under consideration as you will have valid entries in your findings
- Findings will include also incomplete transactions (NO_ID_UPLOADED / NOT_READABLE) which can be ignored for 1:n as these can be valid retries  
- If a [finding](netverify-retrieval-api.md#parameter-facesearchfindings) (findings is not empty) is returned you should either
	- compare the document data (e.g. name, date of birth) with the information from the previous transaction and reject it accordingly if there is a mismatch or
	- reject the transaction automatically to be cautious
- There might be also wrong findings but as long as 1 finding is correct it should be considered as a duplicated user

---
&copy; Jumio Corporation, 395 Page Mill Road, Suite 150 Palo Alto, CA 94306
