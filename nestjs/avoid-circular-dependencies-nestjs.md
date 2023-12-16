# How to avoid circular dependencies in NestJS

### What is a circular dependency?
In programming, a circular dependency occurs when two or more modules (or classes), directly or indirectly depend on one other. Say A, B, C and D are four modules, an example of direct circular dependency is A -> B -> A. Module A depends on module B which in turn depends on A.

An example of indirect circular dependency is A -> B -> C -> A. Module A depends on B which doesn't depend on A directly, but later on in its dependency chain references A.

Note that the concept of circular dependency is not unique to NestJS, and in fact, the modules used here as example don’t even have to be NestJS modules. They simply represent the general idea of modules in programming, which refers to how we organize code.

### Avoid circular dependencies by refactoring
To break the cycle, we can extract the common features from both services into a new service that depends on both services. 

### Working around circular dependencies with forward references
Ideally, circular dependencies should be avoided, but in cases where that’s not possible, Nest provides a way to work around them.

A forward reference allows Nest to reference classes that have not yet been defined by using the `forwardRef()` utility function. We have to use this function on both sides of circular reference.

`UserService`
```typescript
import { forwardRef, Inject, Injectable } from '@nestjs/common';
import { FileService } from '../file-service/file.service';

@Injectable()
export class UserService {
    constructor(
        @Inject(forwardRef(() => FileService))
        private readonly fileService: FileService,
    ) {}

    public async getUserById(userId: string) {
        // ...
    }

    public async addFile(userId: string, pictureId: string) {
        const picture = await this.fileService.getById(pictureId);
        // update user with the picture url
        return { id: userId, name: 'Sam', profilePictureUrl: picture.url };
    }
}
```

`FileService`
```typescript
import { forwardRef, Inject, Injectable } from '@nestjs/common';
import { UserService } from '../user/user.service';
import { File } from './interfaces/file.interface';

@Injectable() 
export class FileService {
    constructor(
        @Inject(forwardRef(() => UserService))
        private readonly userService: UserService,
    ) {}

    public getById(pictureId: string): File {
        // ...
    }

    public async getUserProfilePicture(userId: string): Promise<File> {
        const user = await this.userService.getUserById(userId);
        return this.getById(user.id)
    }
}
```
With this forward reference, the code will compile without errors.

### Forward references for modules
The `forwardRef()` utility function can also be used to resolve circular dependencies between modules, but it must be used on both sides of the modules' association. For example, one could do the following on one side of a circular module reference:
```typescript
@Module({
    imports: [forwardRef(() => SecondCircularModule)]
})
export class FirstCircularModule {}
```