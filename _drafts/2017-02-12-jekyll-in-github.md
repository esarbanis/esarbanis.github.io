---
layout: post
title: Reverse engineering a middleware
---

There came a time that I had to write a middleware-like application using just `es6`. So, instead of re-inventing the wheel I decided to 'borrow' the api of a very well know middleware framework, namely [expressjs](https://expressjs.com).

First things first, went to the [Getting Started](https://expressjs.com/en/starter/hello-world.html) page and grabbed the 'hello world' example:

```javascript
var express = require('express')
var app = express()

app.get('/', function (req, res) {
  res.send('Hello World!')
})

app.listen(3000, function () {
  console.log('Example app listening on port 3000!')
})
```

Let's focus on the initialization part:
```javascript
var express = require('express')
var app = express()
```
It would be nice not to name my implementation 'express' as well. So, lets call it  ... express ... but in latin.
```javascript
var expressus = require('expressus')
var app = expressus()
```

It would be nice to start implementing right about now.
```bash
mkdir expressus
cd expressus
npm init
touch index.js
```

Great! Now let's setup testing. As we are going to reverse engineer the api it is imperative to use TDD for it.
```bash
npm i -D mocha chai
mkdir test
touch test/test.js
```
Edit `package.json` and edit the `scripts > test` to be just `"test": "mocha"`

Superb! Let's write our first test case. Essentially at the begining we just want whatever `expressus()` call returns to not be null:
```javascript
const {expect} = require('chai');
const exressus = require('../index.js');

describe('expressus', () => {
  it('should initialize the framework', () => {
    expect(app).not.to.be.null;
  });
});

```

Of course this test is going to fail as we haven't implemented anything yet. Let's make that pass. Edit `index.js` and add a function.
```javascript
module.exports = () => {
  return {}
};
```

Great! Let's turn our attention to the next 'feature' which is registering a simple middleware in the `/` path for `GET` requests. First, we should write a test for that.

This is going to be a little bit more complex, as we will need to introduce an agent for testing endpoints. In our case we choose `supertest`.
```bash
npm i -D supertest
```

Now let's setup the test case and work our way into the implementation:
```javascript
it('should be able to register a function to a path', (done) => {
  app.get('/', (req, res) => {
    res.send('Hello World!')
  });

  request(app).get('/').expect(200, 'Hello World!', done);
});
```