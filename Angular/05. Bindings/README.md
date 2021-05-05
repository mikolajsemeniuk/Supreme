# Bindings
* Add module
* Models (Variables)
* Models (Observables)
* ngFor, ngIf

### Add module
in `modules/shared.module.ts`
```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { TodoComponent } from '../pages/todo/todo.component';
import { RouterModule } from '@angular/router';
import { FormsModule } from '@angular/forms';

@NgModule({
  declarations: [
    TodoComponent
  ],
  imports: [
    CommonModule,
    RouterModule,
    // ADD THIS
    FormsModule
  ],
  exports: [
    TodoComponent
  ]
})
export class SharedModule { }

```
### Models (Variables)
in `pages/todo.component.ts`
```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-todo',
  templateUrl: './todo.component.html',
  styleUrls: ['./todo.component.scss']
})
export class TodoComponent {
  value = 1
}
```
in `pages/todo.component.html`
```html
<input [(ngModel)]="value" type="number">

<p>
  {{ value }}
</p>
```
### Models (Observables)
in `pages/todo.component.ts`
```ts
import { Component } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';
@Component({
  selector: 'app-todo',
  templateUrl: './todo.component.html',
  styleUrls: ['./todo.component.scss']
})
export class TodoComponent {
  todo$ = new BehaviorSubject<TodoInput>({ title: '', description: '', isDone: false })

  valueHandler(input: number) {
    this.value.next(input);
  }
}
```
in `pages/todo.component.html`
```html
<input [ngModel]="value | async"
    (ngModelChange)="valueHandler($event)"
    type="number">

<p>
  {{ value | async }}
</p>
```
### ngFor, ngIf
in `pages/todo/todo.component.ts`
```ts
import { Component, OnInit } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';
import { TodoInput } from 'src/app/models/todo-input.model';
import { TodoPayload } from 'src/app/models/todo-payload.model';
import { TodoService } from 'src/app/services/todo.service';

@Component({
  selector: 'app-todo',
  templateUrl: './todo.component.html',
  styleUrls: ['./todo.component.scss']
})
export class TodoComponent implements OnInit {
  todos$ = new Observable<TodoPayload[]>()
  todo$ = new BehaviorSubject<TodoInput>({ title: '', description: '', isDone: false })
  
  constructor(private service: TodoService) { }

  ngOnInit(): void { this.getTodos() }

  getTodos(): void {
    this.todos$ = this.service.getTodos()
  }

  setTodo(payload: any, current: TodoInput): void {
    this.todo$.next({ ...current, ...payload })
  }

  removeTodo(id: number): void {
    this.service
      .removeTodo(id)
      .subscribe()
  }
}
```
in `pages/todo/todo.component.html`
```html
<p routerLink="/">go home</p>

<h2>
    Add new
</h2>
<div *ngIf="todo$ | async as todo">
    <input (ngModelChange)="setTodo({ title: $event }, todo)" [ngModel]="todo.title" type="text">
    <input (ngModelChange)="setTodo({ description: $event }, todo)" [ngModel]="todo.description" type="text">
    <input (ngModelChange)="setTodo({ isDone: $event }, todo)" [ngModel]="todo.isDone" type="checkbox">
</div>
your object:<br>
{{ todo$ | async | json }}

<ng-container *ngIf="todos$ | async as todos" else #loading>
    <div *ngFor="let todo of todos" style="margin: 50px;padding:40px;border:solid 1px black">
        <p>
            {{ todo.id }}
        </p>
        <p>
            {{ todo.title }}
        </p>
        <p>
            {{ todo.created }}
        </p>
        <p>
            {{ todo.updated }}
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
    loading...
</ng-template>
```
## Resources
* [bind observable](https://stackoverflow.com/questions/38844835/extending-angular-2-ngmodel-directive-to-use-observables)
