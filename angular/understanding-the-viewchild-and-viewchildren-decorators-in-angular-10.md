# Understanding the ViewChild and ViewChildren decorators in Angular 10
The `@ViewChild` and `@ViewChildren` decorators in Angular provide access to child elements in the view DOM by setting up view queries. A view query is a requested reference to a child element within a component view which contains metadata of the element. The scope of these decorators is limited to the component view and its embedded child views. These decorators are especially helpful in instances where being able to access and modify elements within the view in conventional ways is not possible.

For example, if a library ships with a component or directive with a public non-input or non-output property you'd like to change, these decorators would allow you to access and modify them. These decorators are also helpful in exposing providers configured in child components to ibject dependencies (like services, configuration values, etc.) that the main component may not have access to.

### View queries and AfterViewInit lifecycle hook
The `AfterViewInit` lifecycle hook is called when the component view and its child view are completely initialized. So for immediate modifications or assignments, the best place to access view queries would be in the `ngAfterViewInit` callbacks because the view queries are already resolved and set. Trying to access them before `ngAfterViewInit` responds may generate undefined values. However, the `@ViewChild()` decorator provides a `static` property that can be set to resolve a view query before change detection runs. 

### The ViewChild decorator
This decorator takes 3 properties, a `selector`, a `read`, and a `static` property. The `read` and `static` properties are optional. These properties are specified like this:
```ts
@ViewChild(selector {read: readValue, static: staticValue}) property;
```

**Supported ViewChild selectors**

The `selector` property specifies what child element within the component view is to be queried. According to the @ViewChild documentation, five kinds of selectors are supported. These are:

1. Classes with @Component or @Directive decorators
In this first example, MenuItemComponent is a queried from the MenuComponent view:
```ts
@Component({
    selector: 'menu-item',
    template: `<p>{{menuText}}</p>`
})
export class MenuItemComponent {
    @Input() menuText: string;
}

@Component({
    selector: 'menu',
    template: `<menu-item [menuText]="'Contact Us'"></menu-item>`
})
export class MenuComponent {
    @ViewChild(MenuItemComponent) menu: MenuItem; 
}
```

Here's an example with a directive:
```ts
@Directive({
    selector: '[textHighlight]'
})
export class TextHighlightDirective {}

@Component({
    selector: 'profile',
    template: `<p textHighlight>Some text to highlight</p>`
})
export class ProfileComponent {
    @ViewChild(TextHighlightDirective) highlightedText: TextHighlightDirective; 
}
```

2. A template reference variable as a string. Template reference variables are commonly used within templates but in this instance, it is used to configure a view query:
```ts
@Component({
    selector: 'menu-item',
    template: `<p>{{menuText}}</p>`
})
export class MenuItemComponent {
    @Input() menuText: string;
}

@Component({
    selector: 'menu',
    template: `
        <menu-item #aboutUs [menuText]="'About Us'"></menu-item>
        <menu-item #contactUs [menuText]="'Contact Us'"></menu-item>
    `
})
export class MenuComponent {
    @ViewChild('aboutUs') aboutItem: MenuItem;
    @ViewChild('contactUs') contactItem: MenuItem;
}
```

3. A provider defined in the child component tree of the current component. In this example, the SampleService is specified as a provider token for the FirstChildComponentClass. Since `<first-child>` is an element in the ParentComponent we can access the SampleService from it using the SampleService class as a token:
```ts
export class SampleService {}

@Component({
    selector: 'first-child',
    providers: [SampleService],
})
export class FirstChildComponent {}

@Component({
    selector: 'parent',
    template: '<first-child></first-child>'
})
export class ParentComponent {
    @ViewChild(SampleService) sampleService: SampleService; 
}
```

4. A provider defined through a string token. Although this is stated in the documentation, getting a provider through this method returns undefined values. This is a regression in Ivy which is enabled by default in Angular 9. A fix for this has been made. To get this work, you'll need to disable Ivy in the tsconfig.json file and use ViewEngine instead:
```json
{
    "angularCompilerOptions": {
        "enableIvy": false
    }
}
```
Here's how you can use a provider defined through a string token as selector:
```ts
@Component({
    selector: 'first-child',
    providers: [{ provide: 'TokenA', useValue: 'ValueA' }]
})
export class FirstChildComponent {}

@Component({
    selector: 'parent',
    template: '<first-child></first-child>'
})
export class ParentComponent {
    @ViewChild('TokenA') providerA: string; 
}
```
However, if you'd like to use this type of selector with Ivy, you can use the `read` property to acquire a view query:
```ts
export class ParentComponent {
    @ViewChild(FirstChildComponent, { read: 'TokenA' }) providerA: string;
}
```

5. A `TemplateRef`. It's possible to access embedded templates using the @ViewChild decorator, which can then be used to instantiate embedded views with `ViewContainerRef`:
```ts
@Component({
    selector: 'container',
    template: `<ng-template><h1>This container is empty</h1></ng-template>`
})
export class ContainerComponent {
    @ViewChild(TemplateRef) contTemplate: TemplateRef;
}
```
live example: https://stackblitz.com/edit/view-child-examples?file=src%2Fapp%2Fapp.component.ts

