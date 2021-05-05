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
  
  /*
    { path: 'home/:id', component: HomeComponent },
    { path: 'children', children: [
        { path: '', component: HomeComponent },
        { path: 'one', component: HomeComponent },
        { path: 'two', component: HomeComponent }
      ]
    },
    { path: 'redirect', redirectTo: 'home' },
    { path: '**', redirectTo: 'home' }
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

<!--
<p routerLink="/home/{{ 4 }}">click me please</p>
<p [routerLink]="['/home', 4]">click me again</p>
<p (click)="navigateMe(5)">navigate me by ts code</p>
-->
```
in `app.component.html`
```html
<p routerLink="/todo">go todo</p>

<router-outlet></router-outlet>
```
in `pages/home.component.ts`
```ts
import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-todo',
  templateUrl: './todo.component.html',
  styleUrls: ['./todo.component.scss']
})
export class TodoComponent implements OnInit {
  constructor(private router: Router) { }

  ngOnInit(): void { }

  navigateMe(id: number): void {
    this.router.navigate(['/home', id])
  }
}
```
