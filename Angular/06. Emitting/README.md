# Emitting
* Single emit
* Chain emit
### Single emit
in `pages/todo/todo.component.ts`
```ts
import { Component } from '@angular/core';
import { BehaviorSubject } from 'rxjs';
@Component({
  selector: 'app-todo',
  templateUrl: './todo.component.html',
  styleUrls: ['./todo.component.scss']
})
export class TodoComponent {
  value$ = new BehaviorSubject<number>(1)
  
  setValue(event: any) {
    this.value$.next(event);
  }
}
```
in `pages/todo/todo.component.html`
```html
<div id="todo" *ngIf="value$ | async as value">
    <h1 (click)="setValue(value + 1)">click me</h1>
    <app-todo-header [value]="value" (setValue)="setValue($event)">

    </app-todo-header>

    <app-todo-section>
        
    </app-todo-section>
</div>
```
in `components/todo/todo-header/todo-header.component.ts`
```ts
import { Component, EventEmitter, Input, Output } from '@angular/core';

@Component({
  selector: 'app-todo-header',
  templateUrl: './todo-header.component.html',
  styleUrls: ['./todo-header.component.scss']
})
export class TodoHeaderComponent {
  @Input()
  value: number = 0;

  @Output()
  setValue = new EventEmitter<Event | number>()
}
```
in `components/todo/todo-header/todo-header.component.html`
```html
<div id="todo-header">
    <p (click)="setValue.emit(value - 1)">
        todo-header works!
    </p>
    value: {{ value }}
</div>
```
### Chain emit
in `pages/todo/todo.component.ts`
```ts
import { Component } from '@angular/core';
import { BehaviorSubject } from 'rxjs';
@Component({
  selector: 'app-todo',
  templateUrl: './todo.component.html',
  styleUrls: ['./todo.component.scss']
})
export class TodoComponent {
  value$ = new BehaviorSubject<number>(1)
  
  setValue(event: any) {
    this.value$.next(event);
  }
}
```
in `pages/todo/todo.component.html`
```html
<div id="todo" *ngIf="value$ | async as value">
    <h1 (click)="setValue(value + 1)">click me</h1>
    <app-todo-header [value]="value" (setValue)="setValue($event)">

    </app-todo-header>

    <app-todo-section [value]="value" (setValue)="setValue($event)">
        
    </app-todo-section>
</div>
```
in `components/todo/todo-section/todo-section.component.ts`
```ts
import { Component, EventEmitter, Input, Output } from '@angular/core';

@Component({
  selector: 'app-todo-section',
  templateUrl: './todo-section.component.html',
  styleUrls: ['./todo-section.component.scss']
})
export class TodoSectionComponent {
  @Input()
  value: number = 0

  @Output()
  setValue = new EventEmitter<Event | number>()
}
```
in `components/todo/todo-section/todo-section.component.html`
```html
<div id="todo-section">
    <p>todo-section works! {{ value }}</p>
    <p (click)="setValue.emit(value * 2)">change me 2x</p>
    
    <app-todo-section-card [value]="value" (setValue)="setValue.emit($event)">
        
    </app-todo-section-card>
</div>
```
in `components/todo/todo-section-card/todo-section-card.component.ts`
```ts
import { Component, EventEmitter, Input, Output } from '@angular/core';

@Component({
  selector: 'app-todo-section-card',
  templateUrl: './todo-section-card.component.html',
  styleUrls: ['./todo-section-card.component.scss']
})
export class TodoSectionCardComponent {
  @Input()
  value: number = 0

  @Output()
  setValue = new EventEmitter<Event | number>();
}
```
in `components/todo/todo-section-card/todo-section-card.component.html`
```html
<p>todo-section-card works! heh: {{ value }}</p>
<button (click)="setValue.emit(value - 2)">change me - 2</button>
```
