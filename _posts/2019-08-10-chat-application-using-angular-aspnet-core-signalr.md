---
title: "Chat Application using Angular, ASP.Net Core, SignalR"
author: "Jayesh Agrawal"
date: 2019-08-10 20:55:00 +0530
categories: [dotnet, angular, nodejs]
tags: [chatapp, aspnetcore, singalr, angular, api]
seo:
  date_modified: 2021-02-15 01:55:41 +0530
---

In this article, we are going to create a simple chat application using Angular 8, Asp.net Core 2.2.0, Signal R 1.1.0 as shown below:
![Chat Application](https://1.bp.blogspot.com/-M4OAb3F5wMs/XVQ-oomPpLI/AAAAAAAABBg/AAJSzvM88wEC9MsJnRHpDz-JQ4q4SUvawCLcBGAs/s640/screenshot.gif)

### We install above all prerequisite following in our development machine:
## Prerequisite
- .Net Core SDK 2.2.0: download from [here](https://dotnet.microsoft.com/download).
- Nodejs 10.15.3: download from [here](https://nodejs.org/en/).
- Angular CLI 8.0: Install angular command package by executing this command "npm install -g @angular/cli@8.0.0".

### Setup Asp.net Core with Signal R Project: 
open a command prompt and enter create asp.net core web project as below "dotnet new web".

### Install the following NuGet package in our project:
- Microsoft.AspNetCore.SignalR
- Microsoft.AspNetCore.SpaServices.Extensions

### Call SignalR in the “Startup.ConfigureServices” method to add SignalR services:
- services.AddSignalR();  

## Create MessageHub Class:
SignalR makes real-time client-to-server and server-to-client communications possible and it will call methods to connect clients from a server using SignalR Hub API. You can find more detail about SignalR from here. Now we perform the following steps to create MessageHub class that is extending SignalR Hub API features.

## Create Hubs Folder in the project
- Declare MessageHub class that inherits from SignalR Hub Add NewMessage public method to get a new message from a connected client. Then, Send an async message to all connected client:

```cs
using ChatApp.Models;  
using Microsoft.AspNetCore.SignalR;  
using System.Threading.Tasks;  
  
namespace ChatApp.Hubs  
{  
    public class MessageHub : Hub  
    {  
        public async Task NewMessage(Message msg)  
        {  
            await Clients.All.SendAsync("MessageReceived", msg);  
        }  
    }  
}  
```
- Then, Add route to handle a request in Startup.ConfigureService method for MessageHub.

```cs
app.UseSignalR(options =>  
{  
      options.MapHub<MessageHub>("/MessageHub");  
 }); 
```  

- Add a Message Class

```cs
public class Message  
   {  
       public string clientuniqueid { get; set; }  
       public string type { get; set; }  
       public string message { get; set; }  
       public DateTime date { get; set; }  
   } 
``` 
## Add Angular 8 into the project

We scaffold Angular into our project, for this, we execute "ng new ClientApp --skip-install" on visual studio code terminal and Here, --skip-install option is used to skip installation of the npm packages.

Now we enter "npm install" command to install all angular npm packages in the terminal. And then, update SPA static file service configuration to angular output files folder location. for this, we add below code to "Startup.Configure" method:

```ts
app.UseSpa(spa =>  
            {  
                // To learn more about options for serving an Angular SPA from ASP.NET Core,  
                // see https://go.microsoft.com/fwlink/?linkid=864501  
  
                spa.Options.SourcePath = "ClientApp";  
  
                if (env.IsDevelopment())  
                {  
                    //spa.UseProxyToSpaDevelopmentServer("http://localhost:4200");  
                    spa.UseAngularCliServer(npmScript: "start");  
                }  
            });  
```
- Here, UseAngularCliServer will pass the request through to an instance of Angular CLI server which keeps up-to-date CLI-built resources without having to run the Angular CLI server manually.
Now add SignalR client library to connect MessageHub from the Angular as below screenshot:

- Create Chat Service to connect SignalR:
Create chat.service.ts class to establish a connection with Message Hub and to publish & receive chat messages.
Then, we have added chat service as a provider in Ng Modules.

```ts
import { EventEmitter, Injectable } from '@angular/core';  
import { HubConnection, HubConnectionBuilder } from '@aspnet/signalr';  
import { Message } from '../models/message';  
  
@Injectable()  
export class ChatService {  
  messageReceived = new EventEmitter<Message>();  
  connectionEstablished = new EventEmitter<Boolean>();  
  
  private connectionIsEstablished = false;  
  private _hubConnection: HubConnection;  
  
  constructor() {  
    this.createConnection();  
    this.registerOnServerEvents();  
    this.startConnection();  
  }  
  
  sendMessage(message: Message) {  
    this._hubConnection.invoke('NewMessage', message);  
  }  
  
  private createConnection() {  
    this._hubConnection = new HubConnectionBuilder()  
      .withUrl(window.location.href + 'MessageHub')  
      .build();  
  }  
  
  private startConnection(): void {  
    this._hubConnection  
      .start()  
      .then(() => {  
        this.connectionIsEstablished = true;  
        console.log('Hub connection started');  
        this.connectionEstablished.emit(true);  
      })  
      .catch(err => {  
        console.log('Error while establishing connection, retrying...');  
        setTimeout(function () { this.startConnection(); }, 5000);  
      });  
  }  
  
  private registerOnServerEvents(): void {  
    this._hubConnection.on('MessageReceived', (data: any) => {  
      this.messageReceived.emit(data);  
    });  
  }  
}
```
## Create a Chat App Component(app.component.html)
```html
<div class="container">  
  <h3 class=" text-center chat_header">Chat Application</h3>  
  <div class="messaging">  
    <div class="inbox_msg">  
      <div class="mesgs">  

        <div class="msg_history">  
          <div *ngFor="let msg of messages">  
          <div class="incoming_msg" *ngIf="msg.type == 'received'">  
            <div class="incoming_msg_img"> </div>  
            <div class="received_msg">  
              <div class="received_withd_msg">  
                <p>  
                 {{msg.message}}   
                </p>  
                <span class="time_date"> {{msg.date | date:'medium'}} </span>  
              </div>  
            </div>  
          </div>  
          <div class="outgoing_msg" *ngIf="msg.type == 'sent'">  
            <div class="sent_msg">  
              <p>  
                  {{msg.message}}   
              </p>  
              <span class="time_date"> {{msg.date | date:'medium'}}</span>  
            </div>  
          </div>  
        </div>  
        </div>  
        <div class="type_msg">  
          <div class="input_msg_write">  
            <input type="text" class="write_msg" [value]="txtMessage"  
            (input)="txtMessage=$event.target.value" (keydown.enter)="sendMessage()" placeholder="Type a message" />  
            <button class="msg_send_btn" type="button"  (click)="sendMessage()"><i class="fa fa-paper-plane-o" aria-hidden="true"></i></button>  
          </div>  
        </div>  
      </div>  
    </div>  
  
  </div>  
</div>  
```

app.component.ts:

```ts
import { Component, NgZone } from '@angular/core';  
import { Message } from '../models/Message';  
import { ChatService } from '../services/chat.service';  
  
@Component({  
  selector: 'app-root',  
  templateUrl: './app.component.html',  
  styleUrls: ['./app.component.css']  
})  
export class AppComponent {  
  
  title = 'ClientApp';  
  txtMessage: string = '';  
  uniqueID: string = new Date().getTime().toString();  
  messages = new Array<Message>();  
  message = new Message();  
  constructor(  
    private chatService: ChatService,  
    private _ngZone: NgZone  
  ) {  
    this.subscribeToEvents();  
  }  
  sendMessage(): void {  
    if (this.txtMessage) {  
      this.message = new Message();  
      this.message.clientuniqueid = this.uniqueID;  
      this.message.type = "sent";  
      this.message.message = this.txtMessage;  
      this.message.date = new Date();  
      this.messages.push(this.message);  
      this.chatService.sendMessage(this.message);  
      this.txtMessage = '';  
    }  
  }  
  private subscribeToEvents(): void {  
  
    this.chatService.messageReceived.subscribe((message: Message) => {  
      this._ngZone.run(() => {  
        if (message.clientuniqueid !== this.uniqueID) {  
          message.type = "received";  
          this.messages.push(message);  
        }  
      });  
    });  
  }  
}  
```

Now our chat application is ready. let's run following command in terminal and test apps: "dotnet run".

## Summary:
In this article, we have learned about how we can create sample chat app using angular 8, asp.net core with Signal R. Please find entire source code on GitHub:[https://github.com/AngularExamplesHub/angular-chat-app](https://github.com/AngularExamplesHub/angular-chat-app). 
