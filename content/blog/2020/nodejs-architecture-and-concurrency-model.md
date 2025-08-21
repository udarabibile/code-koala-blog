+++
author = "Udara Bibile"
authors = ["s"]
authorImage = "/img/udarabibile.png"
title = "NodeJS Architecture & Concurrency Model"
date = "2020-01-15"
description = "Single threaded JavaScript into asynchronous non-blocking I/O model using Event Loop"
tags = ["chown", "chmod"]
categories = ["linux", "os"]
images  = ["img/2020/linux-permissions-cmd.jpeg"]
type = "post"
aliases = ["migrate-from-jekyl"]
draft = false
+++

Hope y'all are familiar with basics of node.js and how its a popular choice for making server applications. This article aims to shed some light on node.js architecture and how it works internally.

* * * * *

JavaScript on browser and JavaScript Runtimes
---------------------------------------------

JavaScript language is popularly used for web pages and to make them interactive. Thereby javascript code is required to be interpreted by browsers to show as web pages, and this is done by JavaScript Engine.

JavaScript Engine is used to convert JavaScript code into machine code to be executed. JS Engine acts as an interpreter and consists of Memory Heap (Store objects of variables, functions), and Call Stack (To execute functions).

![](https://miro.medium.com/max/60/1*HM7lI4yROtSKnguaLOmsmg.png?q=20)

![](https://miro.medium.com/max/1884/1*HM7lI4yROtSKnguaLOmsmg.png)

JavaScript Engine

Modern browsers creates JavaScript runtime making use of JavaScript Engine to convert web page javascript code to be executed. Different browsers make use different JS Engines to be used within their browser.

![](https://miro.medium.com/max/60/1*G6Df7UN077zp0aCutcok4w.png?q=20)

![](https://miro.medium.com/max/2244/1*G6Df7UN077zp0aCutcok4w.png)

Different modern browsers adapting different JScript engines

As an example Google Chrome browser make use of javascript engine called V8. This V8 is only created to convert javascript language (ECMAScript) into required machine code for the operating system which browser is installed. However Chrome would implement some other Web APIs required for browsers such as `window`, `document` which are not part of javascript language, but elements required for web page.

Also note that since different browser use different javascript engines, and it might interpret javascript code in different ways and thereby some code might yield different results.

* * * * *

JavaScript to run without browser → NodeJS
------------------------------------------

JavaScript being language of the frontend, is known as one of most popular programming language used. Thereby node.js was an attempt to make this language to write backend applications for server, making javascript to be used for full stack development.

In 2009, Ryan Dahl used fastest javascript engine, Chrome's V8 and made it to function without a browser. Wrapper code were written around V8 engine using C++ making it executable directly on operating system.

Node.js will use V8 engine to interpret javascript code into machine code. Since node.js is written for operating system rather than browser, some of Web API for browser such as `window`, `document` is not implemented. However some OS related functionality such as `fs`, `http` is implemented in node.js standard library. Simply node.js is JavaScript runtime environment like a browser.

It should be noted that node.js wouldn't support all javascript features (or ECMAScript) straight away, but features which are implemented in Chrome's V8 Engine. (*Note that functions are released in groups called shipping, staged, in-progress and any experimental features can be enabled by flags: *`*node -v8-options*`)

* * * * *

NodeJS is JavaScript Runtime Environment
----------------------------------------

To wrap up, lets address common misconceptions of node.js.

-   Node.js is not a programming language → JavaScript is the language
-   Node.js is not framework for server application → Express.js is a framework

Simply node.js can be summed as follows by office docs:

> "Node.js® is a JavaScript runtime built on Chrome's V8 JavaScript engine."

* * * * *

NodeJS Architecture and Notable Components
------------------------------------------

![](https://miro.medium.com/max/60/1*5USTrYHazNhCrEDnUeozLw.png?q=20)

![](https://miro.medium.com/max/2648/1*5USTrYHazNhCrEDnUeozLw.png)

NodeJS Architecture

Brief objectives of components are given:

-   V8 JavaScript Engine: Consists of Memory Heap, Call Stack, Garbage Collector and converts javascript code into machine code of given OS.
-   LibUv: Consists of Thread Pool and handles Event Loop, Event Queue. It's multi-platform C library focusing on asynchronous I/O operations.
-   Node.js Standard Library: Consists of libraries operating system related functions for Timers `setTimeout`, File System `fs`, Network Calls `http`.
-   llhttp: parsing HTTP request/response (Previously `http-parse` used)
-   c-ares: C library for async DNS request used in `dns` module.
-   open-ssl: Cryptographic functions used in `tls (ssl)`, `crypto` modules.
-   zlib: Interface to compress and decompress by sync, async and streaming.
-   Node.js API: Exposed JavaScript API to be used by applications

NodeJS Dependencies:

[

Dependencies | Node.js
----------------------

### Node.js® is a JavaScript runtime built on Chrome's V8 JavaScript engine.

#### nodejs.org

](https://nodejs.org/en/docs/meta/topics/dependencies/)

NodeJS API Official Documentation:

[

Node.js v13.5.0 Documentation
-----------------------------

#### nodejs.org

](https://nodejs.org/api/)

As an example, node.exe is an executable for Windows OS where node.js can be used. Either programs can be executed as `./node.exe server.js` or access REPL directly.

* * * * *

NodeJS is
---------

-   Single Threaded → Only one user thread for node.js program execution.
-   Non-Blocking I/O → Not waiting till I/O operation is complete.
-   Asynchronous → Handle dependent code later once its complete.

*Since single threaded:*

-   *There is no locking for shared memory required or handling race conditions making node.js programs less complex.*
-   *I/O operations are asynchronously done thereby not blocking that single thread of execution. How this is achieved via Event Loop is discussed later.*
-   *CPU intensive tasks are not suitable as there isn't multiple threads to support parallelism. So node.js **best suited for I/O intensive and not CPU intensive**.*

* * * * *

JavaScript Execution and Call Stack
-----------------------------------

Javascript engines will include call stack, and javascript being single threaded will have only one call stack. When browser or node.js execute javascript code it should follow order which program is defined in. As an example consider:

Example javascript program

Here in execution order, when setting into function it will be pushed into call stack. If there are function within that function it'll be pushed into stack on top of what was already in call stack. When there are no function is to be pushed, it'll execute function on top of call stack. After execution, when returning from function it will be popped from call stack. So Call Stack: first in last out (FILO) is actually keeping track of functions which are executing.

![](https://miro.medium.com/max/60/1*Pi3tJnRI_ZdYHcnFJE8frA.png?q=20)

![](https://miro.medium.com/max/4036/1*Pi3tJnRI_ZdYHcnFJE8frA.png)

Call Stack upon execution

In browser or node.js, if there is an error or exception it would show error stack trace, or each call stack frame. This will give an idea of call stack status when debugging.

Lets check with undefined variables: `add(val, 32)` → `add(val, thirtytwo)`.

ReferenceError: thirtytwo is not defined\
   at addConst:5:32\
   at convertCtoF:10:12\
   at eval:14:1

Lets check with recursion without condition: `addConst` using `addConst` again.

RangeError: Maximum call stack size exceeded\
   at addConst:5:16\
   at addConst:5:23\
   at addConst:5:23

* * * * *

Call Stack with Slow I/O and blocking nature
--------------------------------------------

Above introduction of V8's call stack show that its synchronous. So assume that there is slow operation such as network or file system, it'll halt execution of single thread and wait till its operation is done. Thereby I/O operations are actually blocking if its executed using just call stack.

const pdf = fs.readFile(file.pdf)\
console.log(pdf.size)\
const doc = fs.readFile(file.doc)\
console.log(doc.size)

In above example `file.pdf` is read then thread is paused there till it's done. Here `file.doc` could have also be read in parallel as its independent of first function call, but due to call stack it'll be executed after `file.pdf` is finished.

![](https://miro.medium.com/max/60/1*gOZzwz3NXV9AK58xQ_1nDQ.png?q=20)

![](https://miro.medium.com/max/1644/1*gOZzwz3NXV9AK58xQ_1nDQ.png)

Serial execution for synchronous I/O

Approaches to handle slow I/O operations could be:

-   Synchronous: Hold up process. Not suitable for single threaded.
-   Fork: Clone new process. Don't scale effectively and high cost.
-   Threads: Complicated for shared memory & need multiple threads.
-   Event Loop: Async callback programming. Suitable for single threaded.

Here node.js have decided to go with Event Loop to handle I/O operations asynchronously, being that its single threaded.

Callbacks are introduced to handle such operations which can run in parallel. Here callback function is given to be executed when first function is done. Hence callback function will include code that is dependent on slow I/O function call.

const pdf = fs.readFile(file.pdf)\
   Then → console.log(pdf.size)\
const doc = fs.readFile(file.doc)\
   Then → console.log(doc.size)

*Note even if javascript only expose one thread for execution & call stack, it should be noted that node.js have thread pool internally. So thereby no limitation for performing multiple I/O operations parallel.*

![](https://miro.medium.com/max/60/1*eMEVR5TwgeW7rc0s5-xwmg.png?q=20)

![](https://miro.medium.com/max/1364/1*eMEVR5TwgeW7rc0s5-xwmg.png)

Parallel execution with callbacks for Asynchronous I/O

* * * * *

Asynchronous Non-Blocking I/O → Event Loop
------------------------------------------

From above it can be seen that node.js is using async callback programming model for handling I/O operations to unblock single thread. Any external operations that might result in delays are attached with a callback function that is to be executed when slow operation is complete. These events executed parallel using Thread Pool, and callbacks are handled by Event Loop.

> The event loop is what allows Node.js to perform non-blocking I/O operations --- despite the fact that JavaScript is single-threaded --- by offloading operations to the system kernel whenever possible.

This diagram shows major components interact to provide asynchronous I/O:

![](https://miro.medium.com/max/60/1*eE_fM5qLGM_pY-J-8u54Mw.png?q=20)

![](https://miro.medium.com/max/2884/1*eE_fM5qLGM_pY-J-8u54Mw.png)

NodeJS Architecture with V8, LibUv and Standard Library

-   Call Stack would execute as explained above, but when there is functions with delay it would not be blocked there. Rather it would fire that event with associated callback function, and will continue with rest of the code.
-   Executed external function would run in background (not in main single thread) using libuv thread pool. For example `fs` read file command will be executed using node.js standard library by background thread & when it's completed, it would add that event and callback to Event Queue (AKA Callback Queue).
-   This Event Queue is consisted of all callback functions of completed functions that are waiting to be executed by main thread. For callback function to be executed by main thread, it should be moved to Call Stack.
-   Event Loop comes into play here, where it'll move callback functions from Event Queue to Call Stack to be executed by main thread. (Event Loop and Call Stack is run by main thread.)
-   When Call Stack is empty and Event Queue have pending functions, Event Loop moves event and its callback from Event Queue to Call Stack and will be executed by main thread.

Thereby any slow I/O operations would not be blocking main thread, but rather passed to node.js to be executed in background. Hence main thread continues with its program in non-blocking nature. Once I/O operation is complete, its callback will be executed by main thread using Event Loop and Event Queue.

* * * * *

NodeJS Concurrency Model
------------------------

Nodejs through Event Loop and Event Queue have build up asynchronous event-driven non-blocking I/O model to support concurrency. Due to single threaded javascript, main thread was not halted for slow external events but rather converts external events into callback invocations.

Let's check following javascript code handle I/O operation asynchronously:

console.log("BEFORE TIMEOUT FUNCTION");setTimeout(function timeout() {\
 console.log("TIMEOUT FUNCTION");\}, 5000);console.log("AFTER TIMEOUT FUNCTION");

![](https://miro.medium.com/freeze/max/60/1*Ra0O_48FDyY2L57zamiFUA.gif?q=20)

![](https://miro.medium.com/max/2696/1*Ra0O_48FDyY2L57zamiFUA.gif)

Call Stack animation with <http://latentflip.com/loupe>

Thereby output will be as follows where callback function is delayed:

`BEFORE TIMEOUT FUNCTION` → `AFTER TIMEOUT FUNCTION` → `TIMEOUT FUNCTION`

* * * * *

I/O Operations by NodeJS Standard Library
-----------------------------------------

It should be properly understood what nodejs considers I/O operation, and is executed by internal thread pool without blocking main thread.

Some notable I/O operations in node.js are not necessarily part of javascript specs in ECMAScript hence not implemented by javascript engine: *(These are implemented either by nodejs or browser. As an example for nodejs, timers used in *`*lib/timers.js*`* which is implemented in *`*src/timers.cc*`* and creates an instance of *`*Timeout*`* object. For example *`*setTimeout*`* is implemented in both nodejs as well as browsers, and will be relative to *`*window*`* in browser, and *`*global*`* object in nodejs.)*

-   Node.js File Systems

*External operations handled by *`*fs*`* module such as reading or writing to files. For example *`*fs.write*`*, *`*fs.readStream*`*, *`*fs.stat*`*.*

-   Node.js Network Calls

*Network calls for any external party such as *`*dns.resolve*`*, *`*dns.lookup*`*, *`*http.get*`*, *`*https.post*`*, *`*socket.connect*`*.*

-   Node.js Timers

*Time related operations to be handled later such as *`*setTimeout*`*, *`*setImmediate*`* or *`*setInterval*`*. Even *`*setTimeout(cb, 0)*`* with *`*0 ms*`* delay will have delay as its pushed to event queue and executed after other code in call stack. Hence this *`*setTimeout*`* can't guarantee exact time but rather give minimum time for execution**.*

It should be note that all time consuming tasks are not considered I/O. For example CPU intensive operation in a loop isn't an I/O operation, & would be blocking main thread. (Thereby nodejs is considered not suitable for CPU intensive tasks but rather I/O intensive tasks.)

for (var i = 0; i < 10000; i++) {\
   crypto.[createHash](https://www.codota.com/code/javascript/functions/crypto/createHash)(...) or sleep(2) || CPU intensive task\
}\
console.log('After CPU Intensive Task');

Here CPU intensive task would block main thread and wouldn't execute console as it would with asynchronous I/O operations. Simply when CPU bound task is encountered its executed immediately, and for I/O bound task its passed to libuv to be handled asynchronously.

It should be noted that if call stack is always not empty due to these CPU bound tasks and blocking main thread, it will never execute anything from event queue of any pending I/O tasks and would lead exhaustion.

* * * * *

Event Loop Phases
-----------------

Previously it was mentioned that Event Loop will be monitoring for empty Call Stack and pending Event Queue to execute callbacks in Call Stack. This procedure follows few stages as following diagram:

![](https://miro.medium.com/max/56/0*DAcZxPG7XD6bcfMw?q=20)

![](https://miro.medium.com/max/982/0*DAcZxPG7XD6bcfMw)

Image src: [Node.js Event Loop](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)

-   Timers --- executes callbacks that have been scheduled with `setTimeout` and `setInterval`.
-   I/O callbacks: executes almost all callbacks with the exception of close callbacks, timers callbacks, and `setImmediate()`.
-   Idle, prepare: internal to Node.js
-   Poll: retrieve new I/O events accepts incoming connections and data processing
-   Check --- invokes callbacks set using `setImmediate`
-   Close callbacks --- runs callbacks for close events eg:`socket.on('close')`.

This demonstrate stages of JavaScript Event Loop as executed by main thread. Each stage demonstrate how Event Loop decides to add any waiting callback functions into call stack to be executed and be completed.

This shed some light on confusing timers including`*setTimeout(cb, 0)*`vs`*setImmediate(cb)*`vs`*process.nextTick(cb)*`*.*As Event Loop stages show `process.nextTick` execute first as it would put that callback into front of Event Queue. Comparing next two, `setTimeout` would push its callback to timer queue while `setImmediate` would push to check queue in those stages. As event loop occur timer queue is executed before check queue making `setTimeout` to execute next, and finally it will be `setImmediate`.

* * * * *

Let's wrap it up...
=================

Gist of NodeJS Architecture
---------------------------

To wrap things up about architecture, nodejs runtime contains V8, LibUv, standard library and related bindings.

> libuv | Cross-platform asynchronous I/O
>
> libuv is a multi-platform support library with a focus on asynchronous I/O. It was primarily developed for use by [Node.js](https://nodejs.org/).

LibUv contains capabilities of event loop, async tcp/udp, async file, thread pool, child processes and handles these platform independently.

> V8 JavaScript Engine
>
> V8 is Google's open source high-performance JavaScript and WebAssembly engine, written in C++. It is used in Chrome and in Node.js
>
> It implements [ECMAScript](https://tc39.es/ecma262/) and [WebAssembly](https://webassembly.github.io/spec/core/), and runs on Windows 7 or later, macOS 10.12+, and Linux systems

V8 contains capabilities of heap memory allocation, call stack execution, garbage collector, optimization compiler and javascript interpreter.

Here nodejs make use of V8 javascript engine, and hence known as V8 embedder. As per V8 engine's requirement, embedder should implement an event loop. Here nodejs being V8 embedder have chosen libuv as event loop implementation. This is where V8 and LibUv is connected, and it's done using C++ bindings.

Being single threaded, nodejs will have one event loop. Between each event loop run, nodejs checks for any waiting asynchronous I/O and shuts down if there isn't any. Simply event loop doesn't generate instantly but when there are pending callbacks to be executed. Finally nodejs application terminates when there isn't any events in call stack or event queue.

* * * * *

Gist of NodeJS Concurrency Model
--------------------------------

To wrap things up about concurrency model containing single thread, event loop and call stack to build up Asynchronous Non-blocking I/O model.

-   Node.js application executed by Node.js
-   If function is CPU bound push to call stack and execute synchronously on main thread.
-   If function is I/O bound push to thread pool to execute asynchronously and continue with execution.
-   Once asynchronous function is complete thread pool push callback and event into event queue.
-   Event loop using main thread looks for pending callbacks on event queue if call stack is empty. Add to call stack to execute callback if its empty.

* * * * *

When NodeJS is useful?
----------------------

NodeJS simply provides non-blocking asynchronous I/O model even with single thread. This makes nodejs more suitable for I/O intensive applications than other available options. (Note CPU intensive applications are not suited for nodejs being its single threaded, and will block execution.)

As javascript programming language is used in nodejs, developers are able to make full stack applications all in one language.

![](https://miro.medium.com/max/54/1*kQDy60fK6Un7HwGFMmMQPA.png?q=20)

![](https://miro.medium.com/max/592/1*kQDy60fK6Un7HwGFMmMQPA.png)