---
title: "MySQL Data Access API Development Using Express.JS, Node.JS"
author: "Jayesh Agrawal"
date: 2017-07-23 20:55:00 +0530
categories: [nodejs]
tags: [expressjs, api, mysql]
seo:
  date_modified: 2021-02-07 01:55:41 +0530
---

In this article, we are looking into the introduction of Node.JS and Express.JS and then, I am going to create a simple MySQL data access API using Express.js, NodeJS.

As we all know, Node.JS is the most trending open source technology that’s directly or indirectly partitioning into advanced web development, and let’s have lookup what is Node.JS?

## Node.JS Definition
Node.JS is a platform that provides environment (outside the web browser) to execute JavaScript and which is built on Chrome's V8 JavaScript engine developed by Ryan Dahl.

## Why Node.JS?
NodeJs processing model depends on a single thread with event looping.

When clients send HTTP requests to NodeJs server, the Event loop is woken up by OS. It will pass the request and response objects as JavaScript closures to worker functions with callbacks.

For long-running jobs like I/O operations running on non-blocking worker threads, when the job is complete, responses are sent to the main thread via a callback. Event loop returns result to the client.

NodeJs has following features:
- Consistently asynchronous
- Event Driven, Lightweight
- Same language on client\server
- NPM, Yarn (built-in support for package management)
- Built Http/https server, sockets
- Develop Real-time, Web, Mobile applications

## Node.JS is not best platform for
- Heavy Server-Side Computation/Processing
 
## Node.JS System Module
- events - To handle events
- fs - To handle the file system
- http - To make Node.js act as an HTTP server
- https - To make Node.js act as an HTTPS server.
- net - To create servers and clients
- os - Provides information about the operation system
- path - To handle file paths

## Node.JS Installation
- Download the Windows installer from the node.js web site.
- Run the installer (the .msi file you downloaded in the previous step.)
- Follow the prompts in the installer (Accept the license agreement, click the NEXT button a bunch of times and accept the default installation settings).
- To see if Node is installed, open the Windows Command Prompt, PowerShell or a similar command line tool, and type node -v.
- To see if NPM is installed, type npm –v. (NPM is a package manager for Node.js packages or modules)

## Express.JS
- The express framework provides an abstraction layer above HTTP module to make handling web traffic (Request/ Response) with developing middleware and APIs. There are also tons of middleware available to complete the common task for express frameworks, such as- CORS, XSRF, POST parsing, Cookies etc.
- JS framework is built on Node.JS HTTP module.
- We can install Express.Js global by just type npm install -g express in command prompt.
- Now, we have our development environment ready with NodeJS, Express.js installation. And, I am going to demonstrate how we can create MySQL data access API using Node.JS and Express.js.

Let’s start with creating a folder as "sample-api".

- Open a folder in command prompt and type "npm init" to initiate the project.

