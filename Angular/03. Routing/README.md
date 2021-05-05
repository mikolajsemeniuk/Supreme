# Routing
* Route
* Navigate
### Route
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
### Navigate
in `modules/shared.module.ts`
```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { TodoComponent } from '../components/todo/todo.component';
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
<p routerLink="/">go home</p>
```
in `app.component.html`
```html
<p routerLink="/todo">go todo</p>
```
