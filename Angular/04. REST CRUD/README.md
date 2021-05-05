# REST CRUD
* Route page
* Add module
* Set url
* Set models
* Modify service

### Route page
In `app-routing.module.ts`
```ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { TodoComponent } from './pages/todo/todo.component';

const routes: Routes = [
  { path: 'todo', component: TodoComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```
### Add module
in `app.module.ts`
```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { SharedModule } from './modules/shared.module';

import { HttpClientModule } from '@angular/common/http';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    SharedModule,
    // ADD THIS
    HttpClientModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
### Set url
in `src/environments/environment.ts`
```ts
// This file can be replaced during build by using the `fileReplacements` array.
// `ng build --prod` replaces `environment.ts` with `environment.prod.ts`.
// The list of file replacements can be found in `angular.json`.

export const environment = {
  apiUrl: 'https://localhost:5001',
  production: false
};

/*
 * For easier debugging in development mode, you can import the following file
 * to ignore zone related error stack frames such as `zone.run`, `zoneDelegate.invokeTask`.
 *
 * This import should be commented out in production mode because it will have a negative impact
 * on performance if an error is thrown.
 */
// import 'zone.js/dist/zone-error';  // Included with Angular CLI.
```
### Set models
in `models/todo-input.model.ts`
```ts
export interface TodoInput {
    title: string
    description: string
    isDone: boolean
}
```
in `models/todo-payload.model.ts`
```ts
export interface TodoPayload {
    id: number
    title: string
    description: string
    created: Date
    updated: Date
    isDone: boolean
}
```
### Modify service
in `services/todo.service.ts`
```ts
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { BehaviorSubject, EMPTY, Observable, of } from 'rxjs';
import { TodoPayload } from '../models/todo-payload.model';
import { catchError, first, shareReplay, switchMap, tap } from 'rxjs/operators'
import { environment } from 'src/environments/environment';
import { TodoInput } from '../models/todo-input.model';

@Injectable({
  providedIn: 'root'
})
export class TodoService {
  private path: string = 'todo'
  todos$ = new BehaviorSubject<TodoPayload[]>([])
  todo$ = new BehaviorSubject<TodoPayload | null>(null)

  constructor(private http: HttpClient) { }

  getTodos(): Observable<TodoPayload[]> {
    return this.http.get<TodoPayload[]>(`${environment.apiUrl}/${this.path}`)
      .pipe(
        shareReplay(),
        switchMap((todos: TodoPayload[]): BehaviorSubject<TodoPayload[]> => {
          this.todos$.next(todos)
          return this.todos$
        }),
        catchError(_ => {
          return of([])
        })
      )
  }

  getTodo(id: number): Observable<TodoPayload> {
    return this.http.get<TodoPayload>(`${environment.apiUrl}/${this.path}/${id}`).pipe(
      shareReplay(),
      catchError(_ => EMPTY),
      first()
    )
  }

  addTodo(input: TodoInput): Observable<TodoPayload> {
    return this.http.post<TodoPayload>(`${environment.apiUrl}/${this.path}`, input)
      .pipe(
        shareReplay(),
        tap((todo: TodoPayload): void => {
          const values: TodoPayload[] = [...this.todos$.value, todo]
          this.todos$.next(values)
        }),
        catchError(_ => EMPTY),
        first()
      )
  }

  setTodo(id: number, input: TodoInput): Observable<TodoPayload> {
    return this.http.put<TodoPayload>(`${environment.apiUrl}/${this.path}/${id}`, input)
      .pipe(
        shareReplay(),
        tap((todo: TodoPayload): void => {
          const values: TodoPayload[] = [...this.todos$.value]
          const valueIndex: number = values.findIndex((item: TodoPayload) => item.id === id)
          values[valueIndex] = todo
          this.todos$.next(values)
        }),
        catchError(_ => EMPTY),
        first()
      )
  }

  removeTodo(id: number): Observable<TodoPayload> {
    return this.http.delete<TodoPayload>(`${environment.apiUrl}/${this.path}/${id}`)
      .pipe(
        shareReplay(),
        tap((todo: TodoPayload): void => {
          const values: TodoPayload[] = this.todos$.value.filter(value => value.id !== todo.id)
          this.todos$.next(values)
        }),
        catchError(_ => EMPTY),
        first()
      )
  }
}
```
