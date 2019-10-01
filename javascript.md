# Javascript Notes

## JavaScript Module Pattern Basics

The Module Pattern is one of the most common design patterns used in JavaScript and for good reason. 

The module pattern is easy to use and creates encapsulation of our code. 

Modules are commonly used as singleton style objects where only one instance exists. The Module Pattern is great for services and testing/TDD. 

**Creating a module**

```javascript
(function() {
  'use strict';
  // Your code here
  // All function and variables are scoped to this function
}());
```

This pattern is well known as a Immediately Invoked Function Expression or IIFE. 

**Exporting our module**

```javascript
    var myModule = (function(globalVariable) {
    'use strict';

    var _privateProperty = 'Hello World';
    var publicProperty = 'I am a public property';

    function _privateMethod() {
        console.log(_privateProperty);
    }

    function publicMethod() {
        _privateMethod();
    }
        
    return {
        publicMethod: publicMethod,
        publicProperty: publicProperty
    };
    }(globalVariable));
    
    myModule.publicMethod();    		        // outputs 'Hello World'   
    console.log(myModule.publicProperty);       // outputs 'I am a public property'
    console.log(myModule._privateProperty);     // is undefined protected by the module closure
    myModule._privateMethod();                  // is TypeError protected by the module closure

```

**Source**

[JavaScript Module Pattern Basics](https://coryrylan.com/blog/javascript-module-pattern-basics)

## Thumb Rules for async-await and promise in javascript

> Innovation: Async code -> Callback -> Promises -> Async/Await keyword

```mermaid
    graph TD
    A(Async code)-->|get value|B(using Callback)
    B-->|too callbacks <br> callback hell|C(using Promises)
    C-->|beautiful code <br> look like sync code|D(using Async/Await keyword)
    D-->R(syntax and structure of your code <br> using async functions is much more like <br> using standard synchronous functions.)

```

> The purpose of `async/await` functions is to simplify the behavior of using promises synchronously and to perform some behavior on a group of Promises. Just as `Promises` are similar to structured callbacks, async/await is similar to combining generators and promises.

**Promise**

1. Use promises whenever you are using asynchronous or blocking code.
2. `resolve` maps to then and `reject` maps to catch for all practical purposes.
3. Make sure to write both `.catch` and `.then` methods for all the promises.

**async-await**

1. `async` functions return a promise
2. `async` functions use an implicit `Promise` to return results. Even if you don’t return a promise explicitly, the `async` function makes sure that your code is passed through a promise.
3. `await` blocks the code execution within the async function, of which it
4. Be extra careful when using await within loops and iterators. You might fall into the trap of writing sequentially-executing code when it could have been easily done in parallel.
5. `await` only blocks the code execution within the async function. It only makes sure that the next line is executed when the promise resolves. So, if an asynchronous activity has already started, await will not have any effect on it.

**Should I Use Promises or async-await**
1. The async function returns a promise. The converse is also true. Every function that returns a promise can be considered as async function.
2. await blocks the execution of the code within the async function in which it is located.
3. If two functions can be run in parallel, create two different async functions and then run them in parallel.
4. To run promises in parallel, create an array of promises and then use `Promise.all(promisesArray)`.
5. Every time you use await remember that you are writing blocking code.
6. If your code contains blocking code, it is better to make it an async function. By doing this, you are making sure that somebody else can use your function asynchronously.
7. By making async functions out of blocking code, you are enabling the user (who will call your function) to decide on the level of asynchronicity they want.

> It’s not necessary to use promise.all, with async/await. The equivalent is something like:

> `const allOfThem = [await promise1, await promise2, …];`

> Note: the above code assumes that you have already assigned the promises to a constant or variable (e.g. promise1) and did not call an async function within the array literal (e.g. `[await getPromise(), await getP2()])`, as the second item would not kick off the code to retrieve anything until the first was done.

Source:

[JavaScript: Promises or async-await](https://medium.com/better-programming/should-i-use-promises-or-async-await-126ab5c98789)

**async functions return a promise**

Now that you’ve seen the benefit of Async/Await, let’s discuss some smaller details that are important to know. First, anytime you add async to a function, that function is going to implicitly return a promise.

```javascript
async function getPromise(){}

const promise = getPromise()
```

Even though getPromise is literally empty, it’ll still return a promise since it was an async function.

If the async function returns a value, that value will also get wrapped in a promise. That means you’ll have to use .then to access it.

```javascript
async function add (x, y) {
  return x + y
}

add(2,3).then((result) => {
  console.log(result) // 5
})
```

Let try with Promise, async-await, how await block the code to the next line is executed



```javascript

function resolveWithPromise() {
  return new Promise(resolve => {
      resolve('resolved of promises');
  });
}

async function resolveWithAsync() {
  return "resolved of async";
}

async function asyncCall() {
  console.log('calling');
  
  //await console 1
  console.log('block before promise/async call with await ' + await resolveWithPromise())
  
  resolveWithPromise()
  .then(a=>{
    console.log(a);
  });

  //await console 2
  console.log('block before promise/async call with await ' + await resolveWithPromise())

  resolveWithAsync()
  .then(a=>{
    console.log(a);
  });
  
  //await console 3
  console.log('block after promise/async call with await ' + await resolveWithAsync())

  console.log('the code run after await');
  // expected output: 'resolved'
}

asyncCall();

```

> try comment out one by one the the "await console 1", "await console 2" and "await console 3" to see the surprising result

> run on https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function

> need to be understand the single thread - event loop in javascript runtime

> Note: Async/await is slightly slower due to its synchronous nature. You should be careful when using it multiple times in a row as the await keyword stops the execution of all the code after it — exactly as it would be in synchronous code.

## Dom Api

But it’s still verbose!

If you are still worried about extra keystrokes you can alias both methods:
```javascript
    const $ = document.querySelector.bind(document);
    $('#container');

    const $$ = document.querySelectorAll.bind(document);
    $$('p');
```

### MutationObserver 

You can observe changes to any DOM node through the MutationObserver interface. This includes text changes, nodes being added to or removed from the observed node or changes to the node’s attributes.

The MutationObserver is an incredibly powerful API to observe virtually any change that occurs on a DOM element and its child nodes.

A new MutationObserver is created by calling its constructor with a callback function. This callback will be called whenever a change occurs on the observed node:

    const observer = new MutationObserver(callback);


**Reference**

[Using the DOM like a Pro](https://itnext.io/using-the-dom-like-a-pro-163a6c552eba)

