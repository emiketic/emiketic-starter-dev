# Asynchronous Operations in JavaScript

# Intro

This documents aims to provide an analytical overview on asynchronous operations in JavaScript environments, most notably Node.js, React, React Native and Redux.

# State of the Art

## _Node.js-style Callback_

- A function uses a callback function, provided as last argument, to communicate operation outcome.
- The callback function takes an `err` and a `result` arguments, in that order.
  - `err` communicates an error, otherwise is null.
  - `result` communicates outcome
- `+` Simple to understand and operate.
- `-` Difficult to manage multiple dependent and/or nested asynchronous operations, causing [callback hell](http://callbackhell.com/)

**Example:** read content from url and save it in file

```javascript
// Definition
function getURLContent(url, callback) {
  ...
  callback(new Error('Something is wrong!')); // communicate error
  ...
  callback(null, response.body); // communicate result
}

// Usage
getURLContent('http://api.example.com/timestamp', function (err, content) {
  if (err) {
    console.error(err.message);
    return;
  }

  saveContentToFile(content, function (err, file) {
    if (err) {
        console.error(err.message);
        return;
    }

    console.log(file);
  });
});
```

## _Node.js-style Callback_ + _caolan/async_

- Solves most issues with _Node.js-style Callback_ by providing a set of utility functions for flow control (series, parallel, dependence, ...) and collection management such as executing and asynchronous operation on each item of an array in parallel or in series.
- `+` Provides a powerful asynchronous operation dependency management beyond parallel and series by mean of `async.auto` and `async.autoInject`
- `+` Supports _async/await_
- `-` Difficult to manage conditional execution in some cases.

```javascript
async.map(['file1','file2','file3'], fs.stat, function(err, results) {
    // results is now an array of stats for each file
});

async.filter(['file1','file2','file3'], function(filePath, cb) {
  fs.access(filePath, function(err) {
    cb(null, !err)
  });
}, function(err, results) {
    // results now equals an array of the existing files
});

async.parallel([
    function(cb) { ... cb(ert, result) },
    function(cb) { ... cb(ert, result) }
], function(err, results) {
    // optional callback
});

async.series([
    function(cb) { ... cb(ert, result) },
    function(cb) { ... cb(ert, result) }
]);
```

**Example:** read content from url and save it in file

```javascript
// Definition
function getURLContent(url, callback) {
  ...
  callback(new Error('Something is wrong!'));
  ...
  callback(null, response.body);
}

// Usage
const async = require('async');

async.autoInject({
    content: (cb) => getURLContent('http://api.example.com/timestamp', cb), // no dependency, async would execute immediately
    file: (content, cb) => saveContentToFile(content, cb), // depends on `content`, async would wait for `content`' result then executes this
}, function (err, result) {
  if (err) {
    console.error(err.message);
    return;
  }

  console.log(result.file);
})
```

**References:**

- [_caolan/async_](http://caolan.github.io/async/)

## _Promise_

- Provides a distinct asynchronous operation model relying on callbacks to communicate outcome.
- `+` Solves the dependent/nested asynchronous operations problem with callbacks and keeps code as flat as possible.
- `+` Provide useful features such as
  - chaining asynchronous operations (`doSomething().then(...).then(...).then(...)`)
  - parallel execution (`*Promise*.all`)
  - racing (`*Promise*.race`)
  - error recovery (`.catch((err) => alternativeResult).then(...)`))
- `-` Difficult to have readable code when managing shared state.

**Example:** read content from url and save it in file

```javascript
// Definition
function getURLContent(url) {
  return new *Promise*(function(resolve, reject) {
    ...
    reject(new Error('Something is wrong!')); // communicate error
    ...
    resolve(response.body); // communicate result
  });
}

// Usage
getURLContent('http://api.example.com/timestamp')
  .then(function (content) {
    return saveContentToFile(content); // chaining
  })
  .then(function (file) {
    console.log(file);
  });
  .catch(function (err) {
    console.error(err.message);
  });
```

**References:**

- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/*Promise*
- https://developers.google.com/web/fundamentals/primers/promises
- https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html

## _async/await_

- Built on top of _Promise_.
- An `async` function returns a promise.
- `await` intercepts a promise outcome and returns result or throws error.
- `+` Enables code style and behavior that mimics synchronous code.
- `+` Results to more readable code.
- `-` Delegates parallel execution and racing to _Promise_.

**Example:** read content from url and save it in file

```javascript
// Definition...

async function getURLContent(url) {
  let response = await fetch(url);
  if (!response.ok) {
    throw new Error('Something is wrong!')); // communicate error
  }
  let content = await response.text();
  return content;  // communicate result
}

// Usage
try {
    let content = await getURLContent('http://api.example.com/timestamp');
    let file = await saveContentToFile(content)
    console.log(file);
} catch (err) {
    console.error(err.message);
}
```

**References:**

- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function
- https://developers.google.com/web/fundamentals/primers/async-functions

## Generator

TODO...

## EventEmitter

- Used to register and invoke event listeners.
- `+` Useful for decoupling modules
- `-` Difficult to manage errors

```javascript
const EventEmitter = require('events').EventEmitter;

const events = new EventEmitter();

events.on('foo', function() {
  console.log('got foo');
});

events.emit('foo');
```

**Example:** read content from url and save it in file

```javascript
const EventEmitter = require('events').EventEmitter;

const BUS = new EventEmitter();

BUS.on('saveURLContentToFile', function(url, done) {
  getURLContent(url, function(err, content) {
    if (err) {
      console.error(err.message);
      return;
    }

    saveContentToFile(content, function(err, file) {
      if (err) {
        console.error(err.message);
        return;
      }

      console.log(file);

      done(content);
    });
  });
});

BUS.emit('saveURLContentToFile', 'http://api.example.com/timestamp', (content) => console.log('done', content));
```

**References:**

- https://nodejs.org/api/events.html
- https://www.npmjs.com/package/events
- https://www.npmjs.com/package/eventemitter3

## Interoperability

_Node.js-style Callback_ and _Promise_ are interchangeable formats.
Besides, an `async` function is actually a promise definition.

### Converting from _Node.js-style Callback_ to _Promise_

```javascript
function getURLContentPromise(url) {
  return new *Promise*((resolve, reject) => {
    getURLContentCallback(url, (err, result) => {
      if (err) {
        reject(err);
        return;
      }
      resolve(result);
    });
  });
}
```

Node.js v8 provides an utility function `util.promisify`, that does this:

```javascript
const getURLContentPromise = util.promisify(getURLContentCallback);
```

### Converting from _Promise_ to _Node.js-style Callback_

```javascript
function getURLContentCallback(url, callback) {
  getURLContentPromise(url)
    .then((result) => callback(null, result))
    .catch(callback);
}
```

### Using an async function as _Promise_ function

```javascript
getURLContentAsync(url)
  .then(...)-
  .catch(...)
```

### Using a _Promise_ function as async function

```javascript
const content = await getURLContentPromise(url);
```

# Technology Stack Usage

## Node.js

_Node.js-style Callback_ + _caolan/async_ is most recommended.

Some libraries have hybrid API, i.e. returns a _Promise_ if no callback is provided.

Some other libraries are opting for _Promise_ API, therefore _async/await_ is usable.

**Recommendation:** Use _async/await_ when possible, otherwise use _Node.js-style Callback_ + _caolan/async_.

## React & React Native

There is no particular restriction on any of the asynchronous models with slight preference for _Promise_ and _async/await_.

**Recommendation:** Use _async/await_.

# Conclusion

1.  Use the most adapted and simple tool for the job.
2.  Prefer _async/await_ and _Promise_.
