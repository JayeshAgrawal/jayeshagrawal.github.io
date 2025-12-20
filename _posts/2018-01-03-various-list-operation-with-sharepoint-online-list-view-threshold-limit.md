---
title: "Various List Operations With SharePoint Online List View Threshold Limit"
author: "Jayesh Agrawal"
date: 2018-01-03 20:55:00 +0530
categories: [microsoft365]
tags: [sharepointonline, thresholdlimit]
seo:
  date_modified: 2021-02-05 01:55:41 +0530
---

In this article, we will learn about how we can work with SharePoint Online List view threshold. A myth with SharePoint List view threshold is the storage limit with the number of records, but it is not correct. List view threshold displays the number of record limits in SharePoint views.

According to Microsoft, the number of records storage limit is 30 million in SharePoint List & Library and the size of an individual file item or item attachment 10 gigabytes. Let's see this issue and its workaround in details:

## Issue
When we have more than 5000 records in list, we are not able to access/create custom views, lookup, do records search, filter & sort with list and we will get this type of warning message in SharePoint online:

![Warning Message](https://2.bp.blogspot.com/-Xp5pRBWlBOo/W0DSX-kd75I/AAAAAAAAAe0/KHJIaPhiBVMjObDY0wYHnRZ_BgNqfY-rwCLcBGAs/s640/Untitled1.png)
## Workaround
- By default, the list view threshold is configured at 5,000 items.
- If SharePoint is an on-premise server, we can increase items of the list view threshold. But it is not possible to increase items of List view threshold with SharePoint online because it uses the Large List Resource Throttling feature.
- So, how we can display records of that list with SharePoint Views? Here, List column indexing comes into place.

Suppose that we have a 'Product' list which has around 5700+ records in the list as below:

| Column Name | Datatype               |
|-------------|------------------------|
| Name        | Single line of text    |
| MfgYear     | Single line of text    |
| Price       | Currency               |
| IsDiscount  | Yes/No                 |

If we want to create a view base on some condition like 'MfgYear' equal to 2017 which is querying record of Product list, it will display a List view threshold warning message instead of records because we will be querying 5000+ records.

To display records in view, we will add index on 'MfgYear' Column and if filtering record criteria of this ‘MfgYear’ column is fewer than 5000 records it will be displayed records.

## Note 
If we exceeded the List View Threshold and have been blocked, we can normally still add indexes to columns when we have fewer than 20,000 items in our list or library.

Now, we will perform some use case operation with list as following:
- Update List column data based on another column,
- Delete records based on condition.

Let's see in detail:
# Update List column data based on another column
If we want to add a new column ‘DiscountPrice’ in that list and update that column data based on 'Price' which record have 'IsDiscount' is true and 'MfgYear' is 2017. But If we will use COSM or REST then, we will not be able to directly fetch those records due to List Threshold Warning.

Hence, we will iterate each record using 'ListItemCollectionPosition' property of list to avoid Threshold issue and, we will be taking 200 rows view batch using CAML query.

Please refer to the below code snippet to see how we can update list records:

```c#
using (ClientContext context = new ClientContext(siteUrl))  
{  
    var onlineCredentials = new SharePointOnlineCredentials(login, securePassword);  
    context.Credentials = onlineCredentials;  
    SP.List myList = context.Web.Lists.GetByTitle("Product"); 
    CamlQuery camlQuery = new CamlQuery();  
    camlQuery.ViewXml = string.Format("<View><RowLimit>200</RowLimit></View>"); 
    do  
    {  
        ListItemCollection collListItem = myList.GetItems(camlQuery);  
        try  
        {  
            context.Load(collListItem);  
            context.ExecuteQuery();  
            if (collListItem.Any())  
            { 
                foreach (var item in collListItem)  
                { 
                    if (dt != null && dt.Rows.Count > 0)  
                    {  
                        foreach (DataRow dtRow in dt.Rows)  
                        {  
                            if (Convert.ToBoolean(item["IsDiscount"]) == true && Convert.ToInt32(item["MfgYear"]) 2017 )  
                            {  
                                    item["DiscountPrice"] = Convert.ToDouble(dtRow["Price"]) * 0.10;  
                                    item.Update();  
                                    context.ExecuteQuery();  
                            }  
                        }  
                    }  
                }  
            }  
        }  
        catch (Exception e)  
        {  
           // log exception   
        } 
        // It is important to update ListItemCollectionPosition of objectcamlQuery with current position 
        camlQuery.ListItemCollectionPosition = collListItem.ListItemCollectionPosition;  
         
   // ListItemCollectionPosition is null if there is no other page  
    } while (camlQuery.ListItemCollectionPosition != null);  
} 
``` 
# Delete records based on condition
Suppose we want to delete records in which Price column will have zero value from 'Product' list, but we will not be directly filtering records due to the list threshold limit warning. And if we will delete each record one by one it will be very time consuming. So, we will be again taking 200 rows view batch using CAML query and deleting records in batch.

If we want to delete all records from 'Product' list, then we will not need to filter records.

In such cases we will not use 'ListItemCollectionPosition' property because List Item Position will be moved automatically after deleting each record. Please refer to the below code snippet to see how we can update list records:

```c#
using (ClientContext context = new ClientContext(siteUrl))  
{  
    var onlineCredentials = new SharePointOnlineCredentials(login, securePassword);  
    context.Credentials = onlineCredentials;  
    SP.List myList = context.Web.Lists.GetByTitle("Product"); 
    CamlQuery camlQuery = new CamlQuery();  
    camlQuery.ViewXml = string.Format("<View><RowLimit>200</RowLimit></View>"); 
    do  
    {  
        ListItemCollection collListItem = myList.GetItems(camlQuery);  
        try  
        {  
            context.Load(collListItem);  
            context.ExecuteQuery();  
            if (collListItem.Any())  
            { 
                foreach (var item in collListItem)  
                { 
                    if (dt != null && dt.Rows.Count > 0)  
                    {  
                        foreach (DataRow dtRow in dt.Rows)  
                        {  
                            if (Convert ToDouble(item["Price"]) == 0)  
                            {  
                                    item.DeleteObject();  
                            }  
                        }  
                        context.ExecuteQuery(); 
                    }  
                }  
            }  
        }  
        catch (Exception e)  
        {  
            //Log Exception    
        }  
       camlQuery.ListItemCollectionPosition = collListItem.ListItemCollectionPosition; 
    } while (camlQuery.ListItemCollectionPosition != null);  
}  
```
## Conclusion
This is how we can work with SharePoint Online List View Threshold. For more detail, please refer to [this](https://support.microsoft.com/en-us/office/manage-large-lists-and-libraries-b8588dae-9387-48c2-9248-c24122f07c59?ui=en-us&rs=en-us&ad=us) article from Microsoft.
