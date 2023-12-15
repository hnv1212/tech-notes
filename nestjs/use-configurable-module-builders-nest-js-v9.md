# How to use configurable module builder in NestJS v9

## What is Nestjs module?
A module is a set of code **encapsulated to be injected into a NestJS application**. You can use modules to create custom services that are meant to perform specific tasks. For example, TypeORM is a typescript-based ORM. The NestJS team created a module that will inject an open database connection, allowing for database commands and queries from the injected module.

NestJS modules are the backbone of the framework's robust dependency injection mechanism. Dependency injection is an application development pattern that aims to decouple the dependencies between 2 classes or modules.

Instead of having strictly defined dependencies for each class, you can use an interface to specify a sort of contract of how your dependency should behave while having no literal definition of how it should run. Finally, a decouple architecture allows for versatile applications and creates a plug-and-play behavior for each module in the application.

## NestJS module state management
By default, NestJS module are singletons, meaning you only need to initiate a module once. While creating singletons for every module can seem excessive from an engineering point of view, NestJS initializes singletons on a component level.

### Module scopes in NestJS
In NestJS, there are three injection scopes of modules:
1. Request-level modules
2. Component-level modules or transient modules
3. Shared application-level modules

By default, most NestJS modules are application-level modules, also known as globally shared modules. But, not every module can be a global module. Some of them need to remain transient or request-level modules. 

For example, if you need an application-level, read-only module, your best bet is to use globally shared module. Data stored within the module won't change often, so it can be deferred as an application-level singleton to converse memory and create a globally accessible class. Module with the `@Global` decorator remove redundancy in both code and component levels since you don't need to reinitialize the module.

To better understand state preservation on a modular level, if you have a constant within a module with either a transient or request scope, it will be an immutable variable until the module destroys it on garbage collection. However, when using a global module spanning the entire application, it will be destroyed only at the end of an application's lifetime.

## Preventing data racing conditions when using singletons
Another thing to be cautious about when using singletons is the data race problem. Node.js is not immune to data racing conditions, and neither is NestJs.

Data race conditions occur when two separate processes try to update the same block of data simultaneously. Because the objects are accessible globally, simultaneous data execution may result in lost data points on execution.

Best practice for avoiding data race conditions involves creating a global read-only module and being more deliberate on each module's injection scopes. Global modules are the most susceptible to data race conditions, and using global modules to communicate or manage states between components will result in an anti-pattern.

But, why can't the same be said of transient component-level modules? At the component level, the encapsulation barriers only extend to the component's needs. Each transient module provider will have a dedicated instance. The separation of concerns at the component level is usually more granular, making it more predictable than in large-scale applications. The same can be said for the request-level singletons, albeit on a smaller scale.

## NestJs modules injection scopes
In summary, there are three injection scopes of modules in NestJS:
1. Request-level modules
2. Component-level modules: Transient
3. Shared application-level modules: Global

Each has its benefits and disadvantages, with data racing being the most common problem for global modules. Most global modules should be read-only, and NestJS will only set the original state once during initialization.

Component-level modules have more nuances. More specifically, you can use them for state management on a smaller scale because of their predictability. The granular encapsulation that singletons provide on a component level makes it a perfect choice for component-level state management.

Keep in mind that the data race condition is only limited to the state of each independent module. Modifying data in external applications like databases should not be a problem since databases have their own data race solution.

## Dynamic modules in NestJS
Default NestJS modules are static and are unconfigurable. Configurable model builders are essentially dynamic module factories that can churn out different modules based on the variables passed on initialization.

Before you get started with a configurable module, you need to understand the basics of dynamic modules. Their use cases typically revolve around creating non-static modules that can receive parameters from external APIs to change how the module behaves, specifically, how each module processes data.

For example, let’s imagine that you create a module for querying data from databases, but you don’t want to hardcode it for specific database providers. How do you solve the problem?

First, you need to create a module that has a configuration function. The configuration function will have a database provider interface as a parameter, which has all the essential functions an application needs to connect and query a database. Because you use an interface as a parameter, you can inject different database providers as long as the provider extends the interface.

The underlying business logic will still be the same, but the database providers will change according to the one you supplied on initialization. Therefore, your module will no longer be static and will instead be dynamic. This is essentially why all configurable modules are dynamic modules under the hood.

