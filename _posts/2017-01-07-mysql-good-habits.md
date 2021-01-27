---
title: "MySQL Good Habits"
author: "Jayesh Agrawal"
date: 2017-01-07 20:55:00 +0530
categories: [MySQL]
tags: [Database, StoredProcedure, GoodHabits]
---

In this article, we are going to learn few good habits that we can consider important aspect while work with MySQL to improve performance & troubleshoot as following below:

## 1 - Do not use stored procedure & function parameters name same as WHERE clause field name
- It will be responding with all record of query because MySQL interprets field value as parameter value similar like 1=1.
