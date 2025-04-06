ng new blog-app --routing --style=scss
cd blog-app
ng generate module features/blog --route blog --module app
ng generate component features/blog/containers/blog-list
ng generate component features/blog/components/post-card
ng generate service features/blog/services/blog

src/app/
├── features/blog/
│   ├── containers/blog-list/
│   ├── components/post-card/
│   ├── services/blog.service.ts
│   ├── models/post.model.ts
│   ├── blog-routing.module.ts
│   └── blog.module.ts
├── app-routing.module.ts
└── app.module.ts

export interface Post {
  id: number;
  title: string;
  body: string;
}

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { Post } from '../models/post.model';

@Injectable({ providedIn: 'root' })
export class BlogService {
  private apiUrl = 'https://jsonplaceholder.typicode.com/posts';

  constructor(private http: HttpClient) {}

  getPosts(): Observable<Post[]> {
    return this.http.get<Post[]>(this.apiUrl);
  }
}

import { Component, OnInit } from '@angular/core';
import { Observable } from 'rxjs';
import { BlogService } from '../../services/blog.service';
import { Post } from '../../models/post.model';

@Component({
  selector: 'app-blog-list',
  templateUrl: './blog-list.component.html',
  styleUrls: ['./blog-list.component.scss']
})
export class BlogListComponent implements OnInit {
  posts$!: Observable<Post[]>;

  constructor(private blogService: BlogService) {}

  ngOnInit(): void {
    this.posts$ = this.blogService.getPosts();
  }
}

<section>
  <h1>Blog Posts</h1>
  <div *ngFor="let post of posts$ | async">
    <app-post-card [post]="post"></app-post-card>
  </div>
</section>

import { Component, Input } from '@angular/core';
import { Post } from '../../models/post.model';

@Component({
  selector: 'app-post-card',
  templateUrl: './post-card.component.html',
  styleUrls: ['./post-card.component.scss']
})
export class PostCardComponent {
  @Input() post!: Post;
}

<article class="card">
  <h2>{{ post.title }}</h2>
  <p>{{ post.body }}</p>
</article>

import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { BlogListComponent } from './containers/blog-list/blog-list.component';

const routes: Routes = [
  { path: '', component: BlogListComponent }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class BlogRoutingModule {}

import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { BlogRoutingModule } from './blog-routing.module';
import { BlogListComponent } from './containers/blog-list/blog-list.component';
import { PostCardComponent } from './components/post-card/post-card.component';

@NgModule({
  declarations: [BlogListComponent, PostCardComponent],
  imports: [CommonModule, BlogRoutingModule]
})
export class BlogModule {}

import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, AppRoutingModule, HttpClientModule],
  bootstrap: [AppComponent]
})
export class AppModule {}

import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  { path: '', redirectTo: 'blog', pathMatch: 'full' },
  {
    path: 'blog',
    loadChildren: () =>
      import('./features/blog/blog.module').then((m) => m.BlogModule)
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
