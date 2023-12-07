# How dependency injection works in Angular

In software development, there are several proposed patterns for handling dependency injection. Angular enforces the constructor injection pattern, which uses the constructor to pass in the dependencies of a class as parameters of the constructor.

Angular has it owns built-in dependency injection (DI) framework that provides dependencies to classes upon instatiation. This is an important feature for building scalable web applications in Angular.

## What is dependency injection in Angular?
According to Angular's document, dependency injection is "a design pattern in which a class requests dependencies from external sources rather than creating them."

In a nutshell, Angular dependency injection aims to decouple the implementation of services from components. This eases testing, overriding, and altering of services without affecting the components dependent on these services.

The best pratice for handling dependency injection in Angular is as follows:
```typescript
//app/products/product-list/product-list.component.ts

export class ProductListComponent implements OnInit {
    products: Product[];
    constructor(private productService: ProductService) {}

    ngOnInit():void {
        this.products = this.productService.getProducts()
    }
}
```
This way, the component does not need to know how to instantiate the service. Instead, it receives the dependency and injects it through its constructor. This approach make it easier to test the service.

## How to handle dependency injection in Angular
When handling dependency injection in an Angular app, you can either take an application-based or a component-based approach.

### Application-based dependency injection
The Angular DI framework makes dependencies available across the entire application by providing an injector that keeps a list of all dependencies the application needs. When a component or service wants to use a dependency, the injector first checks whether it has already created an instance of that dependency. If not, it creates a new one, returns it to the component, and reserves a copy for further use so that the next time the same dependency is requested, it returns the reserved dependency rather than creating a new one.

There are hierarchies associated with injectors in an Angular application. Whenever an Angular component defines a token in its constructor, the injector searches for a type that matches that token in the pool of registered providers. If no match is found, it delegates the search on the parent component's provider up through the component injector tree. If it finds the dependency, it stops and returns an instance of it to the component that requested it.

If the provider lookup finishes with no match, it returns to the injector of the component that requested the provider and searches through the injectors of all the parent modules up the module injector hierarchy until it reaches the root injector. If no match is found, Angular throws an exception. Otherwise, it returns an instance of the dependency on the component.

We already walked through some practical code snippets for this approach.

### Component-based dependency injection
This approach is known for injecting the dependencies directly into the component tree using the @Component decorator's `providers` property to register services with a component injector. This approach is commonly used in Angular applications.

When sharing dependencies across children components, the dependencies are shared across all the children component of the component that provides the dependencies. They are readily available for injection into constructors of the children components, causing each child component to reuse the same instance of the service from the parent component.

Let’s say we want to display a list of recently added products. Obviously, the model for displaying a list of recently added products is the same for displaying all products. Hence, we can share the `products` dependency (service) across the ProductListComponent and RecentProductComponent components.
```bash
ng generate component products/recentProducts --module=products
```

```typescript
// recent-products.component.ts

import { Component, OnInit } from '@angular/core';
import { ProductService } from '../product.service';
import { Product } from '../product.model';

@Component({
    selectors: 'app-recent-products',
    templateUrl: './recent-products.component.html',
    styleUrls: ['./recent-products.component.css']
})
export class RecentProductsComponent implements OnInit {
    products: Product[];
    constructor(private productService: ProductService) { }
    ngOnInit(): void {
        this.products = this.productService.getHeroes();
    }
}
```
Here, we inject ProductService into the constructor of RecentProductsComponent without actually providing it through the providers property of the @component decorator, as we did for ProductListComponent.

