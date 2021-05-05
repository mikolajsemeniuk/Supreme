# NGRX
* Add NGRX
* Add recuder

### Add NGRX
in `terminal`
```sh
ng add @ngrx/schematics@latest &&
ng add @ngrx/store@latest --minimal false &&
ng add @ngrx/effects@latest
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
    !environment.production ? StoreDevtoolsModule.instrument() : [],
    EffectsModule.forRoot([TodoEffects,]),
    StoreDevtoolsModule.instrument({ maxAge: 25, logOnly: environment.production })
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
import { createReducer, on } from '@ngrx/store';
import { TodoPayload } from 'src/app/models/todo-payload.model';
import { addTodoSuccess, loadTodosSuccess, removeTodoSuccess, setTodoSuccess } from './todo.actions';


export const todoFeatureKey = 'todo';

export interface State {
  todos: TodoPayload[]
}

export const initialState: State = {
  todos: []
};


export const reducer = createReducer(
  initialState,
  on(loadTodosSuccess, (state: State, action) => {
    return {
      todos: action.todos
    }
  }),
  on(addTodoSuccess, (state: State, action) => {
    return {
      todos: [...state.todos, action.todo]
    }
  }),
  on(setTodoSuccess, (state: State, action) => {
    return {
      todos: [...state.todos.flatMap(value => value.id == action.todo.id ? action.todo : value)]
    }
  }),
  on(removeTodoSuccess, (state: State, action) => {
    return {
      todos: [...state.todos.filter(todo => todo.id != action.todo.id)]
    }
  })
);
```
### Add action
```sh
ng generate action store/Todo --flat false --skipTests=true
```
in `store/todo/todo.actions.ts`
```ts
import { createAction, props } from '@ngrx/store';
import { TodoInput } from 'src/app/models/todo-input.model';
import { TodoPayload } from 'src/app/models/todo-payload.model';

export const loadTodos = createAction(
  '[Todo] Load Todos'
);

export const loadTodosSuccess = createAction(
  '[Todo] Load Todos Success',
  props<{ todos: TodoPayload[] }>()
);

export const loadTodosFailure = createAction(
  '[Todo] Load Todos Failure',
  props<{ todos: TodoPayload[] }>()
);

export const addTodo = createAction(
  '[Todo] Add Todo',
  props<{ todo: TodoInput }>()
);

export const addTodoSuccess = createAction(
  '[Todo] Add Todo Success',
  props<{ todo: TodoPayload }>()
);

export const addTodoFailure = createAction(
  '[Todo] Add Todo Failure'
);

export const setTodo = createAction(
  '[Todo] Set Todo',
  props<{ id: number, todo: TodoInput }>()
);

export const setTodoSuccess = createAction(
  '[Todo] Set Todo Success',
  props<{ todo: TodoPayload }>()
);

export const setTodoFailure = createAction(
  '[Todo] Set Todo Failure'
);

export const removeTodo = createAction(
  '[Todo] Remove Todo',
  props<{ id: number }>()
);

export const removeTodoSuccess = createAction(
  '[Todo] Remove Todo Success',
  props<{ todo: TodoPayload }>()
);

export const removeTodoFailure = createAction(
  '[Todo] Remove Todo Failure'
);
```
### Add Selector
```sh
ng generate selector store/Todo --flat false --skipTests=true
```
in `store/todo/todo.selectors.ts`
```ts
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { State } from './todo.reducer';

const state = createFeatureSelector<State>('todo')
export const todos = createSelector(
    state,
    state => state.todos
)
```
### Add effect
```sh
ng add @ngrx/effects@latest
ng generate effect store/Todo --flat false --root -m app.module.ts --skipTests=true
```
in `store/todo/todo.effects.ts`
```ts
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { EMPTY, of } from 'rxjs';
import { catchError, exhaustMap, map, mergeMap, switchMap } from 'rxjs/operators';
import { TodoService } from 'src/app/services/todo.service';
import { addTodo, addTodoFailure, addTodoSuccess, loadTodos, loadTodosFailure, loadTodosSuccess, removeTodo, removeTodoFailure, removeTodoSuccess, setTodo, setTodoFailure, setTodoSuccess } from './todo.actions';



@Injectable()
export class TodoEffects {

  getTodos$ = createEffect(() => this.actions$.pipe(
    ofType(loadTodos),
    mergeMap(() => this.service.getTodos()
      .pipe(
        map(movies => loadTodosSuccess({ todos: movies })),
        catchError(() => of(loadTodosFailure({ todos: [] })))
      ))
    )
  )

  addTodo$ = createEffect(() => this.actions$.pipe(
    ofType(addTodo),
    exhaustMap(action => this.service.addTodo(action.todo)
      .pipe(
        map(todo => addTodoSuccess({ todo: todo }),
        catchError(() => of(addTodoFailure()))
      )
    ))
  ))

  setTodo$ = createEffect(() => this.actions$.pipe(
    ofType(setTodo),
    exhaustMap(action => this.service.setTodo(action.id, action.todo)
      .pipe(
        map(todo => setTodoSuccess({ todo: todo })),
        catchError(() => of(setTodoFailure()))
      )
    )
  ))

  removeTodo$ = createEffect(() => this.actions$.pipe(
    ofType(removeTodo),
    exhaustMap(action => this.service.removeTodo(action.id)
      .pipe(
        map(todo => removeTodoSuccess({ todo: todo})),
        catchError(() => of(removeTodoFailure()))
      )
    )
  ))

  constructor(private actions$: Actions, private service: TodoService) {}
}
```
### Add component
in `pages/todo/todo.component.ts`
```ts
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { addTodo, loadTodos, removeTodo, setTodo } from 'src/app/store/todo/todo.actions';
import { State } from 'src/app/store/todo/todo.reducer';
import { todos } from 'src/app/store/todo/todo.selectors';

@Component({
  selector: 'app-todo',
  templateUrl: './todo.component.html',
  styleUrls: ['./todo.component.scss']
})
export class TodoComponent implements OnInit {
  todos$ = this.store.select(todos)
  
  constructor(private store: Store<State>) { }

  ngOnInit(): void { this.getTodos() }

  getTodos(): void {
    this.store.dispatch(loadTodos())
  }

  addTodo(): void {
    let todo = { 
      title: Math.random().toString(36).replace(/[^a-z]+/g, '').substr(0, 5),
      description: Math.random().toString(36).replace(/[^a-z]+/g, '').substr(0, 5),
      isDone: Math.random() < 0.5
    }
    this.store.dispatch(addTodo({ todo }))
  }

  setTodo(id: number): void {
    let todo = {
      title: Math.random().toString(36).replace(/[^a-z]+/g, '').substr(0, 5),
      description: Math.random().toString(36).replace(/[^a-z]+/g, '').substr(0, 5),
      isDone: Math.random() < 0.5
    }
    this.store.dispatch(setTodo({ id, todo }))
  }

  removeTodo(id: number): void {
    this.store.dispatch(removeTodo({ id }))
  }
}
```
in `pages/todo/todo.component.ts`
```html
<p routerLink="/">go home</p>

<h2 (click)="addTodo()">
    add me
</h2>

<ng-container *ngIf="(todos$ | async) as todos else loading">
    <div *ngFor="let todo of todos" style="margin: 50px;padding:40px;border:solid 1px black">
        <p>
            {{ todo.id }}
        </p>
        <p>
            {{ todo.title }}
        </p>
        <p>
            {{ todo.description }}
        </p>
        <p>
            {{ todo.created }}
        </p>
        <p>
            {{ todo.updated }}
        </p>
        <p>
            {{ todo.isDone }}
        </p>
        <p>
            <span (click)="setTodo(todo.id)" style="color:green">edit me</span>
        </p>
        <p>
            <span (click)="removeTodo(todo.id)" style="color:red">delete me</span>
        </p>
    </div>
    <div *ngIf="todos.length == 0">
        you have no todos :(
    </div>
</ng-container>
<ng-template #loading>
    Observable is loading. Please wait
</ng-template>
```
### Add service
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

  constructor(private http: HttpClient) { }

  getTodos(): Observable<TodoPayload[]> {
    return this.http.get<TodoPayload[]>(`${environment.apiUrl}/${this.path}`)
  }

  getTodo(id: number): Observable<TodoPayload> {
    return this.http.get<TodoPayload>(`${environment.apiUrl}/${this.path}/${id}`)
  }

  addTodo(input: TodoInput): Observable<TodoPayload> {
    return this.http.post<TodoPayload>(`${environment.apiUrl}/${this.path}`, input)
  }

  setTodo(id: number, input: TodoInput): Observable<TodoPayload> {
    return this.http.put<TodoPayload>(`${environment.apiUrl}/${this.path}/${id}`, input)
  }

  removeTodo(id: number): Observable<TodoPayload> {
    return this.http.delete<TodoPayload>(`${environment.apiUrl}/${this.path}/${id}`)
  }
}
```
## Resources
* [flatMap (remeber to reset vscode after change `tsconfig.json`)](https://stackoverflow.com/questions/53556409/typescript-flatmap-flat-flatten-doesnt-exist-on-type-any)
