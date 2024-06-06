---
title: "Event Loop, Promises, Task Queues and Callstack"
date: 2024-06-06T19:00:30+08:00
tags:
 - javascript
 - Promise
 - Event Loop
 - Task Queue
draft: false
---
Recently reading Event Loop, Promises, Task Queues, Call Stack, and Web APIs in Js. Just jotting down some notes.  

## JavaScript
Js is single-threaded, when handling long-running tasks it could block the main thread, causing the UI to freeze. To avoid this, Js uses the following mechanisms to handle async tasks.

## Call Stack
When a function is called, it is pushed onto the call stack. When the function returns, it is popped out of the stack. 

## Web APIs
Web APIs are provided by the browser, such as `setTimeout`, `fetch`, `DOM`, getting your location info, etc. They provide a way to handle async tasks. When an async task is called, the task is pushed to the Web API, and the call stack continues to execute the next function. This kind of async tasks will generate a callback function, the calllback will be pushed to the task queue, NOT the call stack!

## Event Loop
The event loop constantly checks the call stack whether it is empty. When it's empty, it will push the first task in the task queue (callback queue) to the call stack. Callback functions won't be immediately executed after the async task is done, they should wait for the call stack to be empty. This can prevent unexected disruptions to the call stack.

e.g. 
```js
console.log('1');
setTimeout(() => {
  console.log('2');
}, 100);
console.log('3');
```

The output will be `1 3 2`. The 100ms timeout doesn't really mean the `console.log(2)` will be executed after 100ms, it means the callback will be pushed to the task queue after 100ms, so if the call stack is filled with other functions, it might take longer to execute the callback.

## (Macro)Task Queue and Microtask Queue
As described above, Task Queue is for callback functions generated by Web APIs, while Microtask Queue is for callback functions generated by `Promise`. Microtask Queue has higher priority than Task Queue.

While call stack is empty, the Event Loop will move on to the task queue only when the Microtask Queue is empty. Starvation may occur if the Microtask Queue is always filled with tasks(like infinite loop)

## Promise
`Promise` is a way to handle async tasks in Js, such as `fetch`. It has 3 fields: `state`, `result`, and `Reactions`.

In a `Promise` constructor, we pass a callback function with two parameters: `resolve` and `reject`.

```js
const promise = new Promise((resolve, reject) => {
    ...
})
.then((result) => {
    ...
})
.catch((error) => {
    ...
})
```

### state
A `Promise` has three states: `pending`, `fulfilled`, and `rejected`. If the `Promise` is resolved, it will be fulfilled, otherwise it will be rejected.

### result
The result of the callback function. If the callback function throws an error , the `Promise` will be rejected, this field will be set to the value we passed to `reject`. Similarly if the `Promise` is resolved this value will be set to the value we passed to `resolved`, and the `fullfilledReactions` will be enqueued to the Microtask Queue.  
BTW, if the state is `pending`, the `result` field will be `undefined`.

### Promise Reactions
#### PromiseFullfillReactions
Callback functions that will be executed when the `Promise` is fulfilled, pushed to the Microtask Queue. We can create this reaction by chaining the `Promise` with `.then()` containing a handler function.   

#### PromiseRejectReactions
Callback functions executed when the `Promise` is rejected, pushed to the Microtask Queue. We can create this reaction by chaining the `Promise` with `.catch()`.

---

example use:
```js
new Promise((resolve, reject) => {
    const fetchResult = fetch('https://example.com');
    if (fetchResult.status === 200) {
        resolve(fetchResult);
    } else {
        reject('Error');
    }
})
.then((result) => {
    console.log(result);
})
.catch((error) => {
    console.log(error);
})
```

another example:
```js
console.log('1');  // pushed to Call Stack and print 1

setTimeout(() => {  // wait for 0ms, directly pushed to Task Queue
  console.log('2');
}, 0);

new Promise((resolve, reject) => {  // Promise object created, executer function pushed to Call Stack and executed immediately
    resolve('3');  // state set to fulfilled, result field set to '3'
}).then(result => console.log(result));  // fullfilledReactions pushed to Microtask Queue

console.log('4');  
// previous executor function is already popped from Call Stack, call stack is empty, 
// pushed this to Call Stack and print 4, stack is empty again
// check microtask queue first, print '3'
// check task queue, print '2'

// output: 1 4 3 2
```