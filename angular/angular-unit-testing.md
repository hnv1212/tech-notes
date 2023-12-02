## What is Angular testing?
Angular testing is a core feature available in every project set up with the Angular CLI.

There are 2 types of Angular testing:
1. **Unit testing** is the process of testing small, isolated pieces of code. Also known as isolated testing, unit tests do not use external resources, such as the network or a database.
2. **Functional testing** refers to testing the functionality of your Angular app from a user experience perspective - i.e., interacting with your app as it's running in a browser just as a user would

## What is Angular unit testing?
Unit testing in Angular refers to the process of testing individual units of code.

An Angular unit test aims to uncover issues such as incorrect logic, misbehaving functions, etc. by isolating pieces of code. This is sometimes more difficult than it sounds, especially for complex projects with poor separation of concerns. Angular is designed to help you write code in such a way that enables you to test your app's functions individually in isolation.

## Why you should unit test Angular apps
Angular unit testing enables you to test your app based on user behavior. While testing each possible behavior would be tedious, inefficient, and ineffective, writing tests for each coupling block in your application can help demonstrate how these blocks behave.

One of the easiest ways to test the strengths of these blocks is to write a test for each one. You don't necessarily need to wait until your users complain about how the input field behaves when the button is clicked. By writing a unit test for your blocks (components, services, etc.), you can easily detect when there is a break.

## How do you write an Angular test?
`app.component.spec.ts`

```typescript
import { TestBed, async } from '@angular/core/testing';
import { AppComponent } from './app.component';

describe('AppComponent', () => {
    beforeEach(async(() => {
        TestBed.configureTestingModule({
            declarations: [
                AppComponent
            ],
        }).compileComponents();
    }));

    it('should create the app', async(() => {
        const fixture = TestBed.createComponent(AppComponent);
        const app = fixture.debugElement.componentInstance;
        expect(app).toBeTruthy();
    }));

    it(`should have as title 'angular-unit-test'`, async(() => {
        const fixture = TestBed.createComponent(AppComponent);
        const app = fixture.debugElement.componentInstance;
        expect(app.title).toEqual('angular-unit-test');
    }));

    it('should render title in a h1 tag', async(() => {
        const fixture = TestBed.createComponent(AppComponent);
        fixture.detectChanges();
        const compiled = fixture.debugElement.nativeElement;
        expect(compiled.querySelector('h1').textContent).toContain('Welcome to angular-unit-test!')
    }));
});
```

Let’s run our first test to make sure nothing has broken yet:
```bash
ng test
```

## What is Karma in Angular?
Karma is a Javascript test runner that runs the unit test snippet in Angular. Karma also ensures the result of the test is printed out either in the console or in the file log.

By default, Angular runs on Karma. Other test runners include Mocha and Jasmine. Karma provides tools that make it easier to call Javascript tests while writing code in Angular.

## How to write a unit test in Angular
The Angular testing package includes 2 utilities called `TestBed` and `async`. `TestBed` is the main Angular utility package.

The `describe` container contains different blocks (`it`, `beforeEach`, `xit`, etc.). `beforeEach` runs before any other block. Other blocks do not depend on each other to run.

From the `app.component.spec.ts` the first block is the `beforeEach` inside the container (`describe`). This is the only block that runs before any other block (`it`). The declaration of the app module in `app.module.ts` file is simulated (declared) in the `beforeEach` block. The component (`AppComponent`) declared in the `beforeEach` block is the main component we want to have in this testing environment. The same logic applies to other test declaration.

The `compileComponents` object is called to compile your component's resources like the template, styles, etc. You might not necessarily compile your component if you are using webpack.

The `fixture.debugElement.componentInstance` creates an instance of the class (AppComponent). 

The third block demonstrates how you can have access to the properties of the created component (AppComponent). The only property added by default is the title. You can easily check if the title you set has changed or not from the instance of the component created.

The fourth block demonstrates how the test behaves in the browser environment. After creating the component, an instance of the created component (`detectChanges`) to simulate running on the browser environment is called. Now that the component has been rendered, you can have access to its child element by accessing the `nativeElement` object of the rendered component (`fixture.debugElement.nativeElement`):

## How to test an Angular service
Services often depend on other services that Angular injects into the constructor. In many cases, it is easy to create and inject these dependencies by adding `providedIn: root` to the injectable object, which makes it accessible by any component or service:

