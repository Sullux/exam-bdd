# exam-bdd

A BDD test utility for exam and other major Node.js test runners. To install, 
run this in the terminal:

```bash
npm install exam-bdd --save-dev
```

## Quick Start Guide

Reading through this quick start guide should give you the basics you need to 
start writing tests. A more thorough discussion can be found below. For these
examples, we will assume that in addition to the test file, there is a code
file in the same directory called `widget.js` containing the following (very
useless) code:

```javascript
module.exports = function() {
  var args = Array.prototype.slice.call(arguments, 0)
  if(!args.length) {
    return undefined
  }
  if(args.length === 1) {
    return [args[0]]
  }
  throw new Error('Can only be called with zero or one arguments.')
}
```

### Simple Test with Inline Functions

A simple test with inline functions. Notice that the BDD syntax integrates 
seamlessly with Exam's define/it syntax. Inline functions are not the 
recommended way to write tests, but this illustrates how the basic syntax works.

```javascript
/* global describe feature scenario is */
require('exam-bdd')
var widget = require('./widget')

describe('widget', function() {
  feature('spin', function() {
    scenario('with no arguments, spins without error', function() {
      this
        .given('an empty widget', function() { this.widget = widget() })
        .when('the widget is spun', function() {
          try {
            this.result = widget.spin()
          } catch (err) {
            this.error = err
          }
        })
        .then('there is no error', function() { is.falsy(this.error) })
        .and('there is an empty result', function() { is(this.result, undefined) })
    })
  })
})
```

### Simple Test with Extracted Functions

Here is the same test, but with the functions extracted. Notice how once the 
functions are extracted, they become reusable.

```javascript
/* global describe feature scenario is */
require('exam-bdd')
var widget = require('./widget')

/* TEST DEFINITIONS */

describe('widget', function() {
  feature('spin', function() {
    scenario('with no arguments, spins without error', function() {
      this
        .given('an empty widget', anEmptyWidget)
        .when('the widget is spun', widgetIsSpun)
        .then('there is no error', thereIsNoError)
        .and('there is an empty result', thereIsAnEmptyResult)
    })
  })
})

/* EXTRACTED FUNCTIONS */

function anEmptyWidget() {
  this.widget = widget()
}

function widgetIsSpun() {
  try {
    this.result = widget.spin()
  } catch (err) {
    this.error = err
  }
}

function thereIsNoError() {
  is.falsy(this.error)
}

function thereIsAnEmptyResult() {
  is(this.result, undefined)
}
```

### Multiple Tests with Extracted Functions

Once the functions are extracted, they can be reused. See how this next example
adds more tests but reuses much of the same logic as the first test. This makes
tests more DRY and easy to write and maintain.

```javascript
/* global describe feature scenario is */
require('exam-bdd')
var widget = require('./widget')

/* DEFINITIONS */

describe('widget', function() {
  feature('spin', function() {
    scenario('with no arguments, spins without error', function() {
      this
        .given('an empty widget', anEmptyWidget)
        .when('the widget is spun', widgetIsSpun)
        .then('there is no error', thereIsNoError)
        .and('there is an empty result', thereIsAnEmptyResult)
    })

    scenario('with one argument, spins without error', function() {
      this
        .given('an empty widget', anEmptyWidget)
        .and('one argument', oneArgument)
        .when('the widget is spun', widgetIsSpun)
        .then('there is no error', thereIsNoError)
        .and('there is a single result', thereIsASingleResult)
    })
  })
})

/* EXTRACTED FUNCTIONS */

function anEmptyWidget() {
  this.widget = widget()
}

function oneArgument() {
  this.arguments = ['foo']
}

function widgetIsSpun() {
  try {
    this.result = widget.spin.apply(widget, this.arguments || [])
  } catch (err) {
    this.error = err
  }
}

function thereIsNoError() {
  is.falsy(this.error)
}

function thereIsAnEmptyResult() {
  is(this.result, undefined)
}

function thereIsASingleResult() {
  is.array(this.result)
  is(this.result.length, 1)
}
```

### Parameterized Functions

Sometimes it is useful to reuse a function for various inputs or expected
outputs. For example, rather than this:

```javascript
function thereIsAnEmptyResult() {
  is(this.result, undefined)
}

function thereIsASingleResult() {
  is.array(this.result)
  is(this.result.length, 1)
}
```

it might be more convenient to do something like this:

```javascript
function theResultIs(value) {
  // NOTE: this does not work!!!
  is.same(this.result, value)
}
```

The problem is that the above function would be called during test setup rather
than being called during test execution. While this line of code tells the
engine which function to use _in the future when running the test_:

```javascript
this.then('there is an empty result', thereIsAnEmptyResult)
```

this code will execute immediately during setup and will not hand a function to
the engine to use when running the test.

```javascript
this.then('the result is undefined', theResultIs(undefined))
```

Since the engine needs to be handed a function, the correct way to write a
parameterized function is like this:

```javascript
function theResultIs(value) {
  return function() {
    is.same(this.result, value)  
  }
}
```

Now this code will work properly:

```javascript
this.then('the result is undefined', theResultIs(undefined))
```

because the `theResultIs(undefined)` call is returning a function for use by the
test.

## Background

Behavior Driven Development, or BDD, is often expressed using the Gherkin 
language. Gherkin uses a given/when/then syntax to define scenarios for 
features. 

TODO