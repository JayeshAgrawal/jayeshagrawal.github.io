---
title: "Create SSRS Drill Down Report"
author: "Jayesh Agrawal"
date: 2017-08-25 10:20:00 +0530
categories: [Database]
tags: [SQLServer, SSRS]
seo:
  date_modified: 2021-01-10 01:55:41 +0530
---

## Create SSRS Drill Down Report

In this blog, we are going to learn about how we can create SSRS drill down report, please refer steps as following:

**To create a report server project****
  - Open SQL Server Data Tools.
  - On the File menu, point to New, and then click Project.
  - In the Installed Templates list, click Business Intelligence.
  - Click Reporting Services.
  - Click Report Server Project. If you do not see the "Business Intelligence" or "Report Server Project" options, you need to update SSDT with the Business Intelligence templates.
  - In Name, type "ProductReports".
  - Click OK to create the project.
  - The ProductReports project is displayed in Solution Explorer.

**To create a new report definition file****
In the Solution Explorer pane, right-click the Reports folder, point to Add, and click New Item. If the Solution Explorer window is not visible, from the View menu, click Solution Explorer:
  - In the Add New Item window, click Report.
  - In Name, type “OrderReport.rdl” and then click Add.
  - Report Designer opens and displays the new .rdl file in Design view.
  - To set up a connection
  - In the Report Data pane, click New and then click Data Source.
  - In Name, type “ProductsDataSource”.
  - Make sure Embedded connection is selected.
  - In Type, select Microsoft SQL Server.

In Connection string, type the following:

```xml
Data Source= devServer\\ProductReportServer;Initial Catalog=ProductReport_Db
```

 - Click Credentials in the left pane and click Use Windows Authentication (integrated security).
 - Click OK.  data source “ProductsDataSource” is added to the Report Data pane.
 
**To define a Transact-SQL query for report data**
 - In the Report Data pane, click New, and then click Dataset. The Dataset Properties dialog box opens.
 - In the Name box, type “OrderReportDataset”.
 - Click Use a dataset embedded in my report.
 - Make sure the name of your data source, “ProductsDataSource”, is in the Data source text box, and that the Query type is Text.
 - Type, or copy and paste, the following Transact-SQL query into the Query box.

```sql
SELECT product.productName, [order].orderName, CONVERT(varchar, [order].orderDate, 101) AS orderDate, [order].quatity, [order].productId FROM [order] INNER JOIN product ON [order].productId = product.productid ORDER BY CONVERT(date, [order].orderDate, 101) DESC
```

 - (Optional) Click the Query Designer button. The query is displayed in the text-based query designer. You can toggle to the graphical query designer by clicking Edit As Text. View the results of the query by clicking the run ssrs_querydesigner_run button on the query designer toolbar.
 - Click OK to exit the query designer.
 - Click OK to exit the Dataset Properties dialog box.

**To add a Table data region and fields to a report layout**
  - In the Toolbox, click Table, and then click on the design surface and drag the mouse. Report Designer draws a table data region with three columns in the center of the design surface. The Toolbox may appear as a tab on the left side of the Report Data pane. To open the Toolbox, move the pointer over the Toolbox tab. If the Toolbox is not visible, from the View menu, click Toolbox.
  - You can also add a table to the report from the design surface. Right-click the design surface, click Insert and then click Table.
  - In the Report Data pane, expand the dataset “OrderReportDataset” to display the fields.
  - Drag the Date field from the Report Data pane to the first column in the table.
  - When you drop the field into the first column, two things happen. First, the data cell will display the field name, known as the field expression, in brackets: [Order Date]. Second, a column header value is automatically added to Header row, just above the field expression. By default, the column is the name of the field. You can select the Header row text and type a new name.
  - Drag the Product Name field from the Report Data pane to the second column in the table.
  - Drag the Order Name field from the Report Data pane to the third column in the table.
  - Drag the Quantity field to the right edge of the third column until you get a vertical cursor and the mouse pointer has a plus sign [+]. When you release the mouse button, a fourth column is created for [Quantity].

**To preview a report**
  - Click the Preview tab. Report Designer runs the report and displays it in Preview view.
  - To format a date field
  - Click the Design tab.
  - Right-click the cell with the [Date] field expression and then click Text Box Properties.
  - Click Number, and then in the Category field, click Date.
  - In the Type box, select January 31, 2000.
  - Click OK.
  - Preview the report to see the change to the [Date] field and then change back to design view.

**To group data in a report**
 - Click the Design tab.
 - If you do not see the Row Groups pane , right-click the design surface and click view and then click Grouping.
 - From the Report Data pane, drag the Date field to the Row Groups pane. Place it above the row called (Details).
 - From the Report Data pane, drag the Order field to the Row Groups pane. Place it below Date and above (Details).
 - Note that the row handle now has two brackets in it, to show two groups. The table now has two Order columns, too.
 - Delete the original Date and Order columns to the right of the double line. This removes this individual record values so that only the group value is displayed. Select the column handles for the two columns, right-click and click Delete Columns.
 
**Pop-up detail report (Drill-through report)**

**Drill-Through Report**
 - A Drill-Through report is a report that a user opens by clicking a link within another report. Drill-Through reports commonly contain details about an item that is contained in an original summary report.
 - Add a new report named “ProductReport” to the solution.
 - Create a new dataset named “ProductDataset” using the “ProductsDataSource” data source created in the previous step.
 - Enter the following query as text into the query designer.

```sql
SELECT productName, productDesc, quatity  
FROM product  
WHERE (productid = @productid); 
Now have a complete drill-through target report, "ProductReport", to act as a companion to the drill-through source report, we use “OrderReport”. we modify the “OrderReport” to implement drill-through functionality to the Product Detail report.
Then click on Text Box properties,
Then go to action tab right in menu. Where we have select “Go to URL” check-box. Click on function button.
Where expression pop up is open. We have added expression for specific data check condition and JavaScript to open report url.
=IIf(CDate(Fields!orderDate.Value) >= CDate("06/02/2016"),"javascript:void(window.open(" &  
Globals!ReportServerUrl & " '/Pages/ReportViewer.aspx?/ProjectReports/ProductReport&productid=" & Fields!productId.Value & "','blank','location=no,toolbar=no,left=100,top=100,height=450,width=800'))","")  
```
 - In this @productid parameter for “ProductReport”, we have passed query-string variable.
 - Make link enabled/disabled based on specific condition 
 - Same way above we have passed condition in link format to show user enabled/disabled functionality within report.
 - Click on text-box properties, then go to font section.
 
In font section, click on function button in colour properties where we can added expression to formatting text base on specific condition. 

```sql
=IIf(CDate(Fields!orderDate.Value) >= CDate("06/02/2016"), "blue", "black")  
```

Then click on okay.

**Conclusion**
In this blog, we have learned how we can create SSRS drill down report, add link enable/disable based on specific condition in reports. 

Thanks for reading article.