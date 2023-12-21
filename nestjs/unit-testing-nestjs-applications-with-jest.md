# Unit testing NestJS applications with Jest

Usually, our functions internally make use of other functions. For example, there might be a method called `userService.createUser()`, which would internally call `userRepository.create()` to create the user entity and `userRepository.save()` to save it in the database of choice. In this case, testing the output would be silly at best and wouldn't really give us any confidence in whether the code is working as expected.

Unit tests are only interested in the logic that the function itself contains, not the external one.

So in this example, the test would check whether userRepository.create() has been called and, if so, with what arguments. Isolating the logic from all the external dependencies is called mocking. It means replacing all the specific implementations with fake ones that are monitored by the testing environment. This way, tracking the calls that have been made to a certain external method or overriding its return value is a breeze.

### What makes unit testing unique

The thing about unit tests is that they shouldn't be dependent on the environment in which they are being run, and they are supposed to be fast.

End-to-end (E2E) tests would be an example of those that are dependent on their environment. In E2E tests, we test an application as a whole, so we provide a test database and prepare the expected environment. But it is slow and needs a special setup, and thus, it is not really feasible to be run during development, which is when we usually run unit tests. There should be a specialized setup on a CI/CD service that would take care of running E2E tests.

Controlling external dependencies has one very important downside, though. In the test, we override the value that would normally be returned by the dependency. If, for whatever reason, the external dependency changes and now returns a `{ token, user }` object instead of a `User` object, the tests would still pass because it would still return a User object in our test environment.

That is why unit testing is only one of many kinds of tests. It ensures that the function has the expected behavior when we control all the external dependencies. There are also, for example, integration tests and contract tests that can assure us the contracts (expected input/output values of the external dependencies) haven’t changed.

## How NestKS helps us write unit tests

### 1. Dependency injection

Nest forces us to write more easily testable code through its built-in dependency injection — a design pattern that states that a central authority is taking care of creating and supplying dependencies, while the classes simply assume that the dependency will be provided instead of creating it themselves. So, instead of writing:

```ts
constructor() {
  this.a = new A();
}
```

we would write:

```ts
constructor(private readonly a: A) {}
```

The latter syntax is possible due to the JS Metadata Reflection API that was proposed not so long ago.

Dependency injection is usuallly based on interfaces rather than concrete classes. In Typescript, however, interfaces (and types in general) are only available during compile time rather than runtime, and thus can't be relied upon afterwards. So instead of using interfaces, in Nest it is common to use class-based injection.

### 2. The testing module

With dependency injection, replacing our dependencies become trivial. But with unit tests, we would rather not recreate the whole application for each test. Instead, we would like to create the bare minimum setup. Thankfully, Nest provides us with the `@nestjs/testing` package. The package enables us to create a Nest module, like we normally would, by declaring only the dependencies used in the tests.

```ts
const module: TestingModule = await Test.createTestingModule({
  providers: [
    PlaylistService,
    {
      provide: getRepositoryToken(Playlist),
      useClass: PlaylistRepositoryFake,
    },
  ],
}).compile();

playlistService = module.get(PlaylistService);
playlistRepository = module.get(getRepositoryToken(Playlist));
```

## The real-world example

We are going to focus on the PlaylistService class that is responsible for creating, finding, updating, and removing playlists.

```ts
@Injectable()
export class PlaylistService {
    public constructor(
        @InjectRepository(Playlist) private readonly playlistRepo: Repository<Playlist>
    ) {}

    public async findOneByIdOrThrow(id: string): Promise<Playlist> {
        const playlist = await this.playlistRepo.findOne({
            id
        })
        if(!playlist) {
            throw new NotFoundException('No playlist found.');
        }
        return playlist;
    }

    public async createOne(createPlaylistData: CreatePlaylistData): Promise<Playlist> {
        const { title, description }  = createPlaylistData;
        const playlist = this.playlistRepo.create({
            title,
            description,
        });
        const createdPlaylist = await this.playlistRepo.save(playlist)
        return createdPlaylist;
    }

    public async removeOne(removePlaylistData: RemovePlaylistData,): Promise<void> {
        const playlist = await this.findOneByIdOrThrow(id);
        await this.playlistRepo.remove([playlist])
        return null;
    }

    public async updateOne(updatePlaylistData: UpdatePlaylistData,): Promise<Playlist> {
        const { id, ...updateData } = updatePlaylistData;
        const existingPlaylist = await this.findOneByIdOrThrow(id);

        const playlist = this.playlistRepo.create({
            ...existingPlaylist,
            ...updateData
        })
        const updatedPlaylist = await this.playlistRepo.save(playlist)
        return updatedPlaylist;
    }
}
```

