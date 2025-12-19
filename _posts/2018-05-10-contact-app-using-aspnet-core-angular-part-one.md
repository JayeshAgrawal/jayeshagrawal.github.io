---
title: "Contact Application Using ASP.NET Core Angular 6.0 - Part One"
author: "Jayesh Agrawal"
date: 2018-05-10 20:55:00 +0530
categories: [DotNet]
tags: [ContactApplication, Angular, ApiDevelopment]
seo:
  date_modified: 2021-02-20 01:55:41 +0530
---

We will develop a contact application using ASP.NET Core Web API, Angular 6.0, and Angular Material UI as shown in the below screenshots. 

![contactapp](https://4.bp.blogspot.com/-eteH6ZphAVg/W0GzsIGdefI/AAAAAAAAAg8/URKnk8KI9-Aw7tV283rqEaYtVH3EHz8TwCEwYBhgL/s1600/1list.png)

We will use Visual Studio code (VS code) for the development editor, I have divided contact application development into two articles:

- [Contact Application using ASP.NET Core Web API, Angular 6.0, and Visual Studio Code Part One](https://crack-codes.blogspot.com/2018/05/contact-app-using-aspnet-core-angular-part-one.html)
In the article, we will set up ASP.NET Core Web API project, and develop the Web API for contact CRUD operations.
- [Contact Application using ASP.NET Core Web API, Angular 6.0, and Visual Studio Code Part Two](https://crack-codes.blogspot.com/2018/06/contact-app-using-aspnet-core-angular-part-two.html)
In the article, we will setup Angular 6 within ASP.NET Core Web API Project, and develop the contact form & list component using Angular Material UI that will consume Web API which we have created in Part One.

So, Let's start with this article, first, we will see how we can create ASP.Net Core Web API Project.

We have the following prerequisites:
- Visual Studio Code (VS code) editor: Please see this [article](https://code.visualstudio.com/docs/setup/windows) to see how we can setup Visual Studio code.
- .Net Core: I have used .Net Core version 2.0.3 for this contact application. Please see this [link](https://www.microsoft.com/net/learn/get-started/windows) from where we can download .Net Core SDK.

When .Net Core SDK is installed, we can check version from the console as shown in the screenshot:

![contactapp](https://1.bp.blogspot.com/-he7MP_Kg-U4/W0GzsrnxdGI/AAAAAAAAAh4/B5NRVsfuukoF3FJgpZJcVkufF5fX4eQfgCEwYBhgL/s1600/3console.png)

After this, we require VS code extensions that adds languages, debuggers, and tools to support our development workflow. Please see this [article](https://code.visualstudio.com/docs/editor/extension-gallery) to see how we can manage extension in VS code.

We have added these extensions in VS code:
- [ASP.NET core VS Code Extension Pack](https://marketplace.visualstudio.com/items?itemName=temilaj.asp-net-core-vs-code-extension-pack)
- [.NET Core Extension Pack](https://marketplace.visualstudio.com/items?itemName=doggy8088.netcore-extension-pack)
- [Angular v6 Snippets](https://marketplace.visualstudio.com/items?itemName=johnpapa.Angular2)
- [Angular 6 Snippets - TypeScript, Html, Angular Material, ngRx, RxJS & Flex Layout](https://marketplace.visualstudio.com/items?itemName=Mikael.Angular-BeastCode)

Now, we are ready for setup ASP.Net Core Web API project.
- Open Command Prompt and enter the following commands:
```bat
mkdir contact-app  
cd contact-app  
dotnet new webapi 
```
- Web API project has been created in the contact-app folder. Then go to the VS code, and open contact-app folder.
- We can see one notification in  the VS code's right end corner.
![notification](https://1.bp.blogspot.com/-9PZE68Zl-z8/W0GztbZkU-I/AAAAAAAAAh0/AZeCuKCQbGwxp0zgRWgw0aXwv31mzH5QgCEwYBhgL/s320/6nofication.png)
- Build and debugger tool has been set up in VS Code.
![debugger](https://2.bp.blogspot.com/-tXAnjmKi0-g/W0GzuSn7UyI/AAAAAAAAAh8/JpM2HGA6M_Q1xO2lTAHIZYLsrp_r-qMPACEwYBhgL/s640/7debug.png)

- Here now we are going to create the following API for the contact application:

| API                                   | Description                       | Request body  | Response body         |
|---------------------------------------|-----------------------------------|---------------|-----------------------|
| GET - /api/contact/getAllContact      | Get all contacts                  | None          | Array of to-do items  |
| GET /api/contact/getContact           | Get contact Item                  | Contact Item  | Contact Item          |
| POST /api/contact/addContact          | Add a new contact                 | Contact Item  | Contact Item          |
| PUT /api/contact/updateContact{id}    | Update an existing contact item   | Contact item  | None                  |
| DELETE /api/contact/deleteContact{id} | Delete an contact item            | None          | None                  |

## Create contact model
In this, we are creating a contact class that has getter-setter property as shown in snippet:
```cs
public class Contact {  
    public long ? id {  
        get;  
        set;  
    }  
    public string name {  
        get;  
        set;  
    }  
    public string email {  
        get;  
        set;  
    }  
    public byte gender {  
        get;  
        set;  
    }  
    public DateTime ? birth {  
        get;  
        set;  
    }  
    public string techno {  
        get;  
        set;  
    }  
    public string message {  
        get;  
        set;  
    }  
}  
```
## Create the database context for contact model
Here we are using a code-first approach to develop the contact application. So we have created ‘ContactAppContext’ class to communicate contact model to contact table.

```cs
public class ContactAppContext: DbContext {  
    public ContactAppContext(DbContextOptions < ContactAppContext > options): base(options) {}  
    public DbSet < Contact > Contact {  
        get;  
        set;  
    }  
}  
```
## Register database context in startup:
To register database context we have followed these steps:
- Go to "startup.cs" and update this code for SQL server connection in "ConfigureServices" method:
```cs
services.AddDbContext<ContactAppContext>(options =>  
options.UseSqlServer(Configuration.GetConnectionString("ContactDb"))); 
```
- Then added "Microsoft.EntityFrameworkCore.SqlServer" Package in this project to support SQL server connection.

![contactapp](https://4.bp.blogspot.com/-9XlPrVUVv7Q/W0Gzt6pG1nI/AAAAAAAAAh4/kb-21KWYeE8L3XZNlb89Uab2e5G3TbhkwCEwYBhgL/s640/8package.png)

Here we need to configure connection string in appsettings.json file as shown in the snippet.
```json
"ConnectionStrings": {  
   "ContactDb": "Data Source=JAYESH-PC\\SQLEXPRESS;Initial Catalog=ContactDb;Integrated Security=True"  
}  
```
## Initiate Database Migration
To create Initial migration script as per model, we enter "dotnet ef migrations add InitialCreate" command in VS code terminal and we get this issue:

![databasemigration](https://3.bp.blogspot.com/-AYd2g4KuoiE/W0GzqT9bOSI/AAAAAAAAAhk/u9k_v2qg52IIb_aWYlL6B8LB1TIRfkXVwCEwYBhgL/s640/10migration.png)

To solve the issue we have to add this .Net Core CLI tool package in "contact-app.csproj" file and then enter "dotnet restore" in VS code terminal to restore these packages to create initial migration script:
- Microsoft.EntityFrameworkCore.Tools
- Microsoft.EntityFrameworkCore.Tools.DotNet
![datamigration](https://4.bp.blogspot.com/-prsz8zfSpeo/W0GzuLpRHxI/AAAAAAAAAh4/3z5LE0ks89wE599F7GqagXXeXFlxOw8TgCEwYBhgL/s640/9package.png)

Now we enter "dotnet of migrations add InitialCreate" in VS code terminal and we can see that it will create migration folder and script "InitialCreate" in the project as shown in the screenshot:
![datamigration](https://2.bp.blogspot.com/-qd-pM-bZDr0/W0GzqZSdHBI/AAAAAAAAAh0/jbeQX3QHO50X2r5nEoRnuIW_5hPDPOIqwCEwYBhgL/s320/11migration.png)

Now update the database by entering "dotnet ef database update" command in VS code terminal and it will create a database in SQL server:
![datamigration](https://2.bp.blogspot.com/-l4Gy9N7dvQs/W0GzqrmdPII/AAAAAAAAAhs/CrWGBVdcCWY2md8YDDk78v9CBDoFUKYqgCEwYBhgL/s320/12migration.png
)
Please see this [article](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/) regarding .Net Core Migration.
## Create contact API controller
After the database is ready, we have created API controller for CRUD operations as in the below snippet:
```cs
// set route attribute to make request as 'api/contact'  
[Route("api/[controller]")]  
public class ContactController: Controller {  
    private readonly ContactAppContext _context;  
    // initiate database context  
    public ContactController(ContactAppContext context) {  
            _context = context;  
        }  
        [HttpGet]  
        [Route("getAllContact")]  
    public IEnumerable < Contact > GetAll() {  
            // fetch all contact records  
            return _context.Contact.ToList();  
        }  
        [HttpGet("{id}")]  
        [Route("getContact")]  
    public IActionResult GetById(long id) {  
            // filter contact records by contact id  
            var item = _context.Contact.FirstOrDefault(t => t.id == id);  
            if (item == null) {  
                return NotFound();  
            }  
            return new ObjectResult(item);  
        }  
        [HttpPost]  
        [Route("addContact")]  
    public IActionResult Create([FromBody] Contact item) {  
            // set bad request if contact data is not provided in body  
            if (item == null) {  
                return BadRequest();  
            }  
            _context.Contact.Add(new Contact {  
                name = item.name,  
                    email = item.email,  
                    gender = item.gender,  
                    birth = item.birth,  
                    techno = item.techno,  
                    message = item.message  
            });  
            _context.SaveChanges();  
            return Ok(new {  
                message = "Contact is added successfully."  
            });  
        }  
        [HttpPut("{id}")]  
        [Route("updateContact")]  
    public IActionResult Update(long id, [FromBody] Contact item) {  
            // set bad request if contact data is not provided in body  
            if (item == null || id == 0) {  
                return BadRequest();  
            }  
            var contact = _context.Contact.FirstOrDefault(t => t.id == id);  
            if (contact == null) {  
                return NotFound();  
            }  
            contact.name = item.name;  
            contact.email = item.email;  
            contact.gender = item.gender;  
            contact.birth = item.birth;  
            contact.techno = item.techno;  
            contact.message = item.message;  
            _context.Contact.Update(contact);  
            _context.SaveChanges();  
            return Ok(new {  
                message = "Contact is updated successfully."  
            });  
        }  
        [HttpDelete("{id}")]  
        [Route("deleteContact")]  
    public IActionResult Delete(long id) {  
        var contact = _context.Contact.FirstOrDefault(t => t.id == id);  
        if (contact == null) {  
            return NotFound();  
        }  
        _context.Contact.Remove(contact);  
        _context.SaveChanges();  
        return Ok(new {  
            message = "Contact is deleted successfully."  
        });  
    }  
}  
}  
```
## Build and run contact application project
To build contact application project we have entered "dotnet build" command VS code terminal:
![datamigration](https://3.bp.blogspot.com/-EE8N1NMUqNE/W0GzqwGLnrI/AAAAAAAAAhs/XZJbQC8oGDQJSmK1Bs6u3AbOTwRWwTN4gCEwYBhgL/s400/13build.png)
- Setup development variable and URL for project in terminal:
```bat
set ASPNETCORE_ENVIRONMENT=Development
set ASPNETCORE_URLS=http://localhost:5000
```
Then, to run project we have to enter "dotnet run":
![datamigration](https://4.bp.blogspot.com/-XsSzYCkJYtA/W0GzrFpCEpI/AAAAAAAAAhs/tWm7DHOFApcqHPqhUvyPNPVzkLnl8t8wwCEwYBhgL/s400/14build.png)

Please see this [article](https://docs.microsoft.com/en-us/dotnet/core/tools/?tabs=netcore2x) for more information regarding .Net Core 2x commands.

## Test API
- To test API we have installed [ChromeRestClient](https://install.advancedrestclient.com/#/install).
- Then we have to open Rest Client, to get contact item by id we set HTTP parameter to get and request URL as shown in the below screenshot:

![RestAPI](https://4.bp.blogspot.com/-Usy5JVl9vVE/W0GzrZLXSNI/AAAAAAAAAho/ZaZVRD1W1pIitkPc2DZ4WwrcjWWBNTP8gCEwYBhgL/s640/15Rest.png)

For adding contact item, we have set the body to contact JSON object:

![RestAPI](https://3.bp.blogspot.com/-ME10psOPtHg/W0GzrnZm6sI/AAAAAAAAAh4/0EhHsTJYEs4RdSHsyGugLSIEohiFGYarwCEwYBhgL/s400/16Rest.png)

Then, we request add contact URL and set HTTP parameter to post.

![RestAPI](https://1.bp.blogspot.com/-aUbPcNU6vQg/W0GzsWNW4vI/AAAAAAAAAh8/CLvPNH_SGOUnfVS_mdrQ0_CRvOAokKvhwCEwYBhgL/s320/17Rest.png)

## Conclusion
In this article, we learned about how we can develop ASP.Net Core Web API using VS code.

In the next article, we will learn about how we can set up Angular 6 in Web API Project and develop the contact list & contact form component using Angular Material UI. Please see this [link](https://crack-codes.blogspot.com/2018/06/contact-app-using-aspnet-core-angular-part-two.html) for the Part-2 of this article.

Please see entire code in this GitHub [https://github.com/JayeshAgrawal/contact-app](https://github.com/JayeshAgrawal/contact-app).

Please share your feedback.