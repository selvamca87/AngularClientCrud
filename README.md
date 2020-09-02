# AngularClientCrud

Angular CRUD Application example with Web API

In this tutorial, I will show you how to build an Angular 8 CRUD Application to consume Web APIs, display, modify & search data.

We will build an Angular 8 front-end Tutorial Application in that:

Each Tutorial has id, title, description, published status.
We can create, retrieve, update, delete Tutorials.
There is a Search bar for finding Tutorials by title.
Here are screenshots of our Angular CRUD Application.

Setup Angular 8 Project
Let’s open cmd and use Angular CLI to create a new Angular Project as following command:

ng new Angular8ClientCrud
? Would you like to add Angular routing? Yes
? Which stylesheet format would you like to use? CSS
We also need to generate some Components and Services:

ng g s services/tutorial

ng g c components/add-tutorial
ng g c components/tutorial-details
ng g c components/tutorials-list
Set up App Module
Open app.module.ts and import FormsModule, HttpClientModule:

...
import { FormsModule } from '@angular/forms';
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  declarations: [ ... ],
  imports: [
    ...
    FormsModule,
    HttpClientModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
Define Routes for Angular AppRoutingModule
There are 3 main routes:
– /tutorials for tutorials-list component
– /tutorials/:id for tutorial-details component
– /add for add-tutorial component

app-routing.module.ts

import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { TutorialsListComponent } from './components/tutorials-list/tutorials-list.component';
import { TutorialDetailsComponent } from './components/tutorial-details/tutorial-details.component';
import { AddTutorialComponent } from './components/add-tutorial/add-tutorial.component';

const routes: Routes = [
  { path: '', redirectTo: 'tutorials', pathMatch: 'full' },
  { path: 'tutorials', component: TutorialsListComponent },
  { path: 'tutorials/:id', component: TutorialDetailsComponent },
  { path: 'add', component: AddTutorialComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
Add Navbar and Router View to Angular CRUD App
Let’s open src/app.component.html, this App component is the root container for our application, it will contain a nav element.

<div>
  <nav class="navbar navbar-expand navbar-dark bg-dark">
    <a href="#" class="navbar-brand">bezKoder</a>
    <div class="navbar-nav mr-auto">
      <li class="nav-item">
        <a routerLink="tutorials" class="nav-link">Tutorials</a>
      </li>
      <li class="nav-item">
        <a routerLink="add" class="nav-link">Add</a>
      </li>
    </div>
  </nav>

  <div class="container mt-3">
    <router-outlet></router-outlet>
  </div>
</div>
Create Data Service
This service will use Angular HTTPClient to send HTTP requests.
You can see that its functions includes CRUD operations and finder method.

services/tutorial.service.ts

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

const baseUrl = 'http://localhost:8080/api/tutorials';

@Injectable({
  providedIn: 'root'
})
export class TutorialService {

  constructor(private http: HttpClient) { }

  getAll() {
    return this.http.get(baseUrl);
  }

  get(id) {
    return this.http.get(`${baseUrl}/${id}`);
  }

  create(data) {
    return this.http.post(baseUrl, data);
  }

  update(id, data) {
    return this.http.put(`${baseUrl}/${id}`, data);
  }

  delete(id) {
    return this.http.delete(`${baseUrl}/${id}`);
  }

  deleteAll() {
    return this.http.delete(baseUrl);
  }

  findByTitle(title) {
    return this.http.get(`${baseUrl}?title=${title}`);
  }
}
Create Angular Components
As you’ve known before, there are 3 components corresponding to 3 routes defined in AppRoutingModule.

Add new Item Component
This component has a Form to submit new Tutorial with 2 fields: title & description. It calls TutorialService.create() method.

components/add-tutorial/add-tutorial.component.ts

import { Component, OnInit } from '@angular/core';
import { TutorialService } from 'src/app/services/tutorial.service';

@Component({
  selector: 'app-add-tutorial',
  templateUrl: './add-tutorial.component.html',
  styleUrls: ['./add-tutorial.component.css']
})
export class AddTutorialComponent implements OnInit {
  tutorial = {
    title: '',
    description: '',
    published: false
  };
  submitted = false;

  constructor(private tutorialService: TutorialService) { }

  ngOnInit() {
  }

  saveTutorial() {
    const data = {
      title: this.tutorial.title,
      description: this.tutorial.description
    };

    this.tutorialService.create(data)
      .subscribe(
        response => {
          console.log(response);
          this.submitted = true;
        },
        error => {
          console.log(error);
        });
  }

  newTutorial() {
    this.submitted = false;
    this.tutorial = {
      title: '',
      description: '',
      published: false
    };
  }
}
components/add-tutorial/add-tutorial.component.html

<div class="submit-form">
  <div *ngIf="!submitted">
    <div class="form-group">
      <label for="title">Title</label>
      <input
        type="text"
        class="form-control"
        id="title"
        required
        [(ngModel)]="tutorial.title"
        name="title"
      />
    </div>

    <div class="form-group">
      <label for="description">Description</label>
      <input
        class="form-control"
        id="description"
        required
        [(ngModel)]="tutorial.description"
        name="description"
      />
    </div>

    <button (click)="saveTutorial()" class="btn btn-success">Submit</button>
  </div>

  <div *ngIf="submitted">
    <h4>You submitted successfully!</h4>
    <button class="btn btn-success" (click)="newTutorial()">Add</button>
  </div>
</div>
List of items Component
This component calls 3 TutorialService methods:

getAll()
deleteAll()
findByTitle()
components/tutorials-list/tutorials-list.component.ts

import { Component, OnInit } from '@angular/core';
import { TutorialService } from 'src/app/services/tutorial.service';

@Component({
  selector: 'app-tutorials-list',
  templateUrl: './tutorials-list.component.html',
  styleUrls: ['./tutorials-list.component.css']
})
export class TutorialsListComponent implements OnInit {

  tutorials: any;
  currentTutorial = null;
  currentIndex = -1;
  title = '';

  constructor(private tutorialService: TutorialService) { }

  ngOnInit() {
    this.retrieveTutorials();
  }

  retrieveTutorials() {
    this.tutorialService.getAll()
      .subscribe(
        data => {
          this.tutorials = data;
          console.log(data);
        },
        error => {
          console.log(error);
        });
  }

  refreshList() {
    this.retrieveTutorials();
    this.currentTutorial = null;
    this.currentIndex = -1;
  }

  setActiveTutorial(tutorial, index) {
    this.currentTutorial = tutorial;
    this.currentIndex = index;
  }

  removeAllTutorials() {
    this.tutorialService.deleteAll()
      .subscribe(
        response => {
          console.log(response);
          this.retrieveTutorials();
        },
        error => {
          console.log(error);
        });
  }

  searchTitle() {
    this.tutorialService.findByTitle(this.title)
      .subscribe(
        data => {
          this.tutorials = data;
          console.log(data);
        },
        error => {
          console.log(error);
        });
  }
}
components/tutorials-list/tutorials-list.component.html

<div class="list row">
  <div class="col-md-8">
    <div class="input-group mb-3">
      <input
        type="text"
        class="form-control"
        placeholder="Search by title"
        [(ngModel)]="title"
      />
      <div class="input-group-append">
        <button
          class="btn btn-outline-secondary"
          type="button"
          (click)="searchTitle()"
        >
          Search
        </button>
      </div>
    </div>
  </div>
  <div class="col-md-6">
    <h4>Tutorials List</h4>
    <ul class="list-group">
      <li
        class="list-group-item"
        *ngFor="let tutorial of tutorials; let i = index"
        [class.active]="i == currentIndex"
        (click)="setActiveTutorial(tutorial, i)"
      >
        {{ tutorial.title }}
      </li>
    </ul>

    <button class="m-3 btn btn-sm btn-danger" (click)="removeAllTutorials()">
      Remove All
    </button>
  </div>
  <div class="col-md-6">
    <div *ngIf="currentTutorial">
      <h4>Tutorial</h4>
      <div>
        <label><strong>Title:</strong></label> {{ currentTutorial.title }}
      </div>
      <div>
        <label><strong>Description:</strong></label>
        {{ currentTutorial.description }}
      </div>
      <div>
        <label><strong>Status:</strong></label>
        {{ currentTutorial.published ? "Published" : "Pending" }}
      </div>

      <a class="badge badge-warning" href="/tutorials/{{ currentTutorial.id }}">
        Edit
      </a>
    </div>

    <div *ngIf="!currentTutorial">
      <br />
      <p>Please click on a Tutorial...</p>
    </div>
  </div>
</div>
If you click on Edit button of any Tutorial, You will be directed to Tutorial page with url: /tutorials/:id.

You can add Pagination to this Component, just follow instruction in the post:
Angular 8 Pagination with ngx-pagination example

Item details Component
For getting data & update, delete the Tutorial, this component will use 3 TutorialService methods:

get()
update()
delete()
components/tutorial-details/tutorial-details.component.ts

import { Component, OnInit } from '@angular/core';
import { TutorialService } from 'src/app/services/tutorial.service';
import { ActivatedRoute, Router } from '@angular/router';

@Component({
  selector: 'app-tutorial-details',
  templateUrl: './tutorial-details.component.html',
  styleUrls: ['./tutorial-details.component.css']
})
export class TutorialDetailsComponent implements OnInit {
  currentTutorial = null;
  message = '';

  constructor(
    private tutorialService: TutorialService,
    private route: ActivatedRoute,
    private router: Router) { }

  ngOnInit() {
    this.message = '';
    this.getTutorial(this.route.snapshot.paramMap.get('id'));
  }

  getTutorial(id) {
    this.tutorialService.get(id)
      .subscribe(
        data => {
          this.currentTutorial = data;
          console.log(data);
        },
        error => {
          console.log(error);
        });
  }

  updatePublished(status) {
    const data = {
      title: this.currentTutorial.title,
      description: this.currentTutorial.description,
      published: status
    };

    this.tutorialService.update(this.currentTutorial.id, data)
      .subscribe(
        response => {
          this.currentTutorial.published = status;
          console.log(response);
        },
        error => {
          console.log(error);
        });
  }

  updateTutorial() {
    this.tutorialService.update(this.currentTutorial.id, this.currentTutorial)
      .subscribe(
        response => {
          console.log(response);
          this.message = 'The tutorial was updated successfully!';
        },
        error => {
          console.log(error);
        });
  }

  deleteTutorial() {
    this.tutorialService.delete(this.currentTutorial.id)
      .subscribe(
        response => {
          console.log(response);
          this.router.navigate(['/tutorials']);
        },
        error => {
          console.log(error);
        });
  }
}
components/tutorial-details/tutorial-details.component.html

<div *ngIf="currentTutorial" class="edit-form">
  <h4>Tutorial</h4>
  <form>
    <div class="form-group">
      <label for="title">Title</label>
      <input
        type="text"
        class="form-control"
        id="title"
        [(ngModel)]="currentTutorial.title"
        name="title"
      />
    </div>
    <div class="form-group">
      <label for="description">Description</label>
      <input
        type="text"
        class="form-control"
        id="description"
        [(ngModel)]="currentTutorial.description"
        name="description"
      />
    </div>

    <div class="form-group">
      <label><strong>Status:</strong></label>
      {{ currentTutorial.published ? "Published" : "Pending" }}
    </div>
  </form>

  <button
    class="badge badge-primary mr-2"
    *ngIf="currentTutorial.published"
    (click)="updatePublished(false)"
  >
    UnPublish
  </button>
  <button
    *ngIf="!currentTutorial.published"
    class="badge badge-primary mr-2"
    (click)="updatePublished(true)"
  >
    Publish
  </button>

  <button class="badge badge-danger mr-2" (click)="deleteTutorial()">
    Delete
  </button>

  <button type="submit" class="badge badge-success" (click)="updateTutorial()">
    Update
  </button>
  <p>{{ message }}</p>
</div>

<div *ngIf="!currentTutorial">
  <br />
  <p>Cannot access this Tutorial...</p>
</div>
