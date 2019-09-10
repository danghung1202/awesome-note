# RxJs Operators

Some my understanding regard to Rxjs Operators, how it work and how to apply to resolve some problems in real world. All descriptions were based on my experiences when working on LegoHub project

- [RxJs Operators](#rxjs-operators)
  - [combineAll](#combineall)
  - [combineLatest](#combinelatest)
  - [concatMap](#concatmap)
  - [Difference between Rxjs Subject and Observable](#difference-between-rxjs-subject-and-observable)

## combineAll
```javascript
    // https://stackoverflow.com/questions/40533016/angular2-chain-http-requests-with-concat
    // Mocks an http call that takes 1 second to complete
    function fakeRequest(id) {
      console.log('doing the request for id ' + id);
      return Rx.Observable.of(id).delay(1000);
    }
    
    let items = ["1", "2", "3"];
    
    // create an observable from the array that emits every item one after one
    Rx.Observable.from(items)
       // use concatMap, this operator accepts a function that returns an observable.
       // It will subscribe to this observable and not take on any next values
       // untill that observable completes.
       // So it will get '1' and perform the request. As soon as this one completes
       // it will handle the next value, which will be '2' and so on.
      .concatMap(
        (val) => {
          return fakeRequest(val)
       })
       // Use combine all to make sure the subscription below isn't call untill
       // every element in the original array is handled.
      .combineAll()
       // Get back an array with the results from every call, just take the last one
       // if you really need it
      .subscribe((val) => console.log(val));
```

## combineLatest

You have many Observable, when each one of these emit value, we want to combine all lastest values from these streams

```javascript
    this.subSelectedFilter = Observable.combineLatest(this.selectedThemes, this.selectedSubthemes, this.selectedYears,
                (themes, subthemes, years) =>
                    ({
                        themes: themes.map(x => x.value).join(','),
                        subthemes: subthemes.map(x => x.value).join(','),
                        years: years.map(x => x.value).join(',')
                    }))
                .subscribe(result => {
                    this.params = result;
                });
```
## concatMap

I have one existing `Observable`, and I want to start a new Observable for each value and emit the values from each nested Observable in
order where the nested Observable is calculated for each value.

In real application, if you want to execute one request many time sequencely (one by one) and result of each request is independent and has order
you can use `concatMap()`
For more detail: 

https://stackoverflow.com/questions/39566268/angular-2-rxjs-how-return-stream-of-objects-fetched-with-several-subsequent/39578646
```javascript
    //Example
    //Has order
    Observable.from([param1, param2, ..., param_n])
                .concatMap(param => excute_async_request(param))
                .map(response => {
                  //do something with response
                })
                .subscribe(response => console.log(response)) 
                //console log response right after each request is complete
 ```   
In fact there is the case you want to combine all results of all requests after all is complete
In that case using `combineAll()`, look at combineAll section in this gist

In contrast, the `mergeMap()` will excute requests in random order

Has no order

```javascript
    Observable.from([param1, param2, ..., param_n])
                .mergeMap(param => excute_async_request(param))
                .map(response => {
                  //do something with response
                })
                .subscribe()
```
## Find Differences between Unicast and Multicast

What is a Subject? An RxJS Subject is a special type of Observable that allows values to be multicasted to many Observers. 

While plain Observables are unicast (each subscribed Observer owns an independent execution of the Observable), 

Subjects are multicast.
All subscribers to a subject share the same execution of the subject. i.e. when a subject produces data, all of its subscribers
will receive the same data. 

This behavior is different from observables, where each subscription causes an independent
execution of the observable.

If I have one existing Observable, and I want to share a subscription between multiple subscribers 

* Using as BehaviorSubject's . --> using publishBehavior()
* Using as Subject --> using share() or publish()
* Using as ReplaySubject --> using publishReplay()

Example using share()

```javascript
    const stream$ = Observable.fromEvent(this.searchInput.nativeElement, 'keydown').share()
    const subscription1 = stream$.filter().map().subscribe(x => do something 1);
    const subscription2 = stream$.filter().map().subscribe(x => do something 2);
    const subscription3 = stream$.filter().map().subscribe(x => do something 3);
```
## forkJoin

I have some Observables to combine together as one Observable, and I want to be notified when all of them have completed.

In real application, if there are two or many asyn requests (parallel) and we want to do something after all these requests is completed, we can use forkJoin for this case

Example
```javascript
    
    Observable.forkJoin([
              Async_Request_0,
              Async_Request_1,
              Async_Request_2,
              ...
              Async_Request_n,
            .map(results => {
                let r0 = results[0];//result of async request 0
                let r1 = results[1];//result of async request 1
                let r2 = results[2];//result of async request 2
                
                ...
                let rn = results[n];//result of async request n
                //do something with r0, r1, r2, ..., rn
                return something;
            })
            .catch(error => {
                return this.handleError(error);
            });
```

## RxJs_Cache

Using cache in Observable

```javascript
    private _themes: any = null;
    getThemes(): Observable<Theme[]> {
      if (!this._themes) {
          this._themes = this.http.get('url')
              .map(res => res.json())
              .publishReplay(1)
              .refCount()
              .catch(error => {
                  return this.handleError(error);
              });
      }
      return this._themes;
    }
```

## Subject and BehaviorSubject

For `Subject` object, all values was sent before subscriber is not delivered to that subscriber
```javascript
    const subject = new Rx.Subject();
    subject.next(1);
    subject.subscribe(x => console.log('a',x));
    subject.next(2);
    subject.next(3);
    //output: 2,3
```   
For `BehaviorSubject` object, one previous value still delivery to subscriber although this subscriber is declare later
```javascript
    const bsubject = new Rx.BehaviorSubject();
    bsubject.next(1);
    bsubject.subscribe(x => console.log(x));
    bsubject.next(2);
    bsubject.next(3);
    //output: 1,2,3
```
Incase you want to more one previous values are delivered to late subscriber, using `ReplaySubject`.

```javascript
    const replay = new ReplaySubject(number_of_previous_item_will_be_delivery)
```
## switchMap

In real application, example the auto suggestion in search box, each user type keyword,there is one async request to get result for that keyword. It means each user type one character, there is new keyword and there is one async request. The problem is user type very fast so there are many requests since user search. So we want to only last result of last request is processed.

Using `swichMap()` can resolve this.
```javascript   
    Observable.from([param1, param2, ..., param_n])
                // only request take parma_n (last param) is processed, the previous requests is abort
                .switchMap(param => excute_async_request(param)) 
                .map(response => {
                  //do something with response
                })
                .subscribe(response => console.log(response))
```

## Access property without using subscribe

```javascript
    //Suppose you have a Observable object
    interface GameModel
    {
      invaild: boolean
    }
    
    let game$ = Obvesable<GameModel>;
    //using scan to access "invalid" property of GameModel via game$ without subscribe
    invalid() {
      return this.game$
        .scan((accum: boolean, current: any) => {
          return (current && current.get('invalid')) || accum;
        }, false);
    }
```


## Difference between Rxjs Subject and Observable

This shows how subscribe calls are not shared among multiple Observers of the same Observable. When calling observable.subscribe with an Observer, the function subscribe in Observable.create(function subscribe(observer) {...}) is run for that given Observer. Each call to observable.subscribe triggers its own independent setup for that given Observer.

From <http://reactivex.io/rxjs/manual/overview.html#observer> 

What is a Subject? An RxJS Subject is a special type of Observable that allows values to be multicasted to many Observers. While plain Observables are unicast (each subscribed Observer owns an independent execution of the Observable), Subjects are multicast.

From <http://reactivex.io/rxjs/manual/overview.html#observer> 

All sub­scribers to a sub­ject share the same exe­cu­tion of the sub­ject. i.e. when a sub­ject pro­duces data, all of its sub­scribers will receive the same data. This behav­ior is dif­fer­ent from observ­ables, where each sub­scrip­tion causes an inde­pen­dent exe­cu­tion of the observable.

From <http://javascript.tutorialhorizon.com/2017/03/23/rxjs-subject-vs-observable/> 

An Observable is A Data Producer while An Subject can act as both: A Data Producer & A Data Consumer
