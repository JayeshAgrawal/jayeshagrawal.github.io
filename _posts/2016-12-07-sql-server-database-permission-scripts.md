---
title: "SQL Server Database Permission Scripts"
author: "Jayesh Agrawal"
date: 2016-12-07 14:35:00 +0530
categories: [database]
tags: [sqlserver]
seo:
  date_modified: 2021-01-10 01:55:41 +0530
---

## SQL Server Database Permission Scripts

### 1 - Script to get list of all database name and its user in SQL Server

```sql
USE MASTER  
GO  
  
SELECT SL.dbname AS 'Database Name',SL.name AS 'User Name',SP.type_desc AS 'Login Type', 
SL.denylogin, SL.hasaccess, SL.isntname, SL.isntname, SL.isntgroup, SL.isntuser, SL.sysadmin,  
SL.securityadmin, SL.serveradmin, SL.setupadmin, SL.processadmin, SL.diskadmin, SL.diskadmin,  
SL.dbcreator, SL.bulkadmin FROM sys.server_principals AS SP  
INNER JOIN sys.syslogins AS SL ON SP.SID = SL.SID  
```

### 2 - Script to get list of all database users and their roles in SQL Server

```sql
Use Master  
GO  
  
exec sp_msForEachDb ' use [?]  
select db_name() as [Databast Name], r.[name] as [Role], p.[name] as [Member Name],  
p.[default_schema_name] as [Schema],p.[principal_id] as [Principal Id]  
from  
sys.database_role_members m  
join  
sys.database_principals r on m.role_principal_id = r.principal_id  
join  
sys.database_principals p on m.member_principal_id = p.principal_id'  
```

### 3 - Script to get list of users and their permission with all stored procedure in SQL Server database

```sql
use <<databasename>>  
GO  
  
select sys.schemas.name 'Schema'  
, sys.objects.name 'Stored Procedure'  
, sys.database_principals.name username  
, sys.database_permissions.type permissions_type  
, sys.database_permissions.permission_name  
, sys.database_permissions.state permission_state  
, sys.database_permissions.state_desc  
, state_desc + ' ' + permission_name + ' on ['+ sys.schemas.name + '].[' + sys.objects.name + '] to [' + sys.database_principals.name + ']' COLLATE LATIN1_General_CI_AS  
from sys.database_permissions  
join sys.objects on sys.database_permissions.major_id = sys.objects.object_id  
join sys.schemas on sys.objects.schema_id = sys.schemas.schema_id  
join sys.database_principals on sys.database_permissions.grantee_principal_id = sys.database_principals.principal_id  
Where sys.objects.type IN ('P')  
order by 1, 2, 3, 5  
```

**Tested in SQL server 2008 R2, 2008, 2012.**