### Writing tests
```ts
describe('PlaylistService', () => {
    let playlistService: PlaylistService;
    let playlistRepo: Repository<Playlist>;

    beforeEach(async () => {
        const module: TestingModule = await Test.createTestingModule({
            providers: [
                PlaylistService,
                {
                    provide: getRepositoryToken(Playlist),
                    useClass: PlaylistRepositoryFake
                }
            ]
        }).compile();

        playlistService = module.get(PlaylistService)
        playlistRepo = module.get(getRepositoryToken(Playlist))
    })
})
```
Since we don't want to create an actual database connection, we are overriding the `playlistRepo` with `PlaylistRepositoryFake`. The `getRepositoryToken` function lets us get the injection token of the repository.

### Providing a fake
`PlaylistRepositoryFake` is a class that declares the same methods as `PlaylistRepository`, only all the methods are empty and have no behavior.
```ts
export class PlaylistRepositoryFake {
    public create(): void {};
    public async save(): Promise<void> {};
    public async remove(): Promise<void> {};
    public async findOne(): Promise<void> {};
}
```

The fakes have 2 important roles:

#### 1. Expecting certain conditions to be met during the creation of a class instance
The first reason to use fakes is especially important in our example: a class, playlistRepository, that has some specific logic executed during the creation of an instance.

The repository, when created, expects the database module to be imported (in the module context) and the connection to the database to be open. Since we are not importing the database module and there is no database running at all, the repository would throw an error during creation of the module.

By using the whole other class (the fake), all the logic related to the database module is gone. It is simple object that does nothing.

#### 2. Monitoring the calls to the external dependencies
In order to check that a method actually made use of an external dependency, we have to somehow register that the call has been made. For example, we would like to be sure that the .createOne() method actually called playlistRepository.save() and saved the entity in the database.

In the testing module setup, we save the references to each created instance, so playlistRepository and playlistService each have references to the objects instantiated by the testing module.
```ts
playlistService = module.get(PlaylistService);
playlistRepository = module.get(getRepositoryToken(Playlist));
```

Since objects in Javascript are passed around by reference, we can easily override a specific method, like:
```ts
let calledTimes = 0

playlistRepo.save = () => {
    calledTimes += 1;
}
```
By doing so by hand would be tiresome and error-prone. Jest provides us with a better way to do that: `spies`

Spies are defined in the following manner:
```ts
const playlistRepositorySaveSpy = jest
    .spyOn(playlistRepostiroy, 'save')
    .mockResolvedValue(savedPlaylist)
```

This spy does 2 things: it overrides both the `.save()` method of `playlistRepository` and provides an API for developers to choose what should be returned instead. In this snippet, we use `.mockResolvedValue()` as opposed to `.mockReturnValue()` due to the original implementation returning a promise.

By keeping a reference to the spy (assigning it to a variable), we can later on, in the test, do the following:
```ts
expect(playlistRepositorySaveSpy).toBeCalledWith(createdPlaylistEntity);
```

Although we are overriding the behavior of a method, Jest's spies still require the provided object to have said property. So if we provided a simple {} empty object, Jest would throw the following error:
```bash
Cannot spy the updateOne property because it is not a function; undefined given instead
```

### Testing the .createOne() method
the .createOne() method implementation:
```ts
public async createOne(createPlaylistData: CreatePlaylistData): Promise<Playlist> {
    const { title } = createPlaylistData;
    if(!title) {
        throw new BadRequestException('Title is required.');
    }

    const playlist = this.playlistRepo.create({
        title,
    })
    const createdPlaylist = await this.playlistRepo.save(playlist);
    
    return createdPlaylist;
}
```
As a parameter, the method takes an object that holds the title of the playlist. If the title isn't there, the method throws an error.
```ts
describe('creating a playlist', () => {
    it('throw an error when no title is provided', async () => {
        expect.assertions(2)

        try {
            await playlistService.createOne({ title: ''});
        } catch(e) {
            expect(e).toBeInstanceOf(BadRequestException);
            expect(e.message).toBe('Title is required.');
        }
    })
})
```
An example of a matcher would be the `.toEqual()` method that checks whether two objects are the same. It does so by comparing their keys and values recurrently, whereas Jest’s `.toBe()` method would simply check the strict equality using the === operator.

```ts
describe('creating a playlist', () => {
    it('throws an error when no title is provided', async () => {
        // ...
    });

    it('calls the repository with correct parameters', async () => {
        const title = faker.lorem.sentence();
        const createPlaylistData: CreatePlaylistData = {
            title,
        };
        const createdPlaylistEntity = Playlist.of(createPlaylistData);
        const savedPlaylist = Playlist.of({
            id: faker.random.uuid(),
            createdAt: new Date(),
            updatedAt: new Date(),
            title
        })
        const playlistRepositorySaveSpy = jest
            .spyOn(playlistRepository, 'save')
            .mockResolvedValue(savedPlaylist);
        const playlistRepositoryCreateSpy = jest
            .spyOn(playlistRepository, 'create')
            .mockReturnValue(createdPlaylistEntity)
        
        const result = await playlistService.createOne(createPlaylistData);
        expect(playlistRepositoryCreateSpy).toBeCalledWith(createPlaylistData);
        expect(playlistRepositorySaveSpy).toBeCalledWith(createdPlaylistEntity);
        expect(result).toEqual(savedPlaylist);
    })
}
```
First, we set up all the fake data needed for the test. Then, we call the appropriate piece of code (in our case, the method) and we use Jest's `.expect()` function to assert that the results are those we expected.