```typescript
import { Injectable } from "@angular/core";
import { QuoteModel } from "../model/QuoteModel";

@Injectable({
    providedIn: "root"
})
export class QuoteService {
    public quoteList: QuoteModel[] = []

    private daysOfTheWeeks = ["Sun", "Mon", "Tue", "Wed", "Thurs", "Fri", "Sat"];

    constructor() {}

    addNewQuote(quote: String) {
        const date = new Date();
        const dayOfTheWeek = this.daysOfTheWeeks[date.getDate()];
        const day = date.getDay();
        const year = date.getFullYear();
        this.quoteList.push(new QuoteModel(quote, `${dayOfTheWeek} ${day}, ${year}`))
    }

    getQuote() {
        return this.quoteList;
    }

    removeQuote(index: number) {
        this.quoteList.splice(index, 1)
    }
}
```

Here are a few ways to test the `QuoteService` class:

```typescript
/* tslint:disable:no-unused-variable */
import { QuoteService } from './Quote.service';

describe("QuoteService", () => {
    let service: QuoteService;

    beforeEach(() => {
        service = new QuoteService();
    });

    it("should create a post in an array", () => {
        const quoteText = "This is my first post";
        service.addNewQuote(quoteText);
        expect(service.quoteList.length).toBeGreaterThanOrEqual(1);
    });

    it("should remove a created post from the array of posts", () => {
        service.addNewQuote("This is my first post")
        service.removeQuote(0);
        expect(service.quoteList.length).toBeLessThan(1);
    })
})
```

In the first block, `beforeEach`, an instance of `QuoteService` is created to ensure it is only created once and to avoid repetition in other blocks except for some exceptional cases:

The first block tests if the post model `QuoteModel(text, date)` is created into an array by checking the length of the array. The length of the `quoteList` is expected to be 1.

The second block creates a post in an array and removes it immediately by calling `removeQuote` in the service object. The length of the `quoteList` is expected to be 0.

## How to test an Angular component
In our Angular unit testing  example app, the service is injected into the QuoteComponent to access its properties, which will be needed by the view:

```typescript
import { Component, OnInit } from '@angular/core';
import { QuoteService } from '../service/Quote.service';
import { QuoteModel } from '../mode/QuoteModel';

@Component({
    selector: 'app-Quotes',
    templateUrl: './Quotes.component.html',
    styleUrls: ['./Quotes.component.css']
})
export class QuotesComponent implements OnInit {
    public quoteList: QuoteModel[];
    public quoteText: String = "";

    constructor(
        private service: QuoteService
    ) {}

    ngOnInit() {
        this.quoteList = this.service.getQuote();
    }

    createNewQuote() {
        this.service.addNewQuote(this.quoteText)
        this.quoteText = "";
    }

    removeQuote(index) {
        this.service.removeQuote(index);
    }
}

<div class="container-fluid">
  <div class="row">
    <div class="col-8 col-sm-8 mb-3 offset-2">
      <div class="card">
        <div class="card-header">
          <h5>What Quote is on your mind ?</h5>
        </div>
        <div class="card-body">
          <div role="form">
            <div class="form-group col-8 offset-2">
              <textarea #quote class="form-control" rows="3" cols="8" [(ngModel)]="quoteText" name="quoteText"></textarea>
            </div>
            <div class="form-group text-center">
              <button class="btn btn-primary" (click)="createNewQuote()" [disabled]="quoteText == null">Create a new
                quote</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <div class="row">
    <div class="card mb-3 col-5 list-card" id="quote-cards" style="max-width: 18rem;" *ngFor="let quote of quoteList; let i = index"
      (click)="removeQuote(i)">
      <div class="card-body">
        <h6>{{ quote.text }}</h6>
      </div>
      <div class="card-footer text-muted">
        <small>Created on {{ quote.timeCreated }}</small>
      </div>
    </div>
  </div>
</div>
```

The first two blocks in the describe container run consecutively. In the first block, the `FormsModule` is imported into the configure test. This ensures the form's related directives, such as ngModel, can be used.

Also, the QuotesComponent is declared in the `configTestMod` similar to how the components are declared in `ngModule` residing in the appModule file. The second block `creates` a QuoteComponent and its instance, which will be used by the other blocks:

```typescript
let component: QuotesComponent;
let fixture: ComponentFixture<QuotesComponent>;

beforeEach(() => {
    TestBed.configureTestingModule({
        imports: [FormsModule],
        declarations: [QuotesComponent]
    });
});

beforeEach(() => {
    fixture = TestBed.createComponent(QuotesComponent);
    component = fixture.debugElement.componentInstance;
})
```

This block tests if the instance of the component that is created is defined:

```typescript
it("should create Quote component", () => {
    expect(component).toBeTruthy();
})
```

This injected service handles the manipulation of all operations (add, remove, fetch). The `quoteService` variable holds the injected service (`QuoteService`). At this point, the component is yet to be rendered until the `detectChanges` method is called:

