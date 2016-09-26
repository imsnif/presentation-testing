class: center, middle

# Testing (Javascript) Best Practices

.footnote[https://github.com/imsnif/presentation-testing]

---
# What we'll be talking about
## Why?
So we'll (hopefully) all agree tests are awesome.
## What?
Test best practices and meta-guidelines.
## How?
Specific testing guidelines.
## Extras
Testing patterns (coverage, mocks, spies, fixtures)
---

.left-column[
  ## Intro
  ### Why?
  ### What?
  ### How?
  ### Extras
]

.right-column[
  ### Disclaimers

  * This talk is js-centric, but also deals with language-agnostic good practices.

  * The best practices described here are based on my opinion.
]

---

.left-column[
  ## Intro
  ### Why?
  ### What?
  ### How?
  ### Extras
]

.right-column[
  ### Test Types

  * <b>Unit</b> - test the smallest unit of functionality (eg. method), should be the most common

  * <b>Integration/Functional</b> - tests feature level and more complex behavior (eg. api)

  * <b>E2E</b> - test end-to-end functionality, should be sparse and not replace the other two
]

---
.left-column[
  ## Intro
  ### Why?
  ### What?
  ### How?
  ### Extras
]

.right-column[
  ### Tape

  * Lightweight testing framework

  * Makes basic (good) decisions for you

  * Example
  ```javascript
    test('timing test', (t) => {
      t.plan(2)

      t.equal(typeof Date.now, 'function')
      const start = Date.now()

      setTimeout(() => t.equal(Date.now() - start, 100), 100)
    })
  ```

.footnote[[Why I use Tape instead of Mocha and so should you (Eric Elliot)](https://medium.com/javascript-scene/why-i-use-tape-instead-of-mocha-so-should-you-6aa105d8eaf4#.9p8frcu8g)]
]

---
.left-column[
  ### Intro
  ## Why?
  ### What?
  ### How?
  ### Extras
]
.right-column[
  ### Why should we test?
  * Maintainable code
  * Definition of what the code should do
  * Examples built in to our library
  * CI/CD
]
---
.left-column[
  ### Intro
  ### Why?
  ## What?
  ### How?
  ### Extras
]
.right-column[
  ### What are we testing?

  * Find out what *exactly* we'd like to test
  * Don't test our dependent libraries - they should be tested by their developers
  * If testing integration with external code, test ONLY the integration
]
---
.left-column[
  ### Intro
  ### Why?
  ## What?
  ### How?
  ### Extras
]
.right-column[
  ## Guidelines
  * Should always produce the same results if run several times consecutively (no flaky tests)
  * Should not be concurrent unless we want to test concurrency in a specific case
  * Should not repeat themselves
]
---
.left-column[
  ### Intro
  ### Why?
  ## What?
  ### How?
  ### Extras
]
.right-column[
  They should not rely on other tests or on test order

  Bad:
  ```javascript
  let counter = 0

  test('first call to endpoint succeeds', async t => {
    t.plan(1)
    try {
      const response = await RP('/my/awesome/incrementor')
      counter += 1
      t.equal(counter, response, 'response initial value is 1')
    } catch (e) {
      t.fail(e.message)
    }
  })

  test('second call to endpoint succeeds', async t => {
    t.plan(1)
    try {
      const response = await RP('/my/awesome/incrementor')
      counter += 1
      t.equal(counter, response, 'response incremented by 1')
    } catch (e) {
      t.fail(e.message)
    }
  })
  ```
]
---
.left-column[
  ### Intro
  ### Why?
  ## What?
  ### How?
  ### Extras
]
.right-column[
  They should not rely on other tests or on test order

  Better:
  ```javascript

  test('call to endpoint succeeds', async t => {
    t.plan(1)
    try {
      const firstResponse = await RP('/my/awesome/incrementor')
      t.equal(firstResponse, 1, 'response initial value is 1')
      const secondResponse = await RP('/my/awesome/incrementor')
      t.equal(secondResponse, 2, 'response value incremented by 1')
    } catch (e) {
      t.fail(e.message)
    }
  })
  ```
]
---
.left-column[
  ### Intro
  ### Why?
  ## What?
  ### How?
  ### Extras
]
.right-column[
  ## Should be simple, short and 'easy to reason about'

  Bad:
  ```javascript
  test('complex scenario', async t => {
    t.plan(3)
    const created = await callCreateUser()
    let existing = await callGetUser(created.id)
    t.equal(created.name, existing.name, 'can get created user')
    const changed = await callChangeUser(existing.id, {name: 'foo'})
    existing = await callGetUser(created.id)
    t.equal(
      existing.name,
      changed.name,
      'name changed successfully'
    )
    t.notEqual(
      existing.name,
      created.name,
      'name changed successfully'
    )
  })
  ```
]
---
.left-column[
  ### Intro
  ### Why?
  ## What?
  ### How?
  ### Extras
]
.right-column[
  ## Should be simple, short and 'easy to reason about'
  Better:
  ```javascript
  test('create user', async t => {
    t.plan(1)
    const created = await callCreateUser()
    const existing = await callGetUser(created.id)
    t.equal(created.statusCode, 201, 'proper status code returned')
    t.equal(created.name, existing.name, 'can get created user')
  })

  test('change user', async t => {
    t.plan(1)
    const created = await callCreateUser()
    const changed = await callChangeUser(created.id, {name: 'foo'})
    t.equal(created.name, 'foo', 'name changed successfully')
  })

  test('changed user does not match existing', async t => {
    t.plan(1)
    const created = await callCreateUser()
    const changed = await callChangeUser(created.id, {name: 'foo'})
    const existing = await callGetUser(created.id)
    t.notDeepEqual(changed, existing, 'changed user does not match existing')
  })
  ```
]
---
.left-column[
  ### Intro
  ### Why?
  ### What?
  ## How?
  ### Extras
]
.right-column[
  ## Guidelines
  * Tests should be structured as one or more try catch blocks (or syntactic equivalent)

  * Tests should use assertion counting to make sure all cases ran (t.plan())

  * Tests should be understandable from their output (test names + assertion messages)
]
---
.left-column[
  ### Intro
  ### Why?
  ### What?
  ## How?
  ### Extras
]
.right-column[
  ## Always check for proper error messages:

  Bad:
  ```javascript
  test('cannot call endpoint with bad data', async t => {
    t.plan(1)
    try {
      await RP({
        uri: '/my/awesome/endpoint',
        method: 'POST',
        body: {
          number: 'foo'
        }
      })
      // If the call succeeds, the test will hang
    } catch (e) {
      t.ok(e, 'throws an error message')
      // Any sort of error would make our test pass
    }
  })
  ```
]
---
.left-column[
  ### Intro
  ### Why?
  ### What?
  ## How?
  ### Extras
]
.right-column[
  ## Always check for proper error messages:

  Better:
  ```javascript
  test('cannot call endpoint with bad data', async t => {
    t.plan(1)
    try {
      await RP({
        uri: '/my/awesome/endpoint',
        method: 'POST',
        body: {
          number: 'foo'
        }
      })
      t.fail('managed to call endpoint with invalid number')
    } catch (e) {
      t.equal(
        e.message,
        'foo is an invalid number',
        'proper error message received
      )
    }
  })
  ```
]
---
.left-column[
  ### Intro
  ### Why?
  ### What?
  ## How?
  ### Extras
]
.right-column[
  ## Exceptions should be reported as failures
  ```javascript
  test('endpoint should return proper data', async t => {
    t.plan(1)
    try {
      const response = await RP({
        uri: '/my/awesome/endpoint',
        method: 'GET'
      })
      t.equals(
        response,
        'my data',
        'proper data returned from endpoint'
      )
    } catch (e) {
      t.fail(e.message)
    }
  })
  ```
]
---
.left-column[
  ### Intro
  ### Why?
  ### What?
  ### How?
  ## Extras
]
.right-column[
  ### Coverage
  * Aspire to 100%
  * If we don't know how to test it, rewrite it
  * Run coverage as part of our test suite, every time
  * Tests should fail if coverage goes below threshold
]
---
.left-column[
  ### Intro
  ### Why?
  ### What?
  ### How?
  ## Extras
]
.right-column[
  ### Mocks / Stubs
  * Use to cut off external non-tested functionality
  * Make tests predictable
  ```javascript
    const proxyQuire = require('proxyQuire')
    const mysqlMock = {
      createConnection: () => {
        return {
          query: (queryString) => {
            return Promise.resolve([
              {id: 1, name: 'foo'},
              {id: 2, name: 'bar'}
            ])
          }
        }
      }
    }
    test('can get user names', async t => {
      // ...
        const nameService = proxyQuire('../', {mysql: mysqlMock})
        const names = await nameService.getAllNames()
        t.deepEquals(names, ['foo', 'bar'])
      // ...
    })
  ```
]
---
.left-column[
  ### Intro
  ### Why?
  ### What?
  ### How?
  ## Extras
]
.right-column[
  ### Fixtures
  * Use to create state data for tests and then tear it down
  ```javascript
    async function fixtures (names, friends) {
      await names
        .forEach(async f => await createUser(f))
      await friends
        .forEach(async f => await createFriendConnection(f))
    }

    test('can remove friend from user', async t => {
      await fixtures([foo, bar], [{foo: 'bar'}, {bar: 'foo'}])
      await removeFriend('foo', 'bar')
      const currentFriends = await getFriends('foo')
      t.equals(
        currentFriends.length,
        0,
        'friend successfully removed'
      )
    })
  ```
]
---
.left-column[
  ### Intro
  ### Why?
  ### What?
  ### How?
  ## Extras
]
.right-column[
  ### Spies
  * Use to make sure a function was called
  ```javascript
    const sinon = require('sinon')

    test('middleware is called only once', async t => {
      const myApp = require('../')
      const fakeMiddleware = sinon.spy()
      myApp.use(fakeMiddleware)
      myApp.get()
      t.ok(
        fakeMiddleware.calledOnce(),
        'middleware called only once'
      )
    })
  ```
]
---

class: center, middle
# Questions?

.footnote[https://github.com/imsnif/presentation-testing]
