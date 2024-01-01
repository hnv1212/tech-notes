# How to execute a function with a web worker on a different thread in Angular

Web workers document: https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API

With web workers, we are able to execute a script in a background thread and leave the main thread free for UI work. By default, Web Workers accept the file URL as an argument, but in our case, that is not acceptable because we are using TypeScript as the main language an we don't want to mix it with Javascript. The second problem is that scripts need to have a fixed URL, and because we use Webpack for bundling and concatenating files, having an unbundled file is not the best pattern.

The Worker class is a base for Service Worker and SharedWorker. SharedWorker is similar to Worker except it can be accessed from several different contexts including popup windows, iframes, etc. 

Code executed by a worker runs in a different context than the code that runs on the main thread. When we run code in a worker we can't manipulate DOM elements, use window objects, etc. The context in which workers run are called DedicatedWorkerGlobalScope, and is quite limited in term of what you can access and generally do.

Common use cases for workers include the use of pure functions that do heavy processing. Because we don't want them to destroy performances of our web application, we should move them to a worker thread.

Worker threads can communicate with main threads through messages with `postMessage` method. Communication can be bidirectional, meaning that worker threads and main threads can send messages to each other.

https://blog.logrocket.com/wp-content/uploads/2019/11/mainandworkerthread.png

Both the main thread and worker thread can listen on and send messages to each other.

Let's create a Inline Worker class which will accept a function as an argument, and run that function in another thread, like this:
```ts
import { Observable, Subject } from 'rxjs';

export class InlineWorker {
    private readonly worker: Worker;
    private onMessage = new Subject<MessageEvent>();
    private onError = new Subject<ErrorEvent>();

    constructor(func) {
        const WORKER_ENABLED = !!(Worker);

        if(WORKER_ENABLED) {
            const functionBody = func.toString().replace(/^[^{]*{\s*/, '').replace(/\s*}[^}]*$/, '');

            this.worker = new Worker(URL.createObjectURL(
                new Blob([ functionBody ], { type: 'text/javascript' })
            ));

            this.worker.onmessage = (data) => {
                this.onMessage.next(data);
            };

            this.worker.onerror = (data) => {
                this.onError.next(data);
            };

        } else {
            throw new Error('WebWorker is not enabled');
        }
    }

    postMessage(data) {
        this.worker.postMessage(data);
    }

    onmessage(): Observable<MessageEvent> {
        return this.onMessage.asObservable();
    }

    onerror(): Observable<ErrorEvent> {
        return this.onError.asObservable();
    }

    terminate() {
        if(this.worker) {
            this.worker.terminate();
        }
    }
}
```

The most important part of the code shown above is a class that converts a function to a string and creates `ObjectURL` which will be passed to a worker class through a constructor.

### How to use the InlineWorker class
Let's imagine that we have a function in Angular (like the class shown in the code block above), what we want to process in a background.

We are going to build an application that calculates how many prime numbers we have in range.

The main thread will send limit parameters to the worker thread, once the thread complete its job it will yeild results to a main thread and terminate the worker.

It is important to note that we can't use any methods, variables or functions defined outside of a callback function that has been passed to an InlineWorker.

If we need to pass arguments (`postMessage` function accept anything as parameters), we have to do that with `postMessage` method.

```ts
import { Component, OnInit } from '@angular/core';
import { InlineWorker } from './inlineworker.class';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
    
    result = 0;

    ngOnInit() {
        const worker = new InlineWorker(() => {
            // START OF WORKER THREAD CODE
            console.log('Start worker thread, wait for postMessage: ');

            const calculateCountOfPrimeNumbers = (limit) => {
                const isPrime = num => {
                    for(let i = 2; i < num; i++) {
                        if(num % i === 0) { return false; }
                    }
                    return num > 1;
                }

                let countPrimeNumbers = 0;

                while (limit >= 0) {
                    if(isPrime(limit)) { countPrimeNumber += 1; }
                    limit--;
                }

                // this is from DedicatedWorkerGlobalScope (because of that we have postMessage and onmessage methods)
                // and it can't see methods of this class
                // @ts-ignore
                this.postMessage({
                    primeNumbers: countPrimeNumbers
                });
            };

            // @ts-ignore
            this.onmessage = (evt) => {
                console.log('Calculation started: ' + new Date());
                calculateCountOfPrimeNumbers(evt.data.limit);
            };
            // END OF WORKER THREAD CODE
        });

        worker.postMessage({ limit: 300000});

        worker.onmessage().subscribe((data) => {
            console.log('Calculation done: ', new Date() + ' ' + data.data);
            this.result = data.data.primeNumbers;
            worker.terminate();
        });

        worker.onerror().subscribe((data) => {
            console.log(data);
        });
    }
}
```

As we can see, we are passing an anonymous function as a parameter to an InlineWorker. The context of the passed function is isolated, meaning that we can't access anything outside of it. If we try that it will be undefined.

The flow of our application looks something like this:

https://blog.logrocket.com/wp-content/uploads/2019/11/appcomponent.png

We have to put @ts-ignore comment in front of postMessage and `onmessage` methods, as Typescript can't read definitions from the current context. In this case, Typescript is not that helpful.

The listener `onmessage` inside the callback function will listen for any messages passed to this worker, and in our case, it will call `calculateCountOfPrimeNumbers` with passed parameters to it.

Functions will do calculations and with `postMessage` method it will yield results to a listener on the main thread.

With:
```ts
worker.postMessage({ limit: 10000 });
```
We will trigger execution of a worker thread. As we wrote this example in Angular, we will use RxJS observables to pass and listen to data changes.

On the next line, we are subscribing to messages from a worker
```ts
worker.onmessage().subscribe((data) => {
 console.log(data.data);
 worker.terminate();
});
```

Simply, we are outputting a result to a console and then we terminate the worker, so it can't be used anymore. We can send multiple messages to a worker thread and receive multiple results, we are not locked for single execution as in the example above.

It is important that we subscribe to an `onerror` observable because it is the only way to see errors that happen in a worker thread.

## Demo
Here is the demo with worker implementation: https://angular-with-worker-logrocket.surge.sh/

And here is the demo without the worker: https://angular-without-worker-logrocket.surge.sh/ ( UI is blocked while computation is running )