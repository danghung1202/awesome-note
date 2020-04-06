# Daily Notes - Uncategorized topic

## Simple Real Time App Architecture

You can create a simple real time app in less than one hour. 

Using something like this:

1. Using Firebase's FireStore to store data. The data will be automatically updated after any change on database
2. Using Firebase's Cloud Function to call api in somewhere. Don't care about server, host...
3. Using NodeJs, Angular, React, whatever you prefer to create simple client app

![FirebaseRealTimeApp](images/FirebaseRealtimeApp.jpg)


> *Instead using Google Firebase, you can use Microsoft Azure (Notification Hub, CosmosDB, Azure Function)*

*Reference*

[Real Time scraping using Puppeteer](https://medium.com/stink-studios/real-time-scraping-using-puppeteer-40495b5fc270)


## Best practices for a clean and performance Angular application

https://www.freecodecamp.org/news/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f/

### Clean up subscriptions

When subscribing to observables, always make sure you unsubscribe from them appropriately by using operators like take, takeUntil, etc.

**Why?**

Failing to unsubscribe from observables will lead to unwanted memory leaks as the observable stream is left open, potentially even after a component has been destroyed / the user has navigated to another page.

Even better, make a lint rule for detecting observables that are not unsubscribed.

**Before**

```typescript
iAmAnObservable
    .pipe(
       map(value => value.item)     
     )
    .subscribe(item => this.textToDisplay = item);
```

**After**

Using `takeUntil` when you want to listen to the changes until another observable emits a value:

```typescript
private _destroyed$ = new Subject();

public ngOnInit (): void {
    iAmAnObservable
    .pipe(
       map(value => value.item),
       take(1),
       // We want to listen to iAmAnObservable until the component is destroyed,
       takeUntil(this._destroyed$)
     )
    .subscribe(item => this.textToDisplay = item);
}

public ngOnDestroy (): void {
    this._destroyed$.next();
    this._destroyed$.complete();
}

```

Using a private subject like this is a pattern to manage unsubscribing many observables in the component.

> Note the usage of `takeUntil` with take here. This is to avoid memory leaks caused when the subscription hasn’t received a value before the component got destroyed. Without `takeUntil` here, the subscription would still hang around until it gets the first value, but since the component has already gotten destroyed, it will never get a value — leading to a memory leak.


## How difference between `Rather than` and `Other than`?
Rather than is like saying instead of

    The food was cooked with flame rather than electric heat

Other than is like saying except or except for

    He said he doesn't own any property other than his home

I'm sorry I'm not better with technical definitions...😬


## Web Performance Tool

### Basic Terms

TTFB - Time To First Byte
First Paint (Start Render)
Contentful Paint (First Meaningful Paint)
Dom Init
Dom Loaded
Speed Index
Page Load
Fully Loaded


### Some tips to improve


## Angular - Inject all Services that implement some Interface

Using `[InjectionToken](https://angular.io/api/core/InjectionToken)`

```typescript
var myInterfaceToken new InjectionToken<MyInterface>('MyInterface');
```

```typescript

// import `myInterfaceToken` to make it available in this file

@NgModule({
  providers: [ 
    { provide: myInterfaceToken, useClass: Service1, multi:true },
    { provide: myInterfaceToken, useClass: Service2, multi:true },
  ],
  boostrap: [AppComponent],
)
class AppComponent {}
```

```typescript
// import `myInterfaceToken` to make it available in this file

export class CollectorService {
    constructor(@Inject(myInterfaceToken) services:MyInterface[]) {
        services.forEach(s => s.foo());
    }
}
```

Source:

[Inject all Services that implement some Interface](https://stackoverflow.com/questions/35916542/inject-all-services-that-implement-some-interface/35916788#35916788)

## Issues Highlighted during third party penetration tests

Some security issues are raised by the Maginus customer:

https://maginus.atlassian.net/browse/NRAL-2500


![](images/security_issues.png)

## NPM packages are not building on Windows 8.1/10 - related to VsBuild C++

[https://github.com/nodejs/node-gyp/issues/629](https://github.com/nodejs/node-gyp/issues/629)

Applying for install some packages:
bson
bufferutil
kerberos
node-sass
sqlite3
phantomjs
utf-8-validate

Especially, the node-sass package
