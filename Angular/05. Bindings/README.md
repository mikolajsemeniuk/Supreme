# Bindings
* Add module
* Models (Variables)
* Models (Observables)
* ngIf
* ngFor

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
  value = new BehaviorSubject<number>(0)

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
## Resources
* [bind observable](https://stackoverflow.com/questions/38844835/extending-angular-2-ngmodel-directive-to-use-observables)
