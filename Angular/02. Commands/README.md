# Commands
* Modules
* Components
* Services
* Models
* Guards

### Modules
in `terminal`
```sh
ng g m modules/shared --flat
```
in `src/app/modules/shared.module.ts`
```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';

@NgModule({
  declarations: [],
  imports: [
    CommonModule
  ],
  // ADD THIS
  exports: [
  
  ]
})
export class SharedModule { }
```
in `src/app/app.module.ts`
```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';

// ADD THIS
import { SharedModule } from './modules/shared.module';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    // ADD THIS
    SharedModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
### Components
in `terminal`
```sh
ng g c pages/todo --skipTests=true --module ./modules/shared &&
ng g c components/todo-header --skipTests=true --module ./modules/shared &&
ng g c components/todo-section --skipTests=true --module ./modules/shared
```
in `src/app/shared/modules/shared.module.ts`
```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
// ADD THIS
import { TodoComponent } from '../pages/todo/todo.component';
import { TodoHeaderComponent } from '../components/todo-header/todo-header.component';
import { TodoSectionComponent } from '../components/todo-section/todo-section.component';

@NgModule({
  declarations: [
    HomeComponent
  ],
  imports: [
    CommonModule
  ],
  exports: [
    // ADD THIS MANUALLY
    TodoComponent,
    TodoHeaderComponent,
    TodoSectionComponent
  ]
})
export class SharedModule { }
```
### Services
in `terminal`
```sh
ng g s services/todo --skipTests=true # without tests
```
### Models
in `terminal`
```sh
ng generate interface models/todo --type=model
```
### Guards
add it later...
