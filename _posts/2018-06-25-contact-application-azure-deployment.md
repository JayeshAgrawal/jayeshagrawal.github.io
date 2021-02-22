---
title: "Contact Application - Azure Deployment"
author: "Jayesh Agrawal"
date: 2018-06-25 20:55:00 +0530
categories: [Azure]
tags: [ContactApplication, AspNetCore, Angular, ApiDevelopment]
seo:
  date_modified: 2021-02-20 01:55:41 +0530
---

In this article, we are going to deploy/host a contact application with Visual Studio code to Azure web apps.
Before proceeding with this article I would like to request you that please go through the  first two parts of contact application:

- [Contact Application using ASP.NET Core Web API, Angular 6.0, and Visual Studio Code Part One](https://crack-codes.blogspot.com/2018/05/contact-app-using-aspnet-core-angular-part-one.html)
In the article, we will set up ASP.NET Core Web API project, and develop the Web API for contact CRUD operations.
- [Contact Application using ASP.NET Core Web API, Angular 6.0, and Visual Studio Code Part Two](https://crack-codes.blogspot.com/2018/06/contact-app-using-aspnet-core-angular-part-two.html)
In the article, we will setup Angular 6 within ASP.NET Core Web API Project, and develop the contact form & list component using Angular Material UI that will consume Web API which we have created in Part One.

Let’s start with database deployment as Azure SQL databases.

- Login to [Azure Portal](https://portal.azure.com/).

![SQLDeployment1](https://3.bp.blogspot.com/-5H2OnIvT4sQ/W0HBrTY5TfI/AAAAAAAAAj4/KziQm_E-FPQCP8i604Rk3khpVuNwdc8swCEwYBhgL/s320/sql1.png)

- Go to left navigation on SQL databases. It will open a popup for SQL databases.
- Click on Add or Create SQL database Button.

![SQLDeployment2](https://2.bp.blogspot.com/-vFlta5cNfcs/W0HBuzgRBII/AAAAAAAAAkk/aR8n_n1AKlgD4BcPZttsTGaectMTjplSgCEwYBhgL/s1600/sql2.png)

- It will open a form where we need to specify SQL Server Instance, Resource group etc. Here, we can create new SQL Server Instance with admin user if we didn't have an already existing SQL Server Instance.

![SQLDeployment3](https://2.bp.blogspot.com/-ffIoZ8zsVI8/W0HBviOi2NI/AAAAAAAAAlY/uWkx-RJYpL0Ullfsxu4Tma6HOyJlohtdACEwYBhgL/s1600/sql3.png)

- Then, we need to configure Price Tier for SQL database as per our requirement. Currently, we have selected Basic Price Tier for this.

![SQLDeployment4](https://2.bp.blogspot.com/-HZeqriQT0dQ/W0HBwFY_d4I/AAAAAAAAAlU/9ohlAdF8qmcnezivjh8QRyTvX-BODtFmACEwYBhgL/s1600/sql4.png)

- Here we also tick pin to dashboard option in form and Click on create a button to submit for Azure SQL Database deployment.

![SQLDeployment5](https://2.bp.blogspot.com/-F74WIQf0bvY/W0HBwYK60KI/AAAAAAAAAlk/Xq8i3HVz0_Uuz1ceLkl-INhwpfRf95_oACEwYBhgL/s1600/sql5.png)

- It will deploy Azure SQL Database & Server and will take a minute to deploy SQL Server.
- We can see SQL Database pinned to dashboard after successful deployment of SQL Database. Click on "contactapp" SQL database from the dashboard. It will open "contactapp" SQL database page and left side navigation will have various options to manage SQL database. Here, we can see the connection of string to connect "contactapp" database.

![SQLDeployment6](https://2.bp.blogspot.com/-lw9LE5Hhs9k/W0HBwxjSg2I/AAAAAAAAAlc/jaH1cX8iApELTqiwuRaKZAVKg8Dp9lnjACEwYBhgL/s1600/sql7.png)

- Now, we have two main options to create “contact” table in Azure Database.

## Option 1

We will go to Query Editor Option from left navigation. It will ask for SQL Server Login.

![SQLDeployment7](https://2.bp.blogspot.com/-Lma6RdJ9Mns/W0HBxIHj8wI/AAAAAAAAAlg/qSPhoEQu6-ITUqVg22SFJLUlsVw_zvjNwCEwYBhgL/s1600/sql8.png)

We need to run following script to create contact table in SQL Database.
```sql
SET ANSI_NULLS ON    
GO      
SET QUOTED_IDENTIFIER ON    
GO      
CREATE TABLE [dbo].[Contact](    
   [id] [bigint] IDENTITY(1,1) NOT NULL,      
   [birth] [datetime2](7) NULL,      
   [email] [nvarchar](max) NULL,      
   [gender] [tinyint] NOT NULL,      
   [message] [nvarchar](max) NULL,      
   [name] [nvarchar](max) NULL,      
   [techno] [nvarchar](max) NULL,      
   CONSTRAINT [PK_Contact] PRIMARY KEY CLUSTERED    
(      
   [id] ASC      
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON)      
)      
GO   
```

## Option 2
We will replace Azure database connection string in app setting of contact application code. Then, apply Code first migration to update table in Azure database on server using this command in VS code terminal:

![SqlDeployment](https://2.bp.blogspot.com/-hjA86oz8tuI/W0HBwzmd28I/AAAAAAAAAlk/9O9uO6qHx8ErZQg4M-y2W-WfNFaiE6QgQCEwYBhgL/s1600/sql9.png)

## Create Azure Web Apps for ASP.NET Core
Here we go to App Service from left navigation in Azure Portal. These will vshow arious web apps templates for the deployment of specifying web apps.

![SqlDeployment](https://2.bp.blogspot.com/-3HcSjGnO4kM/W0HBrjPKmYI/AAAAAAAAAlg/ro1uDhMLPiQWsN9Aq3Nv5uuY5N2NqYb-gCEwYBhgL/s1600/sql10.png)

In this case, we select ASP.NET Core. It will open Web Apps Popup to configure web apps information and create Web Apps for our contact application.

- Now, we will open Visual Studio code.
- Go to the Extensions and search Azure extension.
- Select Azure Extension Pack and install it.

![SqlDeployment](https://4.bp.blogspot.com/-AQit7vUkTpg/W0HBr0sDLFI/AAAAAAAAAlQ/dY9Hkaq7UvAW0eYLlo7VflaO-NJnJ-J7ACEwYBhgL/s1600/sql12.png)

- We need to reopen VS code after installation of Azure Extension Pack. Then we can see Azure icon available in the left navigation of VS code as shown in the below screenshot:

![SqlDeployment](https://3.bp.blogspot.com/-EqpgaU-r9oE/W0HBsSntQvI/AAAAAAAAAlU/RLBDnwaHd94dxUa0aVEdv9fZ7cxMNSA1ACEwYBhgL/s1600/sql13.png)

- Click on plus button in App Service header to configure Azure web apps with contact application.
- It will ask for you to sign in to Azure. Then we can see the popup for device authentication in right-bottom corner of VS code. Click on “Copy & Open” button.

![SqlDeployment](https://3.bp.blogspot.com/-qrtDvmYVQGY/W0HBsarNPmI/AAAAAAAAAlU/gfnZeQbSB-cZQmdMt1goNXADLnb0AI3dQCEwYBhgL/s1600/sql14.png)
![AzureDeployment](https://4.bp.blogspot.com/-UKY23MhIv7U/W0HBs9VHdYI/AAAAAAAAAlY/AljeEwCmpiM9R6SQtUYZk3dOjnX9TXxGgCEwYBhgL/s1600/sql15.png)

- Enter past code and click on continue button.
- Now, we can see Contact Application Azure web apps under App Service in VS Code.
![AzureDeployment](https://4.bp.blogspot.com/-FlIRwnu-7oQ/W0HBtU8metI/AAAAAAAAAlM/BniingWvNdoJ4QpVqL9798zq3fp_akh6wCEwYBhgL/s1600/sql17.png)
- Publish ‘Contact-App’ application code though VS code terminal and note the publish folder location.
![AzureDeployment](https://4.bp.blogspot.com/-AfALXy_47Zk/W0HBt9MCZqI/AAAAAAAAAlQ/q3yZQNZ3ChkDuI984jI60iVqN_ZP9Hf_gCEwYBhgL/s1600/sql18.png)
- Right click on “Contact Application” and go to deploy web app option.
- It will ask folder location for deployment. We need to select publish folder location.
![AzureDeployment](https://3.bp.blogspot.com/-nxBnGcEs6Gc/W0HBu6W6WiI/AAAAAAAAAlg/o5b0bJZa3lo7ROnEVX7l2bm-5uiFinXywCEwYBhgL/s1600/sql20.png)
- It will start deployment. We can see deployment log in VS code output window. After successfully completing deployment it will show completion message & Web App URL.
![AzureDeployment](https://3.bp.blogspot.com/-diKdWTx9wZU/W0HBvZTvb3I/AAAAAAAAAlQ/URlLUCT0HvY5Q-oOt0obQH0lpM_VwWiPACEwYBhgL/s1600/sql21.png)

## Conclusion
In the article, we have shown detailed steps of deploying Contact Application using Visual Studio Code to Azure web apps and Azure Database. Please share your feedback on this. 
