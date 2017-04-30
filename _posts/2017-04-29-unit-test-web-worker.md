---
layout:     post
title:      Unit Testing Complex Web Workers in AngularJS with Karma and Jasmine
date:       2017-04-29T13:30:00-0700
summary:    Creating unit tests for complex Web Workers can be difficult since they run in an isolated context and only expose methods for sending data into the worker and receiving data back when the worker is finished. To unit test individual Web Worker functions, we need to execute the worker in an accessible context, expose its methods, and mock the isolated context properties workers expect.
categories: article
tags:
- AngularJS
- Karma
- testing
- Jasmine
- Web Worker
permalink: /articles/unit-test-web-worker
---

Creating unit tests for complex Web Workers can be difficult for a few reasons. First, they run in an isolated [DedicatedWorkerGlobalScope](https://developer.mozilla.org/en-US/docs/Web/API/DedicatedWorkerGlobalScope) context that is separate from the current window. Second, Web Workers only expose a `postMessage` method for sending data into the worker and an `onmessage` method for receiving data back when the worker is finished. To unit test individual Web Worker functions, we need to execute the worker in an accessible context, expose its methods, and mock the isolated context properties that workers expect.

This article expands on [Ryan Oglesby](http://ryanogles.by/about.html)'s original ideas in [*"Testing JasvaScript Web Workers with Jasmine"*](http://ryanogles.by/javascript/jasmine/html5/testing/2014/08/29/testing-jasvascript-web-workers-with-jasmine.html).

## Worker Setup
1. Wrap the worker code in a function expression using `this` as the execution context. This pattern allows you to instantiate the worker in a controlled context during testing and distinguish between the testing and production environment by checking for one of the unique methods of the `DedicatedWorkerGlobalScope` context.
2. Structure the Web Worker to expose all functions on the current context and return the context if it's not `DedicatedWorkerGlobalScope`. This will make it possible to test each worker function directly.

**Example worker:** */scripts/find-primes.worker.js*

```javascript
/**
 * An example Web Worker that finds the prime numbers in a given array of numbers.
 */
'use strict';

(function(context) {

  // Alias the current context so it can be returned for testing.
  var windowContext = true,
      worker = context || {};

  // Check for a method unique to DedicatedWorkerGlobalScope to determine execution context.
  if (typeof worker.importScripts !== 'undefined') {
    windowContext = false;
  }

  // Define our worker methods.
  worker.findPrimes = findPrimes;
  worker.isPrime = isPrime;
  worker.onmessage = onMessage;

  function findPrimes(list) {
    var primes = [];
    list.forEach(function(num) {
      if (worker.isPrime(num)) {
        primes.push(num);
      }
    });
    return primes;
  }

  function isPrime(num) {
    for (var i = 2, s = Math.sqrt(num); i <= s; i++) {
      if (num % i === 0) { return false; }
    }
    return num !== 1;
  }

  function onMessage(messageEvent) {
    var primes = worker.findPrimes(messageEvent.data);
    worker.postMessage(primes); // Expected to exist on context.
    worker.close(); // Expected to exist on context.
  }

  // If execution context is window return for testing.
  if (windowContext) { return worker; }
})(this);
```

Note that while Web Workers allow external scripts to be imported (that's the `importScripts` property we're using to test for the execution context), using imported scripts poses some challenges for testing and build/distribution so it may not be worth the trouble. If you do need to import scripts into the Web Worker and make them available to worker inside your unit tests, you can mock the objects the scripts provide and inject them before each test.

## Test Helpers
1. Create a spec helper with methods to get and instantiate the Web Worker in the `window` context
2. Optionally, create a mock for the Web Worker including the methods expected from `DedicatedWorkerGlobalScope`

**Example Spec Helper: /test/unit/spec-helper.js**

```javascript
/* jshint evil:true */

'use strict';

(function() {

  angular
    .module('app.test.specHelper', [])
    .factory('SpecHelper', SpecHelper);

  SpecHelper.$inject = [];

  function SpecHelper() {
    var service = {
      getWorker: getWorker,
      testWorker: testWorker
    };

    return service;

    /**
     * Gets a Web Worker by path.
     * @param  {string} path The Web Worker file path.
     * @return {string}      The Web Worker file text.
     */
    function getWorker(path) {
      var http = new XMLHttpRequest();
      http.open("GET", path, false);
      http.send();
      return http.responseText;
    }

    /**
     * Enables Web Worker testing by instantiating the worker code in a local context and providing
     * an interface for communication.
     * @param  {string} workerCode The Web Worker file text to test.
     * @return {object}            A local version of the worker to test.
     */
    function testWorker(workerCode) {
      var worker = createWorkerContext(workerCode);

      // Mock the DedicatedWorkerGlobalScope close and postMessage methods.
      worker.close = function() {};
      worker.postMessage = function(data) { return data; };

      // Execute the worker like the browser would.
      function createWorkerContext(str) {
        return eval(str);
      }

      return {
        getWorker: function() { return worker; }
      };
    }
  }
})();

```

## Test Setup
1. Inject the spec helper before each test
2. Fetch the Web Worker file using */base* as the folder root (see karma issue [#1607](https://github.com/karma-runner/karma/issues/1607))

**Example Test: /scripts/find-primes.worker.test.js**

```javascript
'use strict';

describe('findPrimes Web Worker', function() {
  var SpecHelper,
      workerInstance,
      workerCode;

  beforeEach(module('app.test.specHelper'));

  // Make our spec helper service available to each test.
  beforeEach(inject(function(_SpecHelper_) {
    SpecHelper = _SpecHelper_;
  }));

  // Get the Web Worker and create a new instance of it.
  beforeEach(function() {
    workerCode = SpecHelper.getWorker('/base/scripts/find-primes.worker.js');
    workerInstance = SpecHelper.testWorker(workerCode);
  });

  // Test each worker function independently.
  describe('findPrimes', function() {
    it('should call the isPrime method once for each item in a supplied array and return an array.', function() {
      var list = [3, 5, 7],
          primes,
          worker = workerInstance.getWorker();

      spyOn(worker, 'isPrime').and.returnValue(true);

      primes = worker.findPrimes(list);
      expect(worker.isPrime.calls.count()).toEqual(3);
      expect(primes).toEqual(list);
    });
  });

  //...etc.
});

```

## Final Thoughts
Blind testing through `postMessage` and `onmessage` might be sufficient for very simple workers but with a little extra setup we can easily fully test the code inside. Thanks again to [Ryan Oglesby](http://ryanogles.by) for working out most of this and saving me lots of time!