### Using the read property
The `read` property lets you select various tokens from the elements you query. These tokens could be provider tokens used for dependency injection or in some cases, be the type of view query. This is an optional property.

In the example below, the FirstChildComponent has a provider configuration with all kinds of dependency tokens like a class, string tokens, and an injection token. These tokens in conjunction with the `read` property can expose these dependencies to parent components that embed the FirstChildComponent. All these dependencies have been accessed in the ParentComponent using the `ViewChild` decorator and the `read` property sepcifying each of the corresponding tokens.

It's also possible to specify the type the view query should be, using the `read` property. In the same example, the `fcElementRef` and `fcComponent` properties are both queries of FirstChildComponent but are of different types based on what the `read` property was specified as:
```ts
export class SampleService {}

export const ExampleServiceToken = new InjectionToken<string>('ExampleService');

@Component({
    selector: 'first-child',
    providers: [
        SampleService,
        { provide: 'TokenA', useValue: 'valueA' },
        { provide: 'TokenB', useValue: 123 },
        { provide: ExampleServiceToken, useExisting: SampleService },
        { provide: 'TokenC', useValue: true }
    ]
})
export class FirstChildComponent {}

@Component({
    selector: 'parent',
    template: `<first-child></first-child>`
})
export class ParentComponent {
    @ViewChild(FirstChildComponent, { read: 'TokenA' }) dependencyA: string;
    @ViewChild(FirstChildComponent, { read: 'TokenB' }) dependencyB: number;
    @ViewChild(FirstChildComponent, { read: 'TokenC' }) dependencyC: boolean;
    @ViewChild(FirstChildComponent, { read: SampleService }) sampleService: SampleService;
    @ViewChild(FirstChildComponent, { read: ElementRef }) fcElementRef: ElementRef;
    @ViewChild(FirstChildComponent, { read: FirstChildComponent }) fcComponent: FirstChildComponent;
    @ViewChild(FirstChildComponent, { read: ExampleServiceToken }) exampleService: SampleService;
}
```

### Using the static property
The `static` property takes a boolean value and is optional. By default, it is false. If it is true, the view query is resolved before the complete competent view and data-bound properties are fully initialized. If set to false, the view query is resolved after the component view and data-bound properties are completely initialized.

In this example, the paragraph element is queried using both true and false `static` properties and the values logged for each in the ngOnInit and ngAfterViewInit callbacks:
```ts
@Component({
    selector: 'display-name',
    template: `<p #displayName>{{name}}</p>`
})
export class DisplayNameComponent implements OnInit, AfterViewInit {
    @ViewChild('displayName', { static: true }) staticName: ElementRef;
    @ViewChild('displayName', { static: false }) nonStaticName: ElementRef;

    name: string = "Jane";

    ngOnInit() {
        logValues('OnInit')
    }

    ngAfterViewInit() {
        logValues('AfterViewInit')
    }

    logValues(eventType: string) {
        console.log(`[${eventType}]\n staticName: ${this.staticName}, name value: "${this.staticName.nativeElement.innerHTML}"\n nonStaticName: ${this.nonStaticName}, name value: "${this.nonStaticName.nativeElement.innerHTML}"\n`);
    }
}
```
This is what will be logged:
```
[OnInit]
staticName: [object Object], name value: "" // static: true
nonStaticName: undefined, name value: "" // static: false

[AfterViewInit]
staticName: [object Object], name value: "Jane" // static: true
nonStaticName: [object Object], name value: "Jane"  // static: false
```

In the ngOnInit callback, none of the interpolated values have been initialized but with `{ static: true }`, the `staticName` view query is already resolved but the `nonStaticName` is undefined. However, after the AfterViewInit event, all the view queries have been resolved.

## ViewChildren
The @ViewChildren decorator works similarly to the @ViewChild decorator but instead of configuring a single view query, it gets a query list. From the component view DOM, it retrieves a `QueryList` of child elements. This list is updated when any changes are made to the child elements. The @ViewChildren decorator takes two properties, a `selector` and a `read` property. These properties work in the same way as in the @ViewChild decorator. Any child elements that match will be part of the list. Here's an example:
```ts
@Component({
    selector: 'item-label',
    template: `<h6>{{labelText}}</h6>`
})
export class ItemLabelComponent {
    @Input() labelText: string;
}

@Component({
    selector: 'item',
    template: `<item-label *ngFor="let label of labels" [labelText]="label"></item-label>`
})
export class ItemComponent {
    @ViewChildren(ItemLabelComponent) allLabels: ItemLabelComponent;
    
    labels = ['recent', 'popular', 'new'];
}
```
The length of `allLabels`  will be three as all the `<item-label>` will be selected. The QueryList has a number of methods you could use to manipulate the view queries.

For a more expansive illustration of how to use the @ViewChildren decorator and specify the read property, check out: https://stackblitz.com/edit/view-child-examples

## Conclusion:
The @ViewChild and @ViewChildren decorators are excellent utilities for querying child elements in views.