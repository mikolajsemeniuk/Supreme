# Routing
* Route
* Navigate
### Route
In `app-routing.module.ts`
```ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { TodoComponent } from './components/todo/todo.component';

const routes: Routes = [
  { path: 'todo', component: TodoComponent }
  /*
  { path: 'todo/:id', component: TodoComponent },
  { path: 'children', children: [
      { path: '', component: TodoComponent },
      { path: 'one', component: TodoComponent },
      { path: 'two', component: TodoComponent }
    ]
  },
  { path: 'redirect', redirectTo: 'todo' },
  { path: '**', redirectTo: 'todo' }
  */
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```
### Navigate
in `modules/shared.module.ts`
```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { TodoComponent } from '../components/todo/todo.component';
// ADD THIS
import { RouterModule } from '@angular/router';

@NgModule({
  declarations: [
    TodoComponent
  ],
  imports: [
    CommonModule,
    // ADD THIS
    RouterModule
  ],
  exports: [
    TodoComponent
  ]
})
export class SharedModule { }
```
in `pages/todo.component.html`
```html
<p routerLink="/">home works!</p>
<p routerLink="/todo/{{ 4 }}">click me please</p>
<p [routerLink]="['/todo', 5]">click me again</p>
<p (click)="navigateMe(6)">navigate me by ts code</p>
```
in `pages/todo.component.ts`
```ts
import { Component } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-todo',
  templateUrl: './todo.component.html',
  styleUrls: ['./todo.component.scss']
})
export class TodoComponent {
  constructor(private router: Router) { }

  navigateMe(id: number): void {
    this.router.navigate(['/todo', id])
  }
}
```