Now, before we call the method will the fake arguments, we create the spies of which we mock (to track the calls) and stub (override the returned value).

In order to check the mocks have been called, we can use the `expect(mock).toHaveBeenCalledWith()` method and provide the arguments with which we expected the mocks to have been called.

### Testing the other methods
Tests for the three remaining methods — `.updateOne()` , `.findOne()`, and `.removeOne()` 
```ts
describe('updating a playlist', () => {
    it('calls the repository with correct parameters', async () => {
        const playlistId = faker.random.uuid();
        const title = faker.lorem.sentence();

        const updatePlaylistData: UpdatePlaylistData = {
            id: playlistId,
            title,
        }
        const existingPlaylist = Playlist.of({
            id: playlistId,
            createdAt: new Date(),
            updatedAt: new Date(),
            title: faker.lorem.word(),
        })

        const newPlaylistData = Playlist.of({
            ...existingPlaylist,
            title,
        })
        const savedPlaylist = Playlist.of({
            ...newPlaylistData,
        })

        const playlistServiceFindOneByIdOrThrowSpy = jest   
            .spyOn(playlistService, 'findOneByIdOrThrow')
            .mockResolvedValue(existingPlaylist)

        const playlistRepositoryCreateSpy = jest
            .spyOn(playlistRepository, 'create')
            .mockReturnValue(newPlayistData);
        const playlistRepositorySaveSpy = jest
            .spyOn(playlistRepository, 'save')
            .mockResolvedValue(savedPlaylist)

        const result = await playlistService.updateOne(updatePlaylistData)
        expect(playlistServiceFindOneByIdOrThrowSpy).toHaveBeenCalledWith(updatePlaylistData.id);
        expect(playlistRepositoryCreateSpy).toHaveBeenCalledWith({
            ...existingPlaylist,
            title
        });
        expect(playlistRepositorySaveSpy).toHaveBeenCalledWith(newPlaylistData);
        expect(result).toEqual(savedPlaylist);
    })
})

describe('remove a playlist', () => {
    it('calls the repository with correct parameters', async () => {
        const playlistId = faker.random.uuid();
        const removePlaylistData: RemovePlaylistData = {
            id: playlistId
        }
        const existingPlaylist = Playlist.of({
            id: playlistId,
            createdAt: new Date(),
            updatedAt: new Date(),
            title: faker.lorem.sentence()
        })
        const playlistServiceFindOneByIdOrThrowSpy = jest
            .spyOn(playlistService, 'findOneByIdOrThrow')
            .mockResolvedValue(existingPlaylist);
        const playlistRepositoryRemoveSpy = jest
            .spyOn(playlistRepository, 'remove')
            .mockResolvedValue(null);
        
        const result = await playlistService.removeOne(removePlaylistData)
        expect(playlistServiceFindOneByIdOrThrowSpy).toHaveBeenCalledWith(removePlaylistData.id);
        expect(playlistRepositoryRemoveSpy).toHaveBeenCalledWith([
            existingPlaylist
        ])
        expect(result).toBe(null);
    })
})

describe('finding a playlist', () => {
    it('throws an error when a playlist doesnt exist', async () => {
        const playlistId = faker.random.uuid();
        const playlistRepositoryFindOneSpy = jest
            .spyOn(playlistRepository,'findOne')
            .mockResolvedValue(null)

        expect.assertions(3)
        try {
            await playlistService.findOneByIdOrThrow(playlistId);
        } catch (e) {
            expect(e).toBeInstanceOf(NotFoundException);
            expect(e.message).toBe('No playlist found.');
        }

        expect(playlistRepositoryFindOneSpy).toHaveBeenCalledWith({
            id: playlistId,
        })
    })

    it('returns the found playlist', async () => {
        const playlistId = faker.random.uuid();
        const existingPlaylist = Playlist.of({
            id: playlistId,
            createdAt: new Date(),
            updatedAt: new Date(),
            title: faker.lorem.sentence(),
        });

        const playlistRepositoryFindOneSpy = jest
            .spyOn(playlistRepository, 'findOne')
            .mockResolvedValue(existingPlaylist);
        
        const result = await playlistService.findOneByIdOrThrow(playlistId);
        expect(result).toBe(existingPlaylist);
        expect(playlistRepositoryFindOneSpy).toHaveBeenCalledWith({
            id: playlistId
        })
    })
})
```
repo: https://github.com/maciejcieslar/nest-unit-tests/blob/master/src/app/playlist/playlist.service.spec.ts