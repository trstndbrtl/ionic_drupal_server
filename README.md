# Ionic Drupal Server
#### Create a service for ionic stream

## Drupal Configuration
>
This module create a views for Ionic app endpoint
>
dependencies:
  - drupal:views
  - drupal:node
  - basic_auth
  - serialization
  - rest
  - hal
  - restui
>
>
Enable CORS (Cross Origin Resource Sharing) for accessing the data between applications.
Change the CORS config as per the origins of `services.yml` file:
>
```yml
cors.config:
 enabled: true
 # Specify allowed headers, like 'x-allowed-header'.
 allowedHeaders: ['*'] 
 # Specify allowed request methods, specify ['*'] to allow all possible ones.
 allowedMethods: ['*']
 # Configure requests allowed from specific origins.
 allowedOrigins: ['*']
 # Sets the Access-Control-Expose-Headers header.
 exposedHeaders: true
 # Sets the Access-Control-Max-Age header.
 maxAge: false
 # Sets the Access-Control-Allow-Credentials header.
 supportsCredentials: true
```
>
### Access Api
> 
Database list
> 
Visit [{{site.url}}/ionic/v1/api?_format=hal_json](/admin/structure/views/view/ionic_server_api?_format=hal_json)
> 
Database article
> 
Visit [{{site.url}}/ionic/v1/api/{{nid}}?_format=hal_json](/admin/structure/views/view/ionic_server_api?_format=hal_json)

>
### Administration view
>
Visit [{{site.url}}/admin/structure/views/view/ionic_server_api](/admin/structure/views/view/ionic_server_api)


## Ionic Configuration
### Create a service

/providers/database/`database.service.js`

```ts
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';

@Injectable()
export class DataProvider {

  constructor(public http: HttpClient) {
    console.log('Hello DataProvider Provider');
  }

  prepareArticle(url: string) {
    return this.http.get(url);
  }
}
```
>
### Setup App page

/app/`app.module`

Import @angular/http, providers/database/database.service and declare it to @NgModule

```ts
import { HttpModule } from '@angular/http';
import { BrowserModule } from '@angular/platform-browser';
import { ErrorHandler, NgModule } from '@angular/core';
import { IonicApp, IonicErrorHandler, IonicModule } from 'ionic-angular';
import { HttpClientModule } from '@angular/common/http';

import { MyApp } from './app.component';
import { HomePage } from '../pages/home/home';

import { StatusBar } from '@ionic-native/status-bar';
import { SplashScreen } from '@ionic-native/splash-screen';

import { DataProvider } from '../providers/database/database.service';

@NgModule({
  declarations: [
    MyApp,
    HomePage,
  ],
  imports: [
    BrowserModule,
    IonicModule.forRoot(MyApp),
    HttpClientModule,
    HttpModule
  ],
  bootstrap: [IonicApp],
  entryComponents: [
    MyApp,
    HomePage,
  ],
  providers: [
    StatusBar,
    SplashScreen,
    {provide: ErrorHandler, useClass: IonicErrorHandler},
    DataProvider
  ]
})
export class AppModule {}
```

### List all articles on home page

/pages/home/`home.ts`

Dont forget to change `{{site.url}}` with the url of your site

```ts
import { Component } from '@angular/core';
import { NavController } from 'ionic-angular';
import { DataProvider } from '../../providers/database/database.service';

@Component({
  selector: 'page-home',
  templateUrl: 'home.html'
})
export class HomePage {

  display_row;
  articles;

  constructor(public navCtrl: NavController, private data: DataProvider) {}

  ionViewWillLoad() {
    this.display_row = this.data.prepareArticle('{{site.url}}/ionic/v1/api?_format=hal_json');
    this.display_row.subscribe(dt => { this.sendArticles(dt) }, err => { console.log(err); });
  }

  sendArticles(data) {
    this.articles = data;
  }
  
}
```

### The list views

/pages/home/`home.html`

Get the liste articles et loop it.

```html
<ion-content no-padding class="background-page">
  <ion-item *ngFor="let article of articles">
    <h2 [innerHTML]="article.title"></h2>
    <p [innerHTML]="article.body"></p>
  </ion-item>
</ion-content>
```
