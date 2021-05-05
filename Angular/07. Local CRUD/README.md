# CRUD
* Declare model
* Configure service
* Configure component
### Declare model
in `models/todo.model.ts`
```ts
export interface Todo {
    id: number
    title: string
    description: string
    createdAt: Date
    isDone: boolean
}
```
### Configure service
in `services/todo.service.ts`
```ts
import { Injectable } from '@angular/core';
import { BehaviorSubject, EMPTY, Observable, of } from 'rxjs';
import { catchError, first, switchMap, tap } from 'rxjs/operators';
import { Todo } from '../models/todo.model';

// helper
let data = [
  {
    id: 1,
    title: 'one',
    description: 'first',
    createdAt: new Date,
    isDone: false
  },
  {
    id: 2,
    title: 'two',
    description: 'second',
    createdAt: new Date,
    isDone: true
  }
]

@Injectable({
  providedIn: 'root'
})
export class TodoService {
  todos$ = new BehaviorSubject<Todo[]>([])

  constructor() { }

  getTodos(): Observable<Todo[]> {
    return of(data)
      .pipe(
        switchMap((todos: Todo[]): BehaviorSubject<Todo[]> => {
          this.todos$.next(todos)
          return this.todos$
        }),
        catchError(_ => {
          return of([])
        })
      )
  }

  addTodo(todo: Todo): Observable<Todo> {
    return of(todo)
      .pipe(
        tap((todo: Todo): void => {
          const values: Todo[] = [...this.todos$.value, todo]
          this.todos$.next(values)
        }),
        catchError(_ => EMPTY),
        first()
      )
  }

  setTodo(id: number, todo: Todo): Observable<Todo> {
    return of(todo)
      .pipe(
        tap((todo: Todo): void => {
          const values: Todo[] = [...this.todos$.value]
          const valueIndex: number = values.findIndex((item: Todo) => item.id === id)
          values[valueIndex] = todo
          this.todos$.next(values)
        }),
        catchError(_ => EMPTY),
        first()
      )
  }

  removeTodo(id: number): Observable<Todo> {
    return of(this.todos$.value[0])
      .pipe(
        tap(_ => {
          const values: Todo[] = this.todos$.value.filter(value => value.id !== id)
          this.todos$.next(values)
        }),
        catchError(_ => EMPTY),
        first()
      )
  }
}
```
### Configure component
in `components/todo/todo.component.ts`
```ts
import { Injectable } from '@angular/core';
import { BehaviorSubject, EMPTY, Observable, of } from 'rxjs';
import { catchError, first, shareReplay, switchMap, tap } from 'rxjs/operators';
import { Todo } from '../models/todo.model';

// helper
let data = [
  {
    id: 1,
    title: 'one',
    description: 'first',
    createdAt: new Date,
    isDone: false
  },
  {
    id: 2,
    title: 'two',
    description: 'second',
    createdAt: new Date,
    isDone: true
  }
]

@Injectable({
  providedIn: 'root'
})
export class TodoService {
  todos$ = new BehaviorSubject<Todo[]>([])

  constructor() { }

  getTodos(): Observable<Todo[]> {
    return of(data)
    .pipe(
      shareReplay(),
      switchMap((todos: Todo[]): BehaviorSubject<Todo[]> => {
        this.todos$.next(todos)
        return this.todos$
      }),
      catchError(_ => {
        return of([])
      })
    )
  }

  addTodo(todo: Todo): Observable<Todo> {
    return of(todo)
      .pipe(
        shareReplay(),
        tap((todo: Todo): void => {
          const values: Todo[] = [...this.todos$.value, todo]
          this.todos$.next(values)
        }),
        catchError(_ => EMPTY),
        first()
      )
  }

  setTodo(id: number, todo: Todo): Observable<Todo> {
    return of(todo)
      .pipe(
        shareReplay(),
        tap((todo: Todo): void => {
          const values: Todo[] = [...this.todos$.value]
          const valueIndex: number = values.findIndex((item: Todo) => item.id === id)
          values[valueIndex] = todo
          this.todos$.next(values)
        }),
        catchError(_ => EMPTY),
        first()
      )
  }

  removeTodo(id: number): Observable<Todo> {
    return of(this.todos$.value[0])
      .pipe(
        shareReplay(),
        tap(_ => {
          const values: Todo[] = this.todos$.value.filter(value => value.id !== id)
          this.todos$.next(values)
        }),
        catchError(_ => EMPTY),
        first()
      )
  }
}
```
in `components/todo/todo.component.html`
```html
<p>todo works!</p>

<p (click)="addTodo()">
    add me
</p>
<div *ngFor="let todo of todos$ | async as todos; let i = index">
    <p>
        {{ todo.id }}, {{ todo.title }}, {{ todo.createdAt }}, {{ todo.isDone }}
    </p>
    <span style="color:green" (click)="setTodo(todo.id, todo)">Edit</span>
    <span style="color:red" (click)="removeTodo(todo.id)">Remove</span>
</div>
```
### Check
```ts
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { BehaviorSubject, EMPTY, Observable, of } from 'rxjs';
import { catchError, first, shareReplay, switchMap, tap } from 'rxjs/operators';
import { Todo } from '../models/todo.model';

// // helper
// let data = [
//   {
//     id: 1,
//     title: 'one',
//     description: 'first',
//     created: new Date,
//     updated: new Date,
//     isDone: false
//   },
//   {
//     id: 2,
//     title: 'two',
//     description: 'second',
//     createdAt: new Date,
//     created: new Date,
//     updated: new Date,
//     isDone: true
//   }
// ]

@Injectable({
  providedIn: 'root'
})
export class TodoService {
  todos$ = new BehaviorSubject<Todo[]>([])

  constructor(private http: HttpClient) { }

  getTodos(): Observable<Todo[]> {
    return this.http.get<Todo[]>(`https://localhost:5001/todo`)
    .pipe(
      shareReplay(),
      switchMap((todos: Todo[]): BehaviorSubject<Todo[]> => {
        this.todos$.next(todos)
        return this.todos$
      }),
      catchError(_ => {
        return of([])
      })
    )
  }

  addTodo(todo: Todo): Observable<Todo> {
    let payload = { title: todo.title, description: todo.description, isDone: todo.isDone }
    return this.http.post<Todo>(`https://localhost:5001/todo`, payload)
      .pipe(
        shareReplay(),
        tap((todo: Todo): void => {
          const values: Todo[] = [...this.todos$.value, todo]
          this.todos$.next(values)
        }),
        catchError(_ => EMPTY),
        first()
      )
  }

  setTodo(id: number, todo: Todo): Observable<Todo> {
    let payload = { title: todo.title, description: todo.description, isDone: todo.isDone }
    return this.http.put<Todo>(`https://localhost:5001/todo/${id}`, payload)
      .pipe(
        shareReplay(),
        tap((todo: Todo): void => {
          const values: Todo[] = [...this.todos$.value]
          const valueIndex: number = values.findIndex((item: Todo) => item.id === id)
          values[valueIndex] = todo
          this.todos$.next(values)
        }),
        catchError(_ => EMPTY),
        first()
      )
  }

  removeTodo(id: number): Observable<Todo> {
    return this.http.delete<Todo>(`https://localhost:5001/todo/${id}`)
      .pipe(
        shareReplay(),
        tap(_ => {
          const values: Todo[] = this.todos$.value.filter(value => value.id !== id)
          this.todos$.next(values)
        }),
        catchError(_ => EMPTY),
        first()
      )
  }
}
```
