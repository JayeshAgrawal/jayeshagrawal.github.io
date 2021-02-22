---
title: "Send an Email Reminder Notification Based on an Expiration Date using Power Automate"
author: "Jayesh Agrawal"
date: 2020-02-08 14:55:00 +0530
categories: [Microsoft365]
tags: [MicrosoftFlow, PowerAutomate]
seo:
  date_modified: 2021-02-18 01:55:41 +0530
---

## Introduction 
 
In this article, we are going to create an email reminder notification based on an expiration date using Power Automate. We will create a flow that’s run daily that reads & filters data from a SharePoint list that has list items that expire in the next 5 days. Then it will send an expiration notification email to a particular user.
 
## SharePoint List - "ProductSales"

 | Columns    | Type      |
 |------------|-----------|
 | Title	    | Text      |
 | ExpiryDate	| DateTime  |
 
Let's log in to Office.com with office 365 credentials. Go to the “Power Automate” tile and click on it.
 
It will be redirected to Power Automate Page. Click on Create button from left navigation.
 
Click on the scheduled flow tile. It will open the popup.
 
![Flow](https://1.bp.blogspot.com/-Fbs1KY3FmLI/Xkz3-n8pHeI/AAAAAAAABKk/vxbeBlFHkKwrvV-7q_qmGAy-YTzKXWWewCEwYBhgL/s640/001.png)
  
## Create Recurrence Flow
- Enter the flow name and select a specific time when we want to schedule this workflow. Then, click on the create button.

![Flow](https://1.bp.blogspot.com/-Cq91Qh93gFM/Xkz3-b5gYtI/AAAAAAAABKg/eHTGUqHXzzY-Pxyepw26DHWMnTkbNM0rQCEwYBhgL/s640/002.png)

- It will open the Edit Flow page where we can see a Recurring Action.
- Now we are fetching SharePoint List Items, which are going to expire in the next 5 days. We can do this using various flow actions that are available in Power Automate, such as Get Items & Get Files (SharePoint) action. Here, we are using the REST API to get data from the SharePoint List.
- Click on the new step button and find & select the “Send Http Request for SharePoint” action.

 ![Flow](https://1.bp.blogspot.com/-Dj8qmJ8Pk0c/Xkz3-VfiQWI/AAAAAAAABK8/mmXBQTRe_XQnibT_ZetusJTmai3VgVCigCEwYBhgL/s640/003.png)
 
## "Send Http Request for SharePoint"
- Enter the site address and URI for the [REST API](https://docs.microsoft.com/en-us/sharepoint/dev/sp-add-ins/complete-basic-operations-using-sharepoint-rest-endpoints) parameter, which is for getting the "ProductSales" list item and Headers parameters.
- Below we have prepared the Rest API to get the Title, Expiry Date field from SharePoint List with an expiry date filter which is going to expire in the next 5 days.

```cs
_api/web/lists/getbytitle('ProductSales')/items?$select=Title,ExpiryDate&$filter=ExpiryDate lt datetime'@{addDays(utcNow(),5)}' and ExpiryDate gt datetime'@{utcNow()}'
```  
![Flow](https://1.bp.blogspot.com/-p85sKF0XI6g/Xkz3-_TnriI/AAAAAAAABLA/SLZsd2hQYw0USn5Jaz_FsALK5Uq5KcTLwCEwYBhgL/s640/004.png)

- Now we are going to test this flow. Click on the Test button from the right side of the top menu.
- Then, select "I’ll perform action trigger" option and click on Save & Test button.
- After this, Click the run flow button. Then, click the done button.
- We can see the “flow run successfully” message. Then expand the “Send Http Request SharePoint” action.
- Then, Scroll down the screen. You should see the copy body content, where we see data of SharePoint List and click on the Edit button from the right side of the top menu.

## Parse JSON
- Let's parse the API response using the "Parse JSON" action.
- Click on Next Step and find select the “Parse JSON” action.
- Click on the content field, select the body of “Send Http Request SharePoint” from Dynamic Content popup.
- Now click on Generate from the sample button. Enter the copied SharePoint List data content that we copied earlier. We can see the schema is generated now.

![Flow](https://1.bp.blogspot.com/-eoS5c9a8YrI/Xkz3_NHipoI/AAAAAAAABLA/d1R_iN8ytU4wf0iGKB47Xjmb6jT5whsDACEwYBhgL/s640/005.png)
 
## Create an HTML Table
- Click on Next Step and find select “Create HTML table” action.
- Click on the content field, and select the results of “Parse JSON” from the Dynamic Content popup.

![Flow](https://1.bp.blogspot.com/-sGHYP5DBapg/Xkz3_Ihg3GI/AAAAAAAABLI/ToCPyOccxrc9Pqyc5aPULcfFVv2Syt9QQCEwYBhgL/s640/006.png)
 
## Setup Send an Email
- Click on Next Step and find select “Send an Email” action.
- Enter Email Address into the field and add a subject.
- Click on the body, select Outputs of HTML content from the Dynamic Content popup.
- Click on Save Flow.

![Flow](https://1.bp.blogspot.com/-5y40GFLaZkQ/Xkz3_r4Lo0I/AAAAAAAABLE/3itB4L0FU7s6ThqObe11gXhdgvE_mspFACEwYBhgL/s640/007.png)

- Now let’s test the flow. Click on the Test button from the right side of the top menu and click the Run flow button. Then click the done button and verify the email.

## Conclusion
Now we can send an email reminder notification based on an expiration date using Power Automate that will run every and check if any item is expired, then it will send email to the particular user.