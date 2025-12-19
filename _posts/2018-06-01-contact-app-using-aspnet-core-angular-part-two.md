---
title: "Contact Application Using ASP.NET Core, Angular - Part Two"
author: "Jayesh Agrawal"
date: 2018-06-01 12:55:00 +0530
categories: [Angular]
tags: [ContactApplication, AspNetCore, ApiDevelopment]
seo:
  date_modified: 2021-02-20 01:55:41 +0530
---

In this article, we are going to learn about how we can setup Angular 6.0 in ASP.Net Core Web API project and develop the contact list & form component using Angular Material UI.

In the Part 1 article, we set up ASP.NET Core Web API and developed the API for contact CRUD operation. Please go through [Contact Application using ASP.NET Core Web API, Angular 6.0, and Visual Studio Code Part 1](https://crack-codes.blogspot.com/2018/05/contact-app-using-aspnet-core-angular-part-one.html) once before proceeding with this article.

To setup Angular 6.0 in the project: first, we have to install NodeJS in the development environment. We can download NodeJS installer from [NodeJS website](https://nodejs.org/en/). and then check the version on NodeJS as below:

![contactapp](https://1.bp.blogspot.com/-lMlolGQioSM/W0G-WFHRK-I/AAAAAAAAAjY/OZ4x0uJ_2nQj907a5HwWGZIsu0Dt9S_8wCEwYBhgL/s400/0node.png)

Now, let us globally install Angular CLI package by enter "npm install –g angular/cli@6.0.0", as shown in below screenshot

![contactapp](https://1.bp.blogspot.com/-EbE1pV3HTtk/W0G-YLc-5GI/AAAAAAAAAjU/nHi-vA0GCcE82_R4qE80iMHv2VWdGlbwwCEwYBhgL/s640/1node.png)

## Scaffold Angular
To scaffold Angular, enter "ng new Contact-App --skip-install" command in VS code terminal as below.

![contactapp](https://4.bp.blogspot.com/-l_pbMBrcF6U/W0G-YkbmcMI/AAAAAAAAAjk/B9Je08IpAakzehABzrCxa8N688vFG5wmQCEwYBhgL/s400/2node.png)

Here, --skip-install option is used to skip installation of the npm packages. And Contact-App is our Angular app name.

![contactapp](https://4.bp.blogspot.com/-xN03EetKCVU/W0G-Y8XzzpI/AAAAAAAAAjU/9rVMOxS9KrwyBa9WXd7CYQEDM98A5gynQCEwYBhgL/s1600/3project.png)

After the project is created, we move all the files & folders from Contact-App to Root folder as below:

![contactapp](https://1.bp.blogspot.com/-ID9brfd7i_o/W0G-ZOVtLrI/AAAAAAAAAjY/iop2MTxE_E4sa0_N2TZSAl5vqOtRafhKwCEwYBhgL/s640/4project.png)

Then we enter ‘npm install’ command in terminal to install all required packages for Angular.

## Change Angular Configuration
Go to "angular.json" file, it is a configuration schema file.
We changed "wwwroot" folder path in OutputPath.

![contactapp](https://3.bp.blogspot.com/-hP8ZEdiIR6Q/W0G-ZjkDweI/AAAAAAAAAjc/YoeIW4JZqmA8TcHvOtctihVdNfOMuqFkACEwYBhgL/s400/5path.png)

Then enter ng build command in terminal.

![contactapp](https://2.bp.blogspot.com/-nbSKlmRPLko/W0G-ZyyGzqI/AAAAAAAAAjY/vhta5RKjuhozuDjas0mFZabCGwmB4iu2wCEwYBhgL/s640/6build.png)

We can see generated files under wwwroot folder.

![contactapp](https://2.bp.blogspot.com/-fH3KmcSeiCM/W0G-ad2JAqI/AAAAAAAAAjc/Wt--CLndxNUTRaHlVwKDbajnHt_F7xjlACEwYBhgL/s320/7folderpath.png)

## Configure startup to route to angular application
We have added the following code to configure method in "startup.cs". Here we have setup mvc default rout to index.html that generates under wwwroot folder.

```cs
//Redirect non api calls to angular app that will handle routing of the app.    
app.Use(async (context, next) => {  
    await next();  
    if (context.Response.StatusCode == 404 && !Path.HasExtension(context.Request.Path.Value) && !context.Request.Path.Value.StartsWith("/api/")) {  
        context.Request.Path = "/index.html";  
        await next();  
    }  
});  
// configure the app to serve index.html from /wwwroot folder    
app.UseDefaultFiles();  
app.UseStaticFiles();  
// configure the app for usage as api    
app.UseMvcWithDefaultRoute(); 
```
Now run project by enter "dotnet run" command in terminal and open "localhost:5000" URL in browser.

![contactapp](https://1.bp.blogspot.com/-4Fuk8RJTryc/W0G-aymNF1I/AAAAAAAAAjk/aB20RcqKwSk6TnlzLL__k9UDmEJFR_EBQCEwYBhgL/s320/9run.png)

## Setup angular material UI
Install angular material packages and dependencies.

![contactapp](https://3.bp.blogspot.com/-lTCEkR4joKk/W0G-WGjJcrI/AAAAAAAAAjk/x5skpX1Po2c4VR6upL7gInvR_gP3ZEnegCEwYBhgL/s640/10material.png)

Please go through this [article](https://material.angular.io/) to get more details about material components. 

```bat
ng add @angular/material
npm install -d @angular/cdk hammerjs
```
To use angular material UI components in our Angular contact application, we have created a separate module named "app.material.module.ts" in app folder.
In this module, we imported all dependant material components and we have included in our main app module in this module .
Then, we have imported Angular material theme in the main style.css in src folder

```ts
@import '~@angular/material/prebuilt-themes/deeppurple-amber.css';
```
Then, we added this link of material icons into index.html in src folder.

```html
<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">  
```
## Generate contact list & form component
To generate new component, we used "ng generate component" command as below:

![contactapp](https://4.bp.blogspot.com/-f6YlMsQbIgk/W0G-WUV26UI/AAAAAAAAAjg/me0C4ncE_lk_36NK7j75Dt6nNN4T-6_mQCEwYBhgL/s640/11component.png)

This we have set up routing for that component in "app.routing.ts".

```ts
import {  
    ModuleWithProviders  
} from '@angular/core';  
import {  
    Routes,  
    RouterModule  
} from '@angular/router';  
import {  
    AppComponent  
} from './app.component';  
import {  
    ContactlistComponent  
} from './contactlist/contactlist.component';  
import {  
    ContactformComponent  
} from './contactform/contactform.component';  
const appRoutes: Routes = [{  
    path: '',  
    pathMatch: 'full',  
    component: ContactlistComponent  
}, {  
    path: 'contactform',  
    component: ContactformComponent  
}];  
export const Routing: ModuleWithProviders = RouterModule.forRoot(appRoutes);  
```
## Create angular services
To create a consumed contact API that we have created Part 1; we are generating Angular contact services class in app folder using this command:

![contactapp](https://4.bp.blogspot.com/-KQOlxISdNQQ/W0G-XARnB_I/AAAAAAAAAjU/2PupTACrMoMV16oltU9JHDstzldHh2ODACEwYBhgL/s640/12component.png)

```bat
ng generate service contact
```

```ts
import {  
    Injectable  
} from '@angular/core';  
import {  
    HttpClient,  
    HttpParams,  
    HttpErrorResponse  
} from '@angular/common/http';  
import {  
    HttpHeaders  
} from '@angular/common/http';  
import {  
    Observable,  
    throwError  
} from 'rxjs';  
import {  
    catchError  
} from 'rxjs/operators';  
import {  
    IContact  
} from '../model/contact';  
const httpOptions = {  
    headers: new HttpHeaders({  
        'Content-Type': 'application/json'  
    })  
};  
@Injectable()  
export class ContactService {  
    constructor(private http: HttpClient) {}  
    // get all contact data    
    getAllContact(url: string): Observable < IContact[] > {  
        return this.http.get < IContact[] > (url).pipe(catchError(this.handleError));  
    }  
    // insert new contact details    
    addContact(url: string, contact: IContact): Observable < any > {  
        return this.http.post(url, JSON.stringify(contact), httpOptions).pipe(catchError(this.handleError));  
    }  
    // update contact details    
    updateContact(url: string, id: number, contact: IContact): Observable < any > {  
        const newurl = `${url}?id=${id}`;  
        return this.http.put(newurl, contact, httpOptions).pipe(catchError(this.handleError));  
    }  
    // delete contact information    
    deleteContact(url: string, id: number): Observable < any > {  
        const newurl = `${url}?id=${id}`; // DELETE api/contact?id=42    
        return this.http.delete(newurl, httpOptions).pipe(catchError(this.handleError));  
    }  
    // custom handler    
    private handleError(error: HttpErrorResponse) {  
        if (error.error instanceof ErrorEvent) {  
            // A client-side or network error occurred. Handle it accordingly.    
            console.error('An error occurred:', error.error.message);  
        } else {  
            // the backend returned an unsuccessful response code.    
            // the response body may contain clues as to what went wrong,    
            console.error(`Backend returned code ${error.status}, ` + `body was: ${error.error}`);  
        }  
        // return an observable with a user-facing error message    
        return throwError('Something bad happened; please try again later.');  
    }  
} 
``` 
## Update main app html template
We have included router outlet place holder to render component based on Angular route.

```html
<!--The content below is only a placeholder and can be replaced.-->  
<mat-toolbar> <span>Contact Application</span> </mat-toolbar>  
<router-outlet></router-outlet>  
```
## Update contact form component
In contact form component, we used form group named as 'contactFrm' to bind model with each control and injected dialog data to bind form based on request parameter.

contactform.component.html

```html 
<form  (ngSubmit)="onSubmit(contactFrm)"  [formGroup]="contactFrm">  
  <h2>{{data.modalTitle}}</h2>  
    
  <div>  
      <mat-form-field appearance="outline">  
      <mat-label>Name</mat-label>  
      <input matInput placeholder="Name" formControlName="name">  
      <!-- <mat-icon matSuffix>sentiment_very_satisfied</mat-icon> -->  
      <!-- <mat-hint>Hint</mat-hint> -->  
      <mat-error *ngIf="formErrors.name">  
        {{ formErrors.name }}  
      </mat-error>  
    </mat-form-field>  
  </div>  
  <div>  
    <mat-form-field appearance="outline">  
      <mat-label>Email</mat-label>  
      <input type="email" matInput placeholder="email" formControlName="email">  
      <mat-error *ngIf="formErrors.email">  
        {{ formErrors.email }}  
      </mat-error>  
    </mat-form-field>  
    
  </div>  
  <p>  
      <mat-radio-group class="contact-radio-group" formControlName="gender" >  
        <mat-radio-button class="contact-radio-button" *ngFor="let gndr of genders" [value]="gndr.id">  
          {{ gndr.name }}  
        </mat-radio-button>  
      </mat-radio-group>  
      <mat-error *ngIf="formErrors.gender">  
        {{ formErrors.gender }}  
      </mat-error>  
  </p>  
  <div>  
    <mat-form-field appearance="outline">  
      <input matInput [matDatepicker]="picker" placeholder="Choose a birthday" formControlName="birth">  
      <mat-datepicker-toggle matSuffix [for]="picker"></mat-datepicker-toggle>  
      <mat-datepicker #picker></mat-datepicker>  
      
    <mat-error *ngIf="formErrors.birth ">  
      {{ formErrors.birth }}  
    </mat-error>  
    </mat-form-field>  
  </div>  
  <div>  
    <mat-form-field appearance="outline">  
      <mat-select placeholder="Select a Technology" formControlName="techno">  
        <mat-option>-- None --</mat-option>  
        <mat-option *ngFor="let techno  of technologies" [value]="techno">  
          {{ techno }}  
        </mat-option>  
      </mat-select>  
      <mat-error *ngIf="formErrors.techno ">  
        {{ formErrors.techno }}  
      </mat-error>  
    </mat-form-field>  
  </div>  
  <div>  
    <mat-form-field appearance="outline">  
      <textarea matInput placeholder="Message..." formControlName="message"></textarea>  
      <mat-error *ngIf="formErrors.message ">  
        {{ formErrors.message }}  
      </mat-error>  
    </mat-form-field>  
  </div>  
  <div>  
    
    <button type="button" mat-raised-button color="warn" (click)="dialogRef.close()">Cancel</button>   
    <button type="submit" mat-raised-button color="primary" [disabled]="contactFrm.invalid">{{data.modalBtnTitle}}</button>  
  </div>  
    
  </form> 
``` 
To see contactform.component.ts code please refer this GitHub link and final form  looks  as shown in the below screenshot.

![contactapp](https://1.bp.blogspot.com/-g5f5WDeVNzw/W0G-XJ0_j8I/AAAAAAAAAjc/pU13owfAzZgd2TwrWtR4yr0Hqsfr7Iu7wCEwYBhgL/s320/16contactform.png)

## Update contact list component
In contact list component, we have used material table to bind contact record using ‘MatTableDataSource’ and called contact service to retrieve data from API. Then we used MatDialog to open contact form component.
contactlist.component.html

```html 
<div class="spinner" *ngIf="loadingState; else contactlist">  
<mat-spinner></mat-spinner>  
</div>  
<ng-template class="contactlist" #contactlist>  
  <h2 style="text-align: center;">Contact List</h2>  
  <div class="contactlist-container mat-elevation-z8">  
    <div><button title="Create" mat-raised-button color="accent" (click)="addContact()">Create</button></div>  
    <table mat-table #table [dataSource]="dataSource">  
  
      <!-- Id Column -->  
      <!-- <ng-container matColumnDef="id">  
      <th mat-header-cell *matHeaderCellDef> Id </th>  
      <td mat-cell *matCellDef="let element"> {{element.id}} </td>  
    </ng-container> -->  
  
      <!-- Name Column -->  
      <ng-container matColumnDef="name">  
        <th mat-header-cell *matHeaderCellDef> Name </th>  
        <td mat-cell *matCellDef="let element"> {{element.name}} </td>  
      </ng-container>  
  
      <!-- Email Column -->  
      <ng-container matColumnDef="email">  
        <th mat-header-cell *matHeaderCellDef> Email </th>  
        <td mat-cell *matCellDef="let element"> {{element.email}} </td>  
      </ng-container>  
  
      <!-- Gender Column -->  
      <ng-container matColumnDef="gender">  
        <th mat-header-cell *matHeaderCellDef> Gender </th>  
        <td mat-cell *matCellDef="let element"> {{getGender(element.gender)}} </td>  
      </ng-container>  
  
      <!-- Birth Column -->  
      <ng-container matColumnDef="birth">  
        <th mat-header-cell *matHeaderCellDef> Birthday </th>  
        <td mat-cell *matCellDef="let element"> {{element.birth | date: 'MM-dd-yyyy' }} </td>  
      </ng-container>  
  
      <!-- Technology Column -->  
      <ng-container matColumnDef="techno">  
        <th mat-header-cell *matHeaderCellDef> Technology </th>  
        <td mat-cell *matCellDef="let element"> {{element.techno}} </td>  
      </ng-container>  
  
      <!-- Message Column -->  
      <ng-container matColumnDef="message">  
        <th mat-header-cell *matHeaderCellDef> Message </th>  
        <td mat-cell *matCellDef="let element"> {{element.message}} </td>  
      </ng-container>  
  
      <ng-container matColumnDef="action">  
        <th mat-header-cell *matHeaderCellDef> Action </th>  
        <td mat-cell *matCellDef="let element">  
          <button title="Edit" mat-raised-button color="primary" (click)="editContact(element.id)">Edit</button>  
          <button title="Delete" mat-raised-button color="warn" (click)="deleteContact(element.id)">Delete</button>  
        </td>  
      </ng-container>  
  
      <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>  
      <tr mat-row *matRowDef="let row; columns: displayedColumns;"></tr>  
    </table>  
  
  </div>  
</ng-template>  
```
To see contactlist.component.ts code please refer to this GitHub [https://github.com/JayeshAgrawal/contact-app/blob/master/src/app/contactlist/contactlist.component.ts](https://github.com/JayeshAgrawal/contact-app/blob/master/src/app/contactlist/contactlist.component.ts).

## Build components and run the project
Our component is ready. Now build Angular project using 'ng build' command and then run the project by entering 'dotnet run' command in terminal.

![contactapp](https://3.bp.blogspot.com/-KhjhmYPYBjQ/W0G-aQqnK6I/AAAAAAAAAjg/R1wrRVu4l0Q7xfZ45ZARb8UiVM1QmoOKgCEwYBhgL/s400/8run.png)

![contactapp](https://4.bp.blogspot.com/-ClrLTfKcjUQ/W0G-Xe37f7I/AAAAAAAAAjU/2ZzaFUi09h8Gt6QQ6LSuYwgS1E6gwEYJgCEwYBhgL/s640/1list.png)

## Conclusion

This is how we created contact application using ASP.Net Core Web API, Angular 6.0, and Visual Studio code.
You can see the entire code on GitHub [https://github.com/JayeshAgrawal/contact-app/](https://github.com/JayeshAgrawal/contact-app/) and fork it for the starter project.

Please let me know your feedback.