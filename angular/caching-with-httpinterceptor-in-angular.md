# Caching with HttpInterceptor in Angular

HttpInterceptor is one of the most powerful features in Angular. You can use it to mock a backend so you can easily test Angular apps without the hassle of setting up a server.

## Why cache HTTP requests?
Caching HTTP requests helps to optimize your application. Imagine requesting a piece of data from the server every time a request is placed, even if the data has never changed over time. This would impact your app's performance due to the time it takes to process the data in the server and send it over when the data has never changed from previous requests.

To remove the time delay in processing the data in the server when it hasn't changed, we need to check whether the data in the server has changed. If the data has changed, we process new data from the server. If not, we skip the processing in the server and send the previous data. This is called caching.

In HTTP, not all requests are cached. POST, PUT, and DELETE requests are not cached because they change the data in the server.

## Why use HttpInterceptor?
HttpInterceptor are special services in Angular. HTTP requests are passed through them in the chain before the actual request is made to the server.

Put simply, HttpInterceptors intercept and handle HTTP requests. Typically, HttpInterceptors call `next.handle(transformedReq)` to transform outgoing requests before passing them to the next interceptor in the chain. In rare cases, interceptors handle requests themselves instead of delegating to the remainder of the chain.

## Using HttpInterceptor in Angular
We’ll create our HttpInterceptor so that whenever we place a GET request, the request will pass through the interceptors in the chain. Our interceptor will check the request to determine whether it has been cached. If yes, it will return the cached response. If not, it will pass the request along to the remainder of the chain to eventually make an actual server request. The interceptor will watch for the response when it receives the response and cache it so that any other request will return the cached response.

We’ll also provide a way to reset a cache. This is ideal because if the data processed by the GET API has been changed by POST, PUT, DELETE since the last request, and we’re still returning the cached data. We’ll be dealing with a stale data and our app will be displaying wrong results.

**Setting up HttpInterceptor**
All HttpInterceptor implement the `HttpInterceptor` interface:
```typescript
export interface HttpInterceptor {
    intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>>;
}
```

`HttpHandler` are responsible for calling the `intercept` method in the next HttpInterceptor in the chain and also passing in the next `HttpHandler` that will call the next interceptor.

Set up our `CacheInterceptor`:
```typescript
@Injectable()
class CacheInterceptor implements HttpInterceptor {
    private cache: Map<HttpRequest, HttpResponse> = new Map();

    intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
        if(req.method !== "GET") {
            return next.handle(req)
        }
        const cachedResponse: HttpResponse = this.cache.get(req);
        if(cachedResponse) {
            return of(cachedResponse.clone())
        } else {
            return next.handle(req).pipe(
                do(stateEvent => {
                    if(stateEvent instanceof HttpResponse) {
                        this.cache.set(req, stateEvent.clone())
                    }
                })
            ).share()
        }
    }
}
```
Since the request method is not a GET request, we pass it along the chain — no caching.

If it is a GET, we get the `cached` response from the cache map instance using the `Map#get` method passing the `req` as key. We are storing the request HttpRequest as a key and the response HttpResponse as the value in the map instance, `cache`.

The map will be structured like this:

| Key | Value |
| - | - |
| HttpRequest { url: "/api/dogs", ...} | HttpResponse { data: ["alsatians"], ...} |
| HttpRequest { url: "/api/dogs/name='bingo'", ...} | HttpResponse { data: [{name" "bingo", ...}], ...} |
| HttpRequest { url: "/api/cats", ...} | HttpResponse { data: ["serval"],...} |

A key is an HttpRequest instance and its corresponding value is an HttpResponse instance. We use its `get` and `set` methods to retrieve and store the HttpRequests and HttpResponses. So when we call `get` in `cache`, we know we'll get an HttpResponse.

The response is stored in `cachedResponse`. We check to make sure it's not null (i.e., we get a response). If yes, we clone the response and return it.

If we don't get a response from the cache, we know the request hasn't been cached before, so we let it pass and listen for the response. If we see one, we cache it using the `Map#set` method. The `req` becomes the key and the response becomes the value.

We need to watch for the stale data when caching. We need to know when the data has changed and make a server request to update the cache.

We can use different methods to achieve this. We can use the `If-Modified-Since` header, we can set our expiry date on the HttpRequest header, or we can set a flag on the header to detect when to make a full server request.

The CacheInterceptor has to check for the reset param in the header to determine when to rest the cache. Let’s add it to our CacheInterceptor implementation:
```typescript
@Injectable()
class CacheInterceptor implements HttpInterceptor {
    // ...

    intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
        // ...
        if(req.headers.get("reset")) {
            this.cache.delete(req)
        }

        const cachedResponse: HttpResponse = this.cache.get(req);
        // ...
    }
}
```

Lastly, we need to register our CacheInterceptor in the HTTP_INTERCEPTORS array token. Without it, our interceptor won't be in the interceptors chain and we can't cache requests.
```typescript
@NgModule({
    // ...
    providers: {
        provide: HTTP_INTERCEPTORS,
        useClass: cacheInterceptor,
        multi: true
    }
})
```

With this, our CacheInterceptor will pick all HTTP requests made in our Angular app.

