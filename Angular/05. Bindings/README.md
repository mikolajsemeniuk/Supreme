# Bindings
* Models
### Models
in `app.module.ts`
```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { SharedModule } from './modules/shared.module';
// ADD THIS
import { FormsModule } from '@angular/forms';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    SharedModule,
    // ADD THIS
    FormsModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
in `workshop.component.ts`
```ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-todo',
  templateUrl: './todo.component.html',
  styleUrls: ['./todo.component.scss']
})
export class TodoComponent implements OnInit {
  value = 1

  constructor() { }

  ngOnInit(): void { }
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
