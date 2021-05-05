# NGRX
* Add NGRX
* Add recuder

### Add NGRX
in `terminal`
```sh
ng add @ngrx/schematics@latest &&
ng add @ngrx/store@latest --minimal false &&
ng add @ngrx/store-devtools@latest
```
### Add Store
in `terminal`
```sh
ng generate store State --root --state-path store --module app.module.ts
```
in `app.module.ts`
```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { SharedModule } from './modules/shared.module';

import { HttpClientModule } from '@angular/common/http';
import { StoreModule } from '@ngrx/store';
import { StoreDevtoolsModule } from '@ngrx/store-devtools';
import { environment } from '../environments/environment';
import { reducers, metaReducers } from './store';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    SharedModule,
    HttpClientModule,
    StoreModule.forRoot(reducers, { metaReducers }),
    StoreDevtoolsModule.instrument({ maxAge: 25, logOnly: environment.production }),
    !environment.production ? StoreDevtoolsModule.instrument() : []
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
### Add reducer
```sh
ng generate reducer store/Todo --reducers index.ts --skipTests=true --flat false
```
in `store/todo/todo.reducer.ts`
```ts
import { Action, createReducer, on } from '@ngrx/store';
import { TodoPayload } from 'src/app/models/todo-payload.model';


export const todoFeatureKey = 'todo';

export interface State {
  todos: TodoPayload[]
}

export const initialState: State = {
  todos: []
};


export const reducer = createReducer(
  initialState,

);
```
### Add action
```sh
ng generate action store/Todo --flat false --skipTests=true
```
### Add reducer
```sh
ng generate selector store/Todo --flat false --skipTests=true
```
