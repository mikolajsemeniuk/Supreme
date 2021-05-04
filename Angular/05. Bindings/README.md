# Bindings
* Models
### Models
in `modules/shared.module.ts`
```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { TodoComponent } from '../pages/todo/todo.component';
import { TodoHeaderComponent } from '../components/todo-header/todo-header.component';
import { TodoSectionComponent } from '../components/todo-section/todo-section.component';
import { RouterModule } from '@angular/router';
// ADD THIS
import { FormsModule } from '@angular/forms';

@NgModule({
  declarations: [
    TodoComponent,
    TodoHeaderComponent,
    TodoSectionComponent
  ],
  imports: [
    CommonModule,
    RouterModule,
    // ADD THIS
    FormsModule
  ],
  exports: [
    TodoComponent,
    TodoHeaderComponent,
    TodoSectionComponent
  ]
})
export class SharedModule { }
```
in `workshop.component.ts`
```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-todo',
  templateUrl: './todo.component.html',
  styleUrls: ['./todo.component.scss']
})
export class TodoComponent {
  value = 1

  constructor() { }
}

```
in `workshop.component.html`
```html
<p routerLink="/">home works!</p>
<input [(ngModel)]="value" type="number">
<ul>
    <li>{{ value }}</li>
</ul>
```
