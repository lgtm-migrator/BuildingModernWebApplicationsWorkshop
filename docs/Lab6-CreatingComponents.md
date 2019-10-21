# Lab 6 - Creating Components

During this lab you will create components with the Angular CLI to display high scores and add new scores using the Angular Angular Reactive Components. You will learn how to navigate to these components using Angular Routing.

Goals for this lab:

- [Creating and displaying components](#inspect)
- [Displaying high scores in a list](#manage)
- [Creating an add score form using Angular Reactive Components](#working)

## Creating your first component and use it as your homepage

### 1. Create a high scores component with the Angular CLI

The following command will create a component. It is going to be used as your homepage:
```sh
ng generate component high-scores --module=app

# --module=app selects the AppModule to auto import this generated component

# You can also write the command short-handed:
# ng g c high-scores --module=app
```

The script created a component folder `./app/high-scores` containing 4 files:

1. **\high-scores.component.ts**
  
   The typescript file connects with the .html and scss files. This file will also contain all the logic for the component.

2. **\high-scores.component.spec.ts**

   With the spec class you are able to unit test your component. 

3. **\high-scores.component.html**
   
   This will show your content.

4. **\high-scores.component.scss**

   The scss file styles the component.

The Angular CLI command also added your component to the `AppModule`. You can find this file in `./angular-application/src/app/app.module.ts`

```ts
@NgModule({
  declarations: [
    AppComponent,
    // Automatically added component
    HighScoresComponent
    // #end
  ],
  imports: [
    /* ... */
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### 2. Set component as homepage in Angular Routing Module

Make the following changes to the `AppRoutingModule` which is automatically generated by the boiletplate Angular application `./app/app-routing.module.ts`

```ts
import { HighScoresComponent } from './high-scores/high-scores.component';

const routes: Routes = [
  // Add this route
  {
    path: "",
    component: HighScoresComponent
  },
  // #end
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

The `HighScoresComponent` component is now the homepage of your Angular application. Run the application to verify that this is correct and fix any errors that may occur.
```sh
npm start
```

You should now see the contents of the component: "high-scores works!"

## Create a new component to display a list of high scores

For this chapter you will also use the high-scores component you created in the previous chapter.

### 1. Create a sub-component

Create a sub-component inside the `./app/high-scores` component directory. Run the following command from inside the `app` folder.

```sh
ng generate component high-scores/high-scores-list --module=app
```

The script created files inside the `./high-scores/high-scores-list` folder for the `high-scores-list` component.

### 2. Create a model to hold high scores

Create a model interface to hold high scores

```sh
ng generate interface shared/models/high-score

# short-hand command:
# ng g i shared/models/high-score
```

The above script generated the following file `./app/shared/models/high-score.ts`. It is in a shared folder, so it can easily be used throughout the whole application when the application grows larger.

Add the following content to the file:
```ts
export interface HighScore {
  nickname: string;
  game: string;
  points: number;
}
```

> **interface vs class**
>
> An interface is used because you are only interested in the properties (signature) of a high score. Later you will see that these properties directly map onto the results from the API.

### 3. Input for high scores list component 

The next step is to have the high scores list component receive input to be displayed. 

TODO: verify location

Place the following TypeScript snippet in a new file named `./app/high-scores/high-scores-list/app-high-scores-list.ts`:
```ts
@Component({
  selector: "app-high-scores-list",
  templateUrl: "./high-scores-list.component.html",
  styleUrls: ["./high-scores-list.component.scss"]
})
export class HighScoresListComponent implements OnInit {
  // This property will contain data passed from another component
  @Input()
  highScores: HighScore[] = [];

  // This property makes you able to publish events to the component using this component
  @Output()
  onHighScoreSelected: EventEmitter<HighScore> = new EventEmitter();

  constructor() {}

  ngOnInit() {}

  onHighScoreClicked(highScore: HighScore): void {
    this.onHighScoreSelected.emit(highScore);
  }
}
```

`@Input` is used to have other components pass data inside components. `@Output` makes you able to publish events to the component using this component

### 4. Display high scores list using Material components

To display the high scores you will make some changes in the  file containing the HTML of the component. Open the file located at `./app/high-scores/high-scores-list/high-scores-list.component.html`

It will currently show an empty list when running the application. In the next chapter you will provide this component with actual high scores. For now, you are going to add some placeholders that will hold the actual data when available.

``` html
<mat-list>
  <mat-list-item matRipple *ngFor="let highScore of highScores" (click)="onHighScoreClicked(highScore)">
    <mat-icon matListIcon>dvr</mat-icon>
    <h3 matLine>{{ highScore.game }}: {{ highScore.points }}</h3>
    <p matLine>{{ highScore.nickname }}</p>
  </mat-list-item>
</mat-list>
```
 
- `(click)="onHighScoreClicked(highScore)"` will emit an event about a highscore being pressed
- \<mat-list-item> is an example of a Material component, which is imported in the `AppMaterialModule`
- **matRipple** is an Angular directive which shows a nice effect when an item is being clicked on
- **matListIcon** is an Angular directive which makes sure the icon is aligned left of the text.
- **matLine** is an Angular directive which adds additional functionality to the html components (showing text below eachother).

## Connect high scores and high scores list components

### 1. Initialize a list of highscores inside the HighScoresComponent

Create a list of high scores in the following file `./angular-application/src/app/high-scores/high-scores.component.ts`
 
```ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: "app-high-scores",
  templateUrl: "./high-scores.component.html",
  styleUrls: ["./high-scores.component.scss"]
})
export class HighScoresComponent implements OnInit {
  public highScores: HighScore[];

  constructor() {}

  ngOnInit() {
    this.highScores = [
      {
        nickname: "LX360" /* Alex Thissen gamertag */,
        game: "Pac-Man",
        points: 96070
      },
      {
        nickname: "Who is Thijs" /* Thijs Limmen gamertag */,
        game: "Space Invaders",
        points: 7028
      }
    ];
  }

  onHighScoreSelected(highScore: HighScore) {
    // Log the selected highscore to the developer console.
    console.log(`High score selected`, highScore);
  }
}
```

### 2. Pass high scores to high-scores-list component

Next, pass the array of high scores to the app-high-scores-list component. Replace the content in the file `./app/high-scores/high-scores.component.html` with:
``` html
<app-high-scores-list [highScores]="highScores" (onHighScoreSelected)="onHighScoreSelected($event)"></app-high-scores-list>
```

- `app-high-scores-list` is the name of the component you want to use
- `highScores` is the array of high scores
- `onHighScoreSelected($event)` where $event being the emitted value 

### 3. Run the application

You should now see the a list of high scores when navigating to the homepage.

```sh
npm start
```

> #### Suggestion
> 
> Run the application and validate whether you see the high score being logged to the console when you click on it.

## Creating an add score form component using Angular Reactive Components

In this chapter you are going to create a new add score component, navigate to it and display a form using the Angular Reactive Components

### 1. Create a new add score page

Create a new component, just like you did with the high scores.

```sh
ng generate component add-score --module=app

# --module=app selects the AppModule to auto import this generated component
```

The following component should have been generated:  `./angular-application/src/app/add-score/*`

### 2. Add navigation to the add score page

- Make the following changes to the Routing Module `./app/app-routing.module.ts`

```ts
const routes: Routes = [
  {
    path: "",
    component: HighScoresListComponent
  },
  // Add 'add score' route
  {
    path: "add-score",
    component: AddScoreComponent
  }
  // #end
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

Create a routerlink to the `add-score` page by adding the following to the high-scores-list component `./app/high-scores/high-scores-list/high-scores-list.component.html`

```html
<mat-list>
  <!-- ... -->
</mat-list>

<!-- Added link -->
<a mat-button color="primary" [routerLink]="['/add-score']">Submit new Score</a>
<!-- #end -->
```

You can optionally also create a routerlink in the material toolbar in the file `./app/app.component.html` by adding a new attribute in the existing `<a>` tag.

```html
<mat-toolbar color="primary">
  <mat-toolbar-row>
    <a mat-icon-button color="white" [routerLink]="['/add-score']">
      <mat-icon>add</mat-icon>
    </a>
  </mat-toolbar-row>
</mat-toolbar>
```

### 3. Create a form with Material Input components

Create a new form with Material components in the new add score component `./angular-application/src/app/add-score.component.html`

```html
<form class="add-score-form">
  <mat-form-field class="input-nickname">
    <input matInput formControlName="nickname" required placeholder="Nickname" />
  </mat-form-field>

  <mat-form-field class="input-game">
    <mat-label>Games</mat-label>
    <mat-select formControlName="game">
      <mat-option value="Pacman">Pac-Man</mat-option>
      <mat-option value="Space Invaders">Space Invaders</mat-option>
      <mat-option value="Donkey Kong">Donkey Kong</mat-option>
    </mat-select>
  </mat-form-field>

  <mat-form-field>
    <input matInput formControlName="points" type="number" placeholder="Points" value="0" />
  </mat-form-field>

  <button class="form-button" (click)="submitScore()" mat-raised-button color="primary">
    Submit Score
  </button>
</form>
```

In this form you see the following:
- Material Input and Material Select components to enhance the form
- `formControlName` for using Angular Reactive Forms in a moment
- `required` to mark input as required
- `(click)="submitScore()"` which will call a function inside the typescript component file.

Let's add some style to the component for better spacing `./app/add-score/add-score.component.scss`

```scss
.add-score-form {
  margin: 16px;
  max-width: 600px;

  .input-nickname {
    width: 100%;
  }

  .input-game {
    margin-right: 20px;
  }

  .form-button {
    display: block;
  }
}
```

As you can see above the css classes are nested, which means only classes within `.add-score-form` are affected, for example: `.add-score-form .input-nickname` in casual CSS.

### 4. Use Reactive Forms to add scores

Add the `ReactiveFormsModule` to the `AppModule` in file `./app/app.module.ts`

```ts
@NgModule({
  declarations: [
    /* ... */
  ],
  imports: [
    /* ... */
    /* Add this module */
    ReactiveFormsModule
    /* #end */
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Create the form using Angular `FormBuilder` in file `./app/add-score.component.ts`

```ts
@Component({
  selector: "app-add-score",
  templateUrl: "./add-score.component.html",
  styleUrls: ["./add-score.component.scss"]
})
export class AddScoreComponent implements OnInit {
  addScoreForm: FormGroup;

  constructor(private fb: FormBuilder) {}

  ngOnInit() {
    this.addScoreForm = this.fb.group({
      nickname: ["", Validators.required],
      game: [""],
      points: [0]
    });
  }

  submitScore(): void {
    console.log(this.addScoreForm.value);
  }
}
```

Also make the following changes to the HTML in file `./app/add-score.component.html` to attach the `addScoreForm`. The properties in the created `addScoreForm` map to the `formControlName` in the HTML file. 
```html
<!-- add [formGroup]="addScoreForm" to the <form> -->
<form class="add-score-form" [formGroup]="addScoreForm">
<!-- #end -->
  <!-- ... -->


  <!-- Show status of the addScoreForm -->
  <p>
    Form Status: {{ addScoreForm.status }}
  </p>
  <!-- #end -->
  <!-- ... -->
</form>
```

> #### Suggestion:
>
> Run the application and press the button. You should now see the values of the form logged to your developer console. Press F12 in Google Chrome to view the logs. You should also see the form becoming invalid and valid when changing the Nickname field, because it has the `required` validator on it.

> #### Pro tip:
>
> Go to https://angular.io/guide/reactive-forms to learn more about Angular Reactive Forms and validating forms.

## Wrap up
In this lab you have created components and navigated to them using the Angular Router. You used Material components to enhance your application and Angular Reactive Forms to get data from your form. In the next lab you will connect the components to the .NET Core API created in the earlier Labs.

Continue with [Lab 7 - Connect to web API](Lab7-ConnectToAPI.md).