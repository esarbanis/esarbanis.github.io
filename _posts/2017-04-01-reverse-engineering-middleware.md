---
layout: post
title: Reverse engineering a middleware
---

There came a time that I had to write a middleware-like application using just `es6`. So, instead of re-inventing the wheel I decided to 'borrow' the api of a very well known middleware framework, namely [expressjs](https://expressjs.com).

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
    let app;

    beforeEach(() => app = exressus());

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

Now we have the initialization, but we should probably have a way to start this middleware. In the example it is done like so:
```javascript
app.listen(3000,  () => {
  console.log('Example app listening on port 3000!')
})
```

Before we can test that we need to introduce a spying library to our test tools. ```chai-spies``` seems to be an appropriate choice, but before we can use it we need to properly set it up.

```bash
npm i -D chai-spies
echo "--require test/setup.js" > test/mocha.opts
cat <<EOF > test/setup.js
const chai = global.chai = require('chai');
spies = require('chai-spies');
chai.use(spies);
EOF
```

OK, let's create our test case, shall we?
```javascript
const {expect, spy} = require('chai');
...
it('should call the initialization callback', () => {
    const init = spy();
    app.listen(3000,  init);
    expect(init).to.have.been.called();
});
...
```

Now try to make it pass:
```javascript
module.exports = () => {
    return {
        listen(port, init) {
            init();
        },
    }
};
```

You might notice that we are not doing much. We just facilitated the test case with the minimum code. Well that is actually what TDD is all about. If we wanted something more, then we should have created an appropriate failing test. But, it is not time yet.

First, let's turn our attention to the next 'feature' which is registering a simple middleware in the `/` path for `GET` requests. First, we should write a test for that.

This is going to be a little bit more complex, as we will need to introduce an agent for testing endpoints. In our case we choose `supertest`.
```bash
npm i -D supertest
```

Now let's setup the test case and work our way into the implementation:
```javascript
it('should be able to register a function to a path', (done) => {
  app.get('/', (req, res) => {
    res.end('Hello World!'); // Use end instead of send. Request decoration is out of scope.
  });

  request(app).get('/').expect(200, 'Hello World!', done);
});
```

In order to make that test pass, we have to add meaningful functionality to the library and actually introduce our first platform dependency; the ```http``` library.
```javascript
const http = require('http');
```

We have to create a registry for our middlewares, which is going to be indexed by the http method, for more performant access. We are only dealing with ```GET``` so we only put that.
```javascript
const registry = {
    GET: {}
};
```

Furthermore, we have to register the middlewares to our registry. The middleware contract should have the ```path``` it is registered to and the ```action``` to perform on that path.
```javascript
get(path, action) {
    registry.GET[path] = action;
}
```

The middleware registry is set, but we have to have a way to invoke the registry actions. We can achieve this by implementing an interceptor and attach it to the ```http.Server```. By doing so we can get rid of the dummy ```listen``` method and use the ```http.Server```'s one.
```javascript
const interceptor = (req, res) => {
    const path = url.parse(req.url).pathname;
    const {method} = req;
    const action = registry[method][path];

    action(req, res);
};

module.exports = () => {
    const server = http.createServer(interceptor);

    return Object.assign(server, {
        get(path, action) {
            registry.GET[path] = action;
        }
    });
};
```

The ```it('should start the server')``` test should be refactored now to test the new functionality.
 ```javascript
it('should start the server', () => {
    const server = app.listen(3000);
    expect(server.listening).to.be.true;
});
```

This is a good point to conclude, as we have managed to implement the minimum functionality needed to facilitate the Hello World example in the [Getting Started Guide](https://expressjs.com/en/starter/hello-world.html).

---
You can find this version of the code in this [github repository](https://github.com/esarbanis/expressus/tree/Part1)