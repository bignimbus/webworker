![WebWorker logo](https://raw.githubusercontent.com/tanzeelkazi/webworker/master/img/webworker-logo-128.png)
# WebWorker

Version: 1.0.0a

Event driven web worker javascript framework.

Don't you just hate having only the `postMessage` API to communicate with your workers?
Does it leave you wanting for more and distract you away from actual development
and keep you busy around the communication between the worker and the page? EXACTLY!

The `WebWorker` API enables you to communicate with your workers using an event-based
trigger-listener model. Your worker can trigger any event from within itself and the base
page worker instance listener will be triggered with the event data.
Now isn't _that_ just how it should be done!

The `WebWorker` class is a wrapper for the existing native `Worker` browser API. I created this project
because the native worker communication of using ONLY `postMessage` left me wanting for more.
Instead of working on the worker logic itself I found myself creating a whole communications
channel between the worker and the base page every time I wanted to set up a new worker.


## Features
- **Delayed loading**
  - Your web-worker scripts are not loaded until you explicitly call `.start()` or `.load()`
  on the `WebWorker` instance.
- **Delayed starting**
  - Your worker script will not execute unless an explicit call to `.start()` is made.
- **Robust event-driven API**
  - No pulling hair out for using just the `postMessage` API.
  - No mystery callback hooks for communication between the page and the web worker.
- **Custom event support (freedom from being stuck to the `postMessage` API)**
  - Define your own custom events for use from within and/or outside the web worker.
- **Concentrate more on doing than communicating**
  - No more architecting communication protocols between the thread and the base page. This API
  lays down that communication foundation for you in a way that let's you focus on the problem.
- **Logging support (both on the instance on the base page as well as from within the worker thread)**
  - Can't figure out what's breaking inside your worker script? No problem! Although I can't promise you
  a `debugger`, you can log data from within the worker as simply as `self.log(data);` and view the data
  on the base page on the worker instance using `worker.getLog();`. Much easier than struggling with
  `postMessage` and figuring out the hundreds of calls that may have gone through.
- **`postMessage` fallthrough**
  - The `WebWorker` API uses `postMessage` to communicate between the thread and the base page in a
  completely transparent manner. It ignores all `postMessage` calls that are not flagged as intrinsic
  `WebWorker` messages and thus allows you to use the `postMessage` API as you deem fit.
- **Ability to work with native worker object**
  - It is possible to hook into the native worker object to perform any custom actions that you may need.
- **Robust state tracking mechanism**
  - The `WebWorker` API has a robust state tracking mechanism which helps to define a stable workflow for
  your thread from creation to termination.

## Getting started
Getting started is very simple. Just drag-drop the `src/js/web-worker.js` file into your project and you
are ready to go.

## Usage
Following are example usages of the script.

#### Simple usage:
```javascript
var worker = new WebWorker('./worker-script.js');

worker.on('my-custom-event', function () {
          ...
      })
      .on('my-custom-complete-event', function () {
          ...
          worker.terminate();
      });
...
worker.start();
```

#### Using the helpers to quickly define tasks:
```javascript
var worker = new WebWorker('./worker-script.js');
worker.loading(function () {
          console.log('worker is loading');
      })
      .loaded(function () {
          console.log('worker has loaded');
      })
      .starting(function () {
          console.log('worker is starting');
      })
      .started(function () {
          console.log('worker has started');
          ...
          worker.terminate();
      })
      .terminating(function () {
          console.log('worker is terminating');
      })
      .terminated(function () {
          console.log('worker has terminated');
      })
      .error(function () {
          console.log('worker encountered an error');
      });

worker.start();
```

#### Attaching events on the worker object:
```javascript
var worker = new WebWorker('./worker-script.js');
worker.on('my-custom-event', function () {
    console.log('custom event triggered!');
});
...
worker.start();
```

#### Triggering events from within the worker script:
```javascript
self.trigger('my-custom-event');
```
Triggering events within the worker script triggers the event on the worker object on
the base page.

#### Pre-loading without starting:
When you call `.start()`, by default the worker script will load the web worker script and start the worker
for you when ready. If for some reason you wish to do a _pre-load_ of the worker without starting it
immediately, you can use the `.load()` method. Thereafter you may start the worker by explicitly
calling `.start()`.

```javascript
var worker = new WebWorker('./worker-script.js');
worker.on('my-custom-event', function () {
    console.log('custom event triggered!');
});
...
// Load the web worker script
worker.load();
...
// Start the worker script
worker.start();
```

**Important Note:**

The major difference between this library and the native web-worker implementation is that the native
implementation loads and executes the worker script immediately on instantiation. You may have read about
the native worker _starting_ after calling _postMessage_ but that is a bit misleading since _postMessage_ is
intercepted within the _already executed_ worker which can then be used to perform specific tasks depending
on what message was passed.

The `WebWorker` API however, has a _delayed loading_ and _delayed starting_ implementation. What this means is
that calling `.load()` will load the worker script from the server but will NOT execute your portion of the
script unless you explicitly call `.start()`. If you call `.start()` without calling `.load()`, it implicitly
calls `.load()` first and then starts execution when ready.

#### Logging inside and outside the worker:
Within the worker script you can log with:
```javascript
self.log('worker data');
```
The data will be logged within the worker instance on the base page for easy viewing/debugging.

On the worker instance on the base page, you can still log data.
```javascript
worker.log('data on base page');
```
All logged data can be retrieved with the `.getLog()` method that returns an array of logged items.
```javascript
var logData = worker.getLog(); // logData = ['worker data', 'data on base page']
```

#### The native worker object:
If you wish to retrieve and use the native worker object being used by the `WebWorker` you can do so by
using the `.getNativeWorker()` method.
```javascript
worker.getNativeWorker();
```
Note that the native worker object is available only _after_ the worker has loaded i.e. once
the `.load()` call for the worker is complete.
```javascript
worker.loaded(function () {
    var nativeWorker = this.getNativeWorker();
});
...
worker.load();
```
The same holds true if you have called `.start()` to
start the worker as it implicitly calls `.load()`.
```javascript
worker.loaded(function () {
    var nativeWorker = this.getNativeWorker();
});
...
worker.start();
```

#### Tracking worker states:
The `WebWorker` API has a robust state tracking mechanism. The worker can be in any _one_ of the
following states at any given time.

- INITIALIZED
- LOADING
- LOADED
- STARTING
- STARTED
- TERMINATING
- TERMINATED

The current state of the worker can be retrieved with the `.getState()` method.
```javascript
worker.getState(); // returns an integer corresponding to one
                   // of the valid states of the worker.
```
The following example will help you understand the various states the worker
goes through.
```javascript
var worker = new WebWorker('./worker-script.js');

console.log( worker.getState() ); // WebWorker.State.INITIALIZED (0)

worker.loading(function () {
          console.log( this.getState() ); // WebWorker.State.LOADING (1)
      })
      .loaded(function () {
          console.log( this.getState() ); // WebWorker.State.LOADED (2)
      })
      .starting(function () {
          console.log( this.getState() ); // WebWorker.State.STARTING (3)
      })
      .started(function () {
          console.log( this.getState() ); // WebWorker.State.STARTED (4)

          // Terminate the worker when done
          worker.terminate();
      })
      .terminating(function () {
          console.log( this.getState() ); // WebWorker.State.TERMINATING (5)
      })
      .terminated(function () {
          console.log( this.getState() ); // WebWorker.State.TERMINATED (6)
      });

worker.start();
```
There are easy helpers available if the need arises to use the current state of the worker
in your logic. It is recommended to use these instead of checking the `.getState()` values.
```javascript
worker.isLoading();     // TRUE when worker state is WebWorker.State.LOADING

worker.isLoaded();      // TRUE when worker state crosses WebWorker.State.LOADED
                        // but is not WebWorker.State.TERMINATED

worker.isStarting();    // TRUE when worker state is WebWorker.State.STARTING

worker.isStarted();     // TRUE when worker state crosses WebWorker.State.STARTED
                        // but is not WebWorker.State.TERMINATED

worker.isTerminating(); // TRUE when worker state is WebWorker.State.TERMINATING

worker.isTerminated();  // TRUE when worker state is WebWorker.State.TERMINATED
```


## Requirements
Production use requirements are `jQuery 1.9+`.

If you are into developing on the project then you need to have at least `Node.js` installed.
The project has a `setup.sh` file (`setup.bat` for Windows users) that will set up the project
dependencies for you and run the post-install script. Isn't that the easiest way to get set
up!


## Browser support
The following list of browsers are known to be supported at this time.

- ![Chrome](https://raw.githubusercontent.com/alrra/browser-logos/master/chrome/chrome_16x16.png) Chrome 38+
- ![Firefox](https://raw.githubusercontent.com/alrra/browser-logos/master/firefox/firefox_16x16.png) Firefox 33+
- ![Safari](https://raw.githubusercontent.com/alrra/browser-logos/master/safari/safari_16x16.png) Safari 7+
- ![Internet Explorer](https://raw.githubusercontent.com/alrra/browser-logos/master/internet-explorer/internet-explorer_16x16.png) Internet Explorer 11+