```typescript
it("should use the quoteList from the service", () => {
    const quoteService = fixture.debugElement.injector.get(QuoteService);
    fixture.detectChanges();
    expect(quoteService.getQuote()).toEqual(component.quoteList);
})
```

Now let's test whether we can successfully create a post. The properties of the component can be accessed upon instantiation, so the rendered component detects the new changes when a value is passed into the `quoteText` model. The `nativeElement` object gives access to the HTML element rendered, which makes it easier to check if the `quote` added is part of the texts rendered:

```typescript
it("should create a new post", () => {
    component.quoteText = "test text";
    fixture.detectChanges();
    const compiled = fixture.debugElement.nativeElement;
    expect(compiled.innerHTML).toContain("test text");
});
```

Apart from having access to the HTML contents, you can also get an element by its CSS property. When the `quoteText` model is empty or null, the button is expected to be disabled:

```typescript
it("should disable the button when textArea is empty", () => {
    fixture.detectChanges();
    const button = fixture.debugElement.query(By.css("button"));
    expect(button.nativeElement.disabled).toBeTruthy();
});

it("should enable button when textArea is not empty", () => {
    component.quoteText = "test text";
    fixture.detectChanges();
    const button = fixture.debugElement.query(By.css("button"));
    expect(button.nativeElement.disabled).toBeFalsy();
});
```

Just like the way we access an element with its CSS property, we can also access an element by its class name. Multiple classes can be accessed at the same time using `By` e.g `By.css('.className.className')`.

The button clicks are simulated by calling the `triggerEventHandler`. The `event` type must be specified which, in this case, is `click`. A quote displayed is expected to be deleted from the quoteList when clicked on:

```typescript
it("should remove post upon card click", () => {
    component.quoteText = "test text";
    fixture.detectChanges();

    fixture.debugElement
        .query(By.css(".row"))
        .query(By.css(".card"))
        .triggerEventHandler("click", null);
    const compiled = fixture.debugElement.nativeElement;
    expect(compiled.innerHTML).toContain("test text");
});
```

## How to test an async operation in Angular 
`fetchQuotesFromServer` represents an async task that returns an array of quotes after 2 seconds:

```typescript
fetchQuotesFromServer(): Promise<QuoteModel[]> {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve([new QuoteModel("testing quote", "Mon 4, 2018")]);
        }, 2000);
    });
}
```

`spyOn` objects simulate how `fetchQuotesFromServer` method works. It accepts 2 arguments: `quoteService`, which is injected into the component, and the `fetchQuotesFromServer` method.

`fetchQuotesFromServer` is expected to return a promise. `spyOn` chains the method using `and` with a fake promise call, which is returned using `returnValue`. Because we want to emulate how the fetchQuotesFromServer works, we need to pass a `promise` that will resolve with a list of quotes.

Just as we did before, we'll call the `detectChanges` method to get the updated changes. `whenStable` allows access to results of all `async` tasks when they are done:

```typescript
it("should fetch data asynchronously", async () => {
    const fakedFetchedList = [
        new QuoteModel("testing", "Mon 4, 2018")
    ]
    const quoteService = fixture.debugElement.injector.get(QuoteService);
    let spy = spyOn(quoteService, "fetchQuotesFromServer").and.returnValue(
        Promise.resolve(fakedFetchedList)
    );
    fixture.detectChanges();
    fixture.whenStable().then(() => {
        expect(component.fetchedList).toBe(fakedFetchedList);
    })
})
```

## How to automatically generate an Angular unit test
the Angular ecosystem created the `ngentest` package to automate the generation of test specs for each component, directive, and others.

Assuming you wrote the code for your component and want to write a test for it, you'll have to install the `ngentest` package:
```bash
npm install ngentest -g
```

Next, you'll run the following command to auto-generate the unit test specs for your component:
```bash
gentest component-name.ts
```

We can also auto-generate the unit test specs for directives, pipes, and services:
```bash
gentest directive-name.ts -s #output to directive-name.spec.ts
gentest pipe-name.ts # output to pipe-name.test.ts
gentest service-name.ts # output to service-name.test.ts
```

## How `ngentest` works
`ngentest` parses the file name next to the `gentest` command and determines the proper file type. In our case, it is `component-name.ts` as seen in our previous command.

Next, `ngentest` builds data for a unit test from the parsed Typescript using the contents of the file, such as:
- Class
- Imports
- List of functions to test
- Inputs and outputs
- Mocks

Finally, `ngentest` generates the unit test.

The `gentest` package does not generate 100% test coverage, so you'll need to modify the generated unit test specs in order to achieve that.

## Conclusion
Angular ensures that test results are viewed in your browser. This will give a better visualization of the test results:
![Alt text](images/image-angular-test.png)