![create nodejs project](https://2.bp.blogspot.com/-xCjrrG-mDG0/W0DUzUW3D1I/AAAAAAAAAf8/ybpGwwAauxAKJ00w1VJ4wWoJMuu561bWQCEwYBhgL/s640/image001.jpg)

- Enter details like project name, version, description etc. then type 'Yes'.
- It will create a JSON file inside sample-api folder which contains the information regarding the project and NPM package that are needed into this project. We called dependencies.

![create package.json](https://3.bp.blogspot.com/-YwyHQBm1sXs/W0DUzYKsFFI/AAAAAAAAAgE/UFFUfudcMyMFbWNTw0-7_CpWDHlspsMmgCEwYBhgL/s640/image002.jpg)

- Now, we are creating a MySQL table for users and their transaction details, as below.

```sql
--  
-- Table structure for table `users`  
--  
DROP TABLE IF EXISTS `users`;  
CREATE TABLE `users` (  
  `UserID` int(11) NOT NULL AUTO_INCREMENT,  
  `Name` varchar(45) DEFAULT NULL,  
  PRIMARY KEY (`UserID`)  
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;  
  
--  
-- Table structure for table `transactions`  
--  
DROP TABLE IF EXISTS `transactions`;  
CREATE TABLE `transactions` (  
  `TransactionId` int(11) NOT NULL AUTO_INCREMENT,  
  `UserId` int(11) DEFAULT NULL,  
  `TransactionAmount` decimal(10,2) DEFAULT NULL,  
  `Balance` decimal(10,2) DEFAULT NULL,  
  `TransactionDate` date DEFAULT NULL,  
  PRIMARY KEY (`TransactionId`)  
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8;  
```

- Then, we are installing NPM packages for MySQL to make MySQL connection and fetch insert, update records, express to build HTTP server, maintaining server route, body-parser to parsing body into JSON format by entering this command.

![install mysql package](https://1.bp.blogspot.com/-JTILqFBnx0Q/W0DUzi-aiTI/AAAAAAAAAgA/_rxqiRkVO0QGPfNBgRyF6Z-fVRkdLFauQCEwYBhgL/s640/image003.png)

- We can see that the packages are added to the node_modules folder in our project folder (sample-api). And, I have created this type of folder structure for our project. There are no specific rules regarding project structure, but we are maintaining everything with the modular design to keep it separate and well-maintained.

![node modules folder](https://4.bp.blogspot.com/-13MI-9aItXc/W0DU0BvxetI/AAAAAAAAAgE/NXuWXldeNdQ__E3OHhlOfGXRs_1ivsIqgCEwYBhgL/s400/image004.png)

- Now, we are creating MySQLConnect module for establishing the connection with MySQL and executing the MySQL queries.
- For that, we have created ‘connection/MySQLConnect.JS’ file.

```js
// establish Mysql Connection  
var mysql = require('mysql');  
  
function MySQLConnect() {  
  
  this.pool = null;  
    
  // Init MySql Connection Pool  
  this.init = function() {  
    this.pool = mysql.createPool({  
      connectionLimit: 10,  
      host     : 'localhost',  
      user     : 'root',  
      password : 'admin@123',  
      database: 'sample-db'  
    });  
  };  
  
  // acquire connection and execute query on callbacks  
  this.acquire = function(callback) {  
  
    this.pool.getConnection(function(err, connection) {  
      callback(err, connection);  
    });  
  
  };  
  
}  
  
module.exports = new MySQLConnect();
```
- Then, I have created ‘data_access/transaction.js’ file for acquiring the MySQL connection and return the response with data.

```js
//methods for fetching mysql data  
var connection = require('../connection/MySQLConnect');  
  
function Transaction() {  
  
    // get all users data   
    this.getAllUsers = function (res) {  
        // initialize database connection  
        connection.init();  
        // calling acquire methods and passing callback method that will be execute query  
        // return response to server   
        connection.acquire(function (err, con) {  
            con.query('SELECT DISTINCT * FROM users', function (err, result) {  
                con.release();  
                res.send(result);  
            });  
        });  
    };  
  
    this.getTransactionById = function (id, res) {  
        // initialize database connection  
        connection.init();  
        // get id as parameter to passing into query and return filter data  
        connection.acquire(function (err, con) {  
            var query = 'SELECT date_format(t.TransactionDate,\'%d-%b-%Y\') as date, ' +  
                'CASE WHEN t.TransactionAmount >= 0 THEN t.TransactionAmount ' +  
                'ELSE 0 END AS Credit, CASE WHEN t.TransactionAmount < 0 THEN ' +  
                't.TransactionAmount ELSE 0 END AS Debit, t.Balance FROM ' +  
                'transactions t INNER JOIN users u ON t.UserId=u.UserID WHERE t.UserId = ?;';  
            con.query(query, id, function (err, result) {  
                    con.release();  
                    res.send(result);  
                });  
        });  
    };  
  
}  
  
module.exports = new Transaction();  
```
- Now, I have developed routes for returning the data based on each request. So, I have added ‘route/route.js’ file.

```js
//custom route for fetching data  
var transactions = require('../data_access/transaction');  
  
module.exports = {  
    //set up route configuration that will be handle by express server  
    configure: function (app) {  
  
        // adding route for users, here app is express instance which provide use  
        // get method for handling get request from http server.   
        app.get('/api/users', function (req, res) {  
            transactions.getAllUsers(res);  
        });  
  
        // here we gets id from request and passing to it transaction method.  
        app.get('/api/transactions/:id/', function (req, res) {  
            transactions.getTransactionById(req.params.id, res);  
        });  
  
    }  
};
```
- Now, we are writing the code for setting up the server using Express and here, we initialize the database connection and set route configuration with application.

```js
/**  
 * Creating server using express.js 
 * http://localhost:8000/api/users 
 * http://localhost:8000/api/transactions/1 
*/  
var express = require('express');  
var bodyparser = require('body-parser');  
  
var routes = require('./routes/route');  
  
// creating server instance  
var app = express();  
  
// for posting nested object if we have set extended true  
app.use(bodyparser.urlencoded({ extended : true}));  
  
// parsing JSON  
app.use(bodyparser.json());  
  
//set application route with server instance  
routes.configure(app);  
  
// listening application on port 8000  
var server = app.listen(8000, function(){  
    console.log('Server Listening on port ' + server.address().port);  
});
```

- Our API is created; we can verify the API by running the server on localhost. To run our Server, we enter ‘npm start’ command in console. The npm start command is run on the node with starting page which we have set ‘main’ (server.js) key in package.json file. The server is ready for listening on port 8000.

![Run API](https://4.bp.blogspot.com/-OcAJmebnueQ/W0DU0o8ts3I/AAAAAAAAAgM/2N4ezcPymmQJMDYATvaJLXSI2As5RcThACEwYBhgL/s640/image005.jpg)

We are verifying our API call by requesting it from the below URL.
- http://localhost:8000/api/users

![User API](https://3.bp.blogspot.com/-M_a-OpYZUbM/W0DU0rR33XI/AAAAAAAAAgI/9DDPQ2lkKQQOsdRPzzjmvuPdj1u85h5ywCEwYBhgL/s320/image006.png)

## Conclusion
In this article, we have looked into introduction of Node.JS and Express.JS, and we have created simple MySQL data access API using Express.js.
You can find the entire code on [GitHub](https://github.com/JayeshAgrawal/sample-api-expressjs).