![Jumio](/images/netverify.jpg)

# Netverify for Salesforce Installation Guide

This is a walkthrough for the installation of **Jumio Netverify Identity Verification** for **Salesforce**, including configuration requirements.

Netverify for Salesforce is an application which allows orgs to request and verify their Contacts and/or Leads. By navigating to the contact's page they can request a verification by simply clicking a button. An email will be sent to the contact containing a redirect link to Netverify. Results can be viewed as soon as the verification process has been completed by Jumio.

This installation guide is for the **Salesforce Lightning Experience**. Switch Salesforce Lightning to follow this guide. You can return to Classic after installation is complete.

For additional information, you can go to the Jumio Knowledge Base, [here](https://support.jumio.com).

Feedback Survey

---
## Table of Contents
- [Salesforce Jumio Package](#salesforce-jumio-package)
- [Salesforce Setup](#salesforce-setup)
 - [Edit Permission Sets](#edit-permission-sets)
		- [Netverify Setup User Permission Set](#netverify-setup-user-permission-set)
		- [Netverify Standard User Permission Set](#netverify-standard-user-permission-sets)
- [Domain and Site Setup](#domain-and-site-setup)
	- [Domain Setup](#domain-setup)
	- [Site Setup](#site-setup)
		- [Enable Apex Class Access](#enable-apex-class-access)
		- [Assign Permission Set to the Guest User Profile](#assign-permission-set-to-the-guest-user-profile)
		- [Whitelist Jumio callback IPs](#whitelist-jumio-callback-ips)
- [Run Netverify Setup](#run-netverify-setup)
	- [System Connections](#system-connections)
	- [Verification Settings](#verification-settings)
- [Edit Page Layouts](#edit-page-layouts)
	- [Add Netverify Component to a Lightning Record Page](#add-netverify-component-to-a-lightning-record-page)
	- [Add Netverify Component to a Classic Salesforce Page](#add-netverify-component-to-a-classic-salesforce-page)
- [Email](#email)
	- [Email Settings](#email-settings)
	- [Editing Email Template](#editing-email-template)
- [Supplementary Configuration](#supplementary-configuration)
	- [Sample for Linking Orphaned Verifications Using Process Builder](#Sample-for-Linking-Orphaned-Verifications-Using-Process-Builder)
		- [Create a Lookup Relationship to Verification from Lead Object](#Create-a-Lookup-Relationship-to-Verification-from-Lead-Object)
		- [Process 1 - Create a Customer Object for an Orphaned Verification](#Process-1-\--Create-a-Customer-Object-for-an-Orphaned-Verification)
		- [Process 2 - Check if a new Lead should link up to a Verification](#Process-2-\--Check-if-a-new-Lead-should-link-up-to-a-Verification)
	- [Run Reports to find "Orphaned" Verifications](#Run-Reports-to-find-"Orphaned"-Verifications)
	- [Orphaned Verifications List View in Verifications Object](#Orphaned-Verifications-List-View-in-Verifications-Object)
	- [Notification Sample for Error Log Email Alert](#Notification-Sample-for-Error-Log-Email-Alert)

---
# Salesforce Jumio Package

Jumio will provide signed customers with a link to the Netverify Salesforce package.

- Go to the link provided to you. If you are performingthe install on a **sandbox** environment, make sure to update the **domain** of the URL provided to your sandbox URL.

![Jumio](/images/nvsfimages/01_package1.jpeg)

- Select "**Install for Admins Only**", then click **Install**
- Check "**Yes, grant access to these third-party websites**"
- Click **Done** when installation is complete.

*Installation will typical take a few minutes.

# Salesforce Setup

## Edit Permission Sets

The Jumio package will include two permission sets, **Netverify Setup User** and **Netverify Standard User**. These permissions will need to be cloned and additional configurations need to be added.

![Jumio](/images/nvsfimages/02_permission01.jpeg)

### Netverify Setup User Permission Set

The **Netverify Setup User** will be for the system admin performing the installation.

- Navigate to the Quick Find bar in Setup and search "**Permission Sets**", click **Clone** next to the **Netverify Setup User**
- Enter a new label "**Netverify Setup User Clone**", the **API Name** should update automatically when selecting the field
	- If not, name **Netverify\_Setup\_User\_Clone**
- Save the clone

![Jumio](/images/nvsfimages/02_permission02.jpeg)

- Select into the **Netverify Setup User Clone** Permission Set

![Jumio](/images/nvsfimages/02_permission03.jpeg)

- Go to **System Permissions**

![Jumio](/images/nvsfimages/02_permission04.jpeg)

- Click **Edit** and enable **Customize Application**, then save changes

![Jumio](/images/nvsfimages/02_permission05.jpeg)

- Next, click on **Manage Assignments**

![Jumio](/images/nvsfimages/02_permission06.jpeg)

- Click on **Add Assignments** and **Assign** admin user(s)

### Netverify Standard User Permission Set

The **Netverify Standard User** will be for users that will be using the Netverify component and setting up the callback URL Site.

- First, repeat the steps to clone the **Netverify Standard User** Permission Set
- Then navigate to the **Netverify Standard User Clone** Permission Set

![Jumio](/images/nvsfimages/02_permission07.jpeg)

- Go to **Object Settings**

![Jumio](/images/nvsfimages/02_permission08.jpeg)

- If you are going to verify both **Contact** and **Lead** objects you will need to allow read access, for **both** objects, to the following:
	* First Name, Last Name, Email
	* These will come up again when we get to the **Verification Setup** in the installation.
- Select **Contacts**

![Jumio](/images/nvsfimages/02_permission09.jpeg)

- Enable **Read** permission to **Email** (First and Last Name should already be enabled by default)

![Jumio](/images/nvsfimages/02_permission10.jpeg)

![Jumio](/images/nvsfimages/02_permission11.jpeg)

- **Repeat this for the Lead object, if you intend to verify Leads**
- To do this, do the following:
	* Return to **Object Settings**
	* Select **Lead**
	* Select **Read** access on **Email** and **Save**
- Next, go to **Manage Assignments**

![Jumio](/images/nvsfimages/02_permission12.jpeg)

- Click on **Add Assignments** and **Assign** standard user(s)

# Domain and Site Setup
This step is to setup the callback URL using the public Domain and Site provided by Salesforce. Without this step the Netverify component will not be populated by the verification results automatically.

Results can be pulled manually from the component.

## Domain Setup
If you have already setup a public **Domain** for your org in Salesforce, you can skip this step.

- In the Quick Find, search "**My Domain**"
- Check for the availability of a domain and register it

![Jumio](/images/nvsfimages/03_domain1.jpeg)

- Refresh the page after the domain has successfully been registers. You will need to **Log in** to the new domain to continue.
- Click **Log in**, then **Deploy to Users**

![Jumio](/images/nvsfimages/03_domain2.jpeg)

## Site Setup
This step will be to setup a callback URL from within the Domain you just setup.

- On the **Domain** page, if you have not, click **Register My Salesforce Site Domain**
	- This button will not be visible if you've already done so
- To create a new site, click **New** under Site

![Jumio](/images/nvsfimages/04_site1.jpeg)

- Fill out and **Save** the mandatory fields

![Jumio](/images/nvsfimages/04_site2.jpeg)

### Enable Apex Class Access
Once you have a Salesforce Site, you will need to allow access to the **netverify.webhook** Apex Class. This will enable the website to receive and parse the information from the callback.

- Click on your newly created **Site**, back on your Domain page
- Next, navigate to **Public Access Settings**

![Jumio](/images/nvsfimages/05_apex1.jpeg)

- Go to **Apex Class Access**

![Jumio](/images/nvsfimages/05_apex2.jpeg)

- Add **netverify.webhook** to the Enabled Apex Classes, then **Save** changes

![Jumio](/images/nvsfimages/05_apex3.jpeg)

### Assign Permission Set to the Guest User Profile
You will only be able to assign this Permission Set to the Guest User through the **Site** page. Without properly assigning this Permission Set, the callback will not be received.

- Return to the Profile Overview and click **Assigned Users**

![Jumio](/images/nvsfimages/06_guest1.jpeg)

- Select **Site Guest User**

![Jumio](/images/nvsfimages/06_guest2.jpeg)

- Scroll down and select **Edit Assignments** with Permission Set Assignments

![Jumio](/images/nvsfimages/06_guest3.jpeg)

- Add the **Netverify Standard User Clone** Permission Set to the Enabled Permission Sets, then **Save** changes

![Jumio](/images/nvsfimages/06_guest4.jpeg)

### Whitelist Jumio callback IPs
Whitelisting the IPs that Jumio sends callbacks from will allow for the callbacks to make it through the Salesforce firewalls.

You can find the list of IPs here:
https://github.com/Jumio/implementation-guides/blob/master/netverify/callback.md

- Return to your site's Public Access Settings page, then select **Login IP Ranges** under System

![Jumio](/images/nvsfimages/07_ips1.jpeg)

- Add a new IP range for each IP, the **Start** and **End Address** should be the same

![Jumio](/images/nvsfimages/07_ips2.jpeg)

- Salesforce will ask you to confirm as the range does not include your current IP

![Jumio](/images/nvsfimages/07_ips3.jpeg)

- **Repeat until all IPs have been added**

# Run the Netverify Setup
Included within the Netverify package is an app. This is where the configuration of the Netverify component happens. This includes entering credentials and choosing what is viewable in the component.

From **Lightning**

- Click on the **App Launcher**

![Jumio](/images/nvsfimages/08_setup1.jpeg)

- Select **Netverify Setup** app

![Jumio](/images/nvsfimages/08_setup2.jpeg)

From **Classic**

- Select the Force.com App Menu, then select the **Netverify Setup** app

![Jumio](/images/nvsfimages/08_setup2.jpeg)

## System Connections
- Click **Get Started** once the app launches

![Jumio](/images/nvsfimages/09_system1.jpeg)

- The first step is **System Connections**, you will need to enter your **credentials** and select your **data center**
	- You can find your **credentials** in the Jumio Customer Portal, link is provided depending on selected data center

![Jumio](/images/nvsfimages/09_system2.jpeg)

- In the Jumio Customer Portal, **credentials** are under **Settings** and **API credentials**

![Jumio](/images/nvsfimages/09_system3.jpeg)

- Copy and paste your token in secret in the provided fields in the Netverify Setup, then click **Next**
- Next, you will be select the **Salesforce Public Site** you configured previously
	- If you have not setup a Site yet, return to the [Domain and Site Setup] (Domain and Site Setup)

![Jumio](/images/nvsfimages/09_system4.jpeg)

- Copy the provided URL and paste it into the Jumio Customer Portal under **Settings** and **Application Settings**
	- You will also need to enter your Jumio Customer Portal password to save your changes at the bottom of the Portal page, then select **Next**
	- Double check the domain of the provided **URL** that is matches your org domain. This can be different if you're working in a sandbox

![Jumio](/images/nvsfimages/09_system5.jpeg)

- Back in Salesforce, select **Continue** to complete System Connections

## Verification Settings
The next step is for specifying supported Salesforce objects and adjust the Lightning component layout. Each object requires a field to be used as the identifier, which will match Netverify Verification's Customer ID field.

**Do not use email as an identifier here, Emails passed in this field are considered security risk.**

- Select your Salesforce object(s) and their **Identifiers** and click **Next**

![Jumio](/images/nvsfimages/10_verification1.jpeg)

- The next section is for **Customer Field Mapping** where you specify which standard Salesforce objects are used to map results to Contacts and Leads

![Jumio](/images/nvsfimages/10_verification2.jpeg)

- The following section allows you to choose which parameters from the **Verification result** will be shown within the **Netverify component**

![Jumio](/images/nvsfimages/10_verification3.jpeg)

- Select **Next**, then **Finish** to complete the setup.

# Edit Page Layouts
This section is for adding the Netverify component to your Salesforce pages. The suported Objects for this section are Contact, Lead, Account, Case and Opportunity. This guide will go through the Lead setup, repeat the process for other objects which you've selected within the setup.

## Add Netverify Component to a Lightning Record Page 
This is the step-by-step setup to add the Netverify component to a Lightning page using the Lightning setup.

- Navigate to a **Lead** page in your org
	- Click the gear icon
	- Select **Edit Page**

![Jumio](/images/nvsfimages/11_page01.jpeg)

- In this example, we will add an additional tab named **Verification**, where we will put the Netverify component
- Select the **Tab** bar, then select **Add Tab**

![Jumio](/images/nvsfimages/11_page02.jpeg)

- Select the newly created tab, select **Custom** for **Tab Label**, we will name ours "**Verification**"

![Jumio](/images/nvsfimages/11_page03.jpeg)

- Select the **Verifications** tab
	- Drag the **Netverify by Jumio**, under **Custom - Managed**, onto the empty tab
	- **Save**

![Jumio](/images/nvsfimages/11_page04.jpeg)

## Add Netverify Component to a Classic Salesforce Page
If you have users that will be using Netverify from the Salesforce Classic view, you will need to also add the Netverify Visualforce Page to the standard object Page Layouts that you selected within the setup. We will add Visualforce Page to a Leak Page Layout.

- Navigate to the **Object Manager**
	- Select the **Lead** object

![Jumio](/images/nvsfimages/11_page05.jpeg)

- Select **Page Layouts** on the left
	- Click **Lead Layout**

![Jumio](/images/nvsfimages/11_page06.jpeg)

- Select **Visualforce Pages** from the menu
	- Drag the **Section** object onto the Page Layout where you want the Netverify Visualforce Page to be

![Jumio](/images/nvsfimages/11_page07.jpeg)

- **Section Properties** should appear, enter the following
	- Section Name - Verifications
	- Display Section Header On - Details Page
	- Layout - 1-Column
	- Click **Okay**

![Jumio](/images/nvsfimages/11_page08.jpeg)

- Find **netverifyLead** from the Visualforce Page and drag it into the newly created section
	- Select the tool icon to edit the properties

![Jumio](/images/nvsfimages/11_page09.jpeg)

- Enter the following properties
	- Height minimum 350 (pixels)
	- Check **Show scrollbars**
	- Click **Okay**

![Jumio](/images/nvsfimages/11_page10.jpeg)

- **Save** the new Page Layout

# Emails
This section will guide you on how to setup the emails that are being sent to your contacts wheny ou request a verification.

## Email Setting
To allow Salesforce to send emails to the users containing the Netverify URL, you **must** make sure that access level is set to "**All email**".

 - Go to **Salesforce Setup**
 - Search "email" in the quickfind bar
 	- Select **Deliverability**
 	- Under **Access to Send Email (All Email Services)**, set "Access level" to "**All email**"
 	- **Save** changes

![Jumio](/images/nvsfimages/12_email1.jpeg)

## Editing Email Template
We will show you were to locate teh email template which is provided within the Netverify package. This template can be customized. However, the lines of HTML code that include "**netverify\_\_Link\_\_c**" must remin within the template. This is the place holder for the link to the Verification.

You can find more information on the Email Templates [here](https://help.salesforce.com/articleView?id=email_templates_landing_page.htm&type=5).

To access the packaged Email Template:

- Navigate to the quickfind in Setup and search "**Classic Email Templates**"
	- Select the **Netverify by Jumio** from the dropdown
	- Select **Netverify Verification Request**

![Jumio](/images/nvsfimages/12_email2.jpeg)

- You can Edit the template to better fit your org

# Supplementary Configuration
This section is to setup a way to manage and catch "orphaned" transactions. These are the transactions which do not have a Contact in Salesforce and need to be linked to one.

## Web-to-Lead
This section will show you how to create a Web to Lead form to place on your Company website, or any where else Leads would potentially express interest. With a standard Salesforce subscription you are allowed up to 500 Leads per day through the form.

For more information regarding Web-to-Lead you can go [here](https://help.salesforce.com/articleView?id=setting_up_web-to-lead.htm&type=5).

- Navigate to the quickfind in the Setup and search "**Web-to-Lead**"
	- Make sure you have **Web-to-Lead Enabled** checked, if not you can check it by clicking **Edit**
	- Click **Create Web-to-Lead Form**

![Jumio](/images/nvsfimages/13_weblead1.jpeg)

- Add fields you want in your webform to the **Selected Fields** column
	- Enter your **Return URL** (e.g. Thank you page)
	- You may check whether you want to Enable spam filtering (recommended)
		- If you choose this option you will need to enter in your reCAPTCHA API Key Pair
- Click **Generate** when done

![Jumio](/images/nvsfimages/13_weblead2.jpeg)

- Copy and paste the sample HTML provided and send it to your webmaster to add to your website
- Click **Finished**

![Jumio](/images/nvsfimages/13_weblead3.jpeg)

## Sample for Linking Orphaned Verifications Using Process Builder
This section will use the Process Builder within Salesforce to create new processes to lookup and link "orphaned" verifications

For these to work, the Customer Id must be assigned in the Netverify Setup and that field must be passed back in the callback.

### Create a Lookup Relationship to Verification from Lead Object
This part is to create a Lookup Relationship with Lead to Verification. This is required for creating the processes.

- Go to **Object Manager**

![Jumio](/images/nvsfimages/14_lookup1.jpeg)

- Select **Lead** from the list of objects
- Go to **Field & Relationships**
	- Select **New**

![Jumio](/images/nvsfimages/14_lookup2.jpeg)

- select **Lookup Relationship** 
	- select **Next**

![Jumio](/images/nvsfimages/14_lookup3.jpeg)

- Select **Verification**
	- click **Next**
 
![Jumio](/images/nvsfimages/14_lookup4.jpeg)


- Click **Next** through the rest of the setup and **Save** the new Lookup

### Process 1 - Create a Customer Object for an Orphaned Verification

- Search “**Process Builder**” in the Quick Find
- Select **New**
	- Enter in a Process Name - We name ours, **Create a Customer Object (Lead) for Orphaned Verification** 
		- The API Name should automatically populate when selecting in the field
	- Select the process starts when - **a record changes**
	- Select **Save**

![Jumio](/images/nvsfimages/15_process1.jpeg)

- Select **Add Object**
	- Select **Object - Verification**
	- Select start the process - **only when a record is created**
	- Select **Save**

![Jumio](/images/nvsfimages/15_process2.jpeg)

- Select **Add Criteria**
	- Enter a Criteria Name - Our will be **Orphaned Verification with Customer Id and Names**
	- Select Criteria for Executing Actions - **Conditions are met**
	- Set Conditions (Field, Operator, Type, Value):
		- Field - **Salesforce Customer ID**, Operator - **Is null**, Type - **Boolean**, Value - **True**
		- Field - **Customer Id**, Operator - **Is null**, Type - **Boolean**, Value - **False**
		- Field - **First Name**, Operator - **Is null**, Type - **Boolean**, Value - **False**
		- Field - **Last Name**, Operator - **Is null**, Type - **Boolean**, Value - **False**
	- Select Conditions - **All of the conditions are met (AND)**
	- Select **Save**

![Jumio](/images/nvsfimages/15_process3.jpeg)

- Select **Add Action**
	- Select the Action Type - **Create a Record**
	- Enter an Action Name - **Create Lead**
	- Select Record Type - **Lead**
	- Set Field Values (Field, Type, Value):
		- Field - **Company**, Type - **String**, Value - **Auto Generated Lead**
		- Field - **Last Name**, Type - **Field Reference**, Value - **Last Name**
		- Field - **First Name**, Type - **Field Reference**, Value - **First Name**
		- Field - **Verification**, Type - **Field Reference**, Value - **Record Id**
			- Note: Verification was a custom Lookup field created on the selected customer object post-install in order to make this sample work. This Verification field on Lead, looks up to the Verification object
		- Select **Save**

![Jumio](/images/nvsfimages/15_process4.jpeg)

- Select **Activate** 
- Select **Confirm**

### Process 2 - Check if a new Lead should link up to a Verification
This process will be to see if there is an "orphaned" Verification for the new Lead that was just created.

- Search “**Process Builder**” in the Quick Find
- Select **New**
	- Enter in a Process Name - Ours will be **Check new Lead links to a Verification**
		- The API Name will automatically populate when selecting in the field
	- The process starts when - **a record changes**
	- Select **Save**

![Jumio](/images/nvsfimages/15_process5.jpeg)

- Select **Add Object**
	- Select Object - **Lead**
	- Start the process - **only when a record is created**
	- Select **Save**

![Jumio](/images/nvsfimages/15_process6.jpeg)

- Select **Add Criteria**
	- Enter a Criteria Name - **Is new Lead with Verification?**
	- Select Criteria for Executing Actions - **Conditions are met**
	- Set Conditions (Field, Operator, Type, Value):
		- Field - **Verification** (custom Lookup field that would need to be created on Lead that looks up to Verification) 
		- Operator - **Does not equal** 
		- Type - **Global Constant** 
		- Value - **$GlobalConstant.Null**
	- Select Conditions - **All of the conditions are met (AND)**
	- Select **Save**

![Jumio](/images/nvsfimages/15_process7.jpeg)

- Select **Add Action**
	- Select the Action Type - **Update Records**
	- Enter an Action Name - **Populate Salesforce Customer Id with Lead Id**
	- Select a Record Type to Update - Check “**Select a record related to the Lead**” - 
		- Select **Verification**
	- Criteria for Updating Records - **No criteria - just update the records!**
Set new field values for the records you update (Field, Type, Value):
		- Field - **Salesforce Customer Id** 
		- Type - **Field Reference**
		- Value - **Lead Id**
	- Select **Save**

![Jumio](/images/nvsfimages/15_process8.jpeg)

- Select **Activate**
- Select **Confirm**

## Run Reports to find "Orphaned" Verifications
This section will show you how to run the packaged report to find Verifications without a Salesforce Contact.

To run Reports using the packaged "Orphaned Verifications Reporting" Report Type follow these steps:

- Navigate to **Reports** within your Org
	- Click the **App Launcher**
	- Select **Reports**

![Jumio](/images/nvsfimages/16_reports1.jpeg)

![Jumio](/images/nvsfimages/16_reports2.jpeg)

- Select **New Report**
- Select **Other Reports**
	- Select **Orphaned Verifications Reporting**
	- Click **Continue**

![Jumio](/images/nvsfimages/16_reports3.jpeg)

- Select the **FILTERS** tab
	- Select Show - **All verifications**
	- Click **Add Filter** in the text field
		- Choose **Salesforce Customer Id**, **equals**, leave blank
	- Click **Apply**

![Jumio](/images/nvsfimages/16_reports4.jpeg)

- Then **Save** the report
- Enter Report Name - **Orphaned Verifications**
	- Report Unique Name will automatically populate
- Report Description - **Verifications where the Salesforce Customer Id is null** 
- Report Folder - **Unfiled Public Reports**
	- Create a new Folder or select a different location

![Jumio](/images/nvsfimages/16_reports5.jpeg)

- **Save**

## Orphaned Verifications List View in Verifications Object
The Verification object that was included in the Jumio package has an Orphaned Verifications View, among a couple other views that can be useful for viewing all transactions.

You can find the Verifications object in the App Launcher.

![Jumio](/images/nvsfimages/17_listview1.jpeg)

![Jumio](/images/nvsfimages/17_listview2.jpeg)

## Notification Sample for Error Log Email Alert

This Process Builder sample shows how to automate an Email to a System Admin when an Error Log record gets created.

If you perform this step, you will want to create a new Custom Object tab for the Error Log object. And if the User is the System Admin receiving the email alert and viewing the Error Log Salesforce record, go to the the Netverify Setup User Permission Set in Object Settings under Error Logs, make the Tab Visible and Available.

