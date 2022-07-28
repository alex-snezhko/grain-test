# grain-test
A lightweight testing framework for the Grain programming language.


## Usage Guide

### Getting started

Grain-test is comprised of the core testing library: `grain-test.gr`, and a Python test runner script: `run-tests` (optional but very useful). To get started, let's jump into our project and get these files (for the sake of example, we'll assume that our tests will be located in the root of our project, although you may not always want it this way); for example, on Linux/MacOS:
```
cd /location/of/my/project
curl https://raw.githubusercontent.com/alexsnezhko3/grain-test/main/run-tests -o run-tests
chmod +x run-tests
curl https://raw.githubusercontent.com/alexsnezhko3/grain-test/main/grain-test.gr -o grain-test.gr
```


### Basic example

Now that we have what we need, let's get started! Let's assume that we want to write tests against this code:
```
// example.gr

export let square = (a) => a * a
```

Let's create a file `example.test.gr` and write our first test!
```
// import everything from the library for convenience
import * from "./grain-test"
// import our code
import Example from "./example"

// test that our code works as expected
test("our code works", () => {
  let actual = Example.square(3)
  assertThat(actual, equals(9))
})
```

We can already see a few things being used here: we are invoking the `test` function to run a test, giving it a description of what our test is doing. We use the `assertThat` function to verify our code does what we expect it to, and use the `equals(...)` matcher in our assertion which will. As the name indicates, this matcher will attempt to match the first argument given to `assertThat` to the value we give the matcher, in this case `Example.square(3)` and `9` respectively. Now let's run our test; you can either run it directly like `grain example.test.gr`, or use the test runner script, which we will be doing.

To run the test runner, first ensure that you have a valid Python 3 installation on your `PATH`. The script is "shebanged" to use Python by default, so if you are on Linux or MacOS, you can simply run:
```
./run-tests
```
If you are on Windows, you can invoke Python on the test runner script i.e.
```
python run-tests
```
If that succeeds, we'll be met with a message telling us our code passed!
```
Running ./example.test.gr
  ✓ our code works
  

Tests: 1 passed
```
By default, the test runner script will search for all files with the extension `.test.gr`, but we'll see how we can change this, as well as further customize how the script works, later.

If we have a bug in our code (say we mistakenly defined `square` as `export let square = (a) => a + a`), we'll see that our test now fails, with a description of what went wrong as caught by our assertion
```
Running ./example.test.gr
  ✗ our code works
    ● Expected 6 to equal 9
  

Tests: 1 failed
```


### Testing the same code against multiple inputs/expected outputs

We can write tests that test the same code against multiple sets of values, to avoid duplicating tests if we want to test our functions against various input/edge cases. First, let's write some code to test in a `example.gr` file:
```
// check if a value is in a range, exclusive
export let inRangeExclusive = (low, high, val) => {
  if (val <= low) {
    false
  } else if (val > high) { // Uh-oh, this should be val >= high !
    false
  } else {
    true
  }
}
```
Now, in `example.test.gr`, we can do:
```
import * from "./grain-test"
import Example from "./example"

testMultiple("test our function works against multiple inputs", [
  (0, 5, 2, true),
  (1, 3, 1, false),
  (10, 20, 30, false),
  (10, 20, 20, false) // this test will fail because our code has a bug in it
], ((low, high, val, expected)) => {
  let actual = Example.inRangeExclusive(low, high, val)
  assertThat(expected, equals(actual))
})
```
Running the test runner script `./run-tests`, we get the output:
```
Running ./example.test.gr
  ✓ test our function works against multiple inputs - run 1 (test data: (0, 5, 2, true))
  
  ✓ test our function works against multiple inputs - run 2 (test data: (1, 3, 1, false))
  
  ✓ test our function works against multiple inputs - run 3 (test data: (10, 20, 30, false))
  
  ✗ test our function works against multiple inputs - run 4 (test data: (10, 20, 20, false))
    ● Expected false to equal true
  

Tests: 3 passed, 1 failed
```
In this example, we used tuples for our test case values, but all that `testMultiple` does is pass down each value in the list to the callback function running our tests. Therefore, we just as easily could have used any other types of values; here's an example using records, clarifying the role of each value in the inputs:
```
// ...

record TestInput {
  low: Number,
  high: Number,
  val: Number,
  expected: Bool
}

testMultiple("...", [
  { low: 0, high: 5, val: 2, expected: true },
  // ...
], ({ low, high, val, expected }) => {
  // ...
})
```


### Creating test suites

We can group our tests into test suites, which allows us to better organize our tests and also enables some additional functionality. Let's define a test suite:
```
import * from "./grain-test"
// implementation elided
import MyCode from "./mycode"

testSuite("all of our code works", [
  Test("our first function works", () => {
    assertThat(MyCode.firstFunction(1, 2), equals(3))
  }),
  Test("our second function works", () => {
    // test our second function
  }),
  TestMultiple("our third function works against multiple inputs", [/* ... */], (/* ... */) => {
    // test our third function
  })
])
```
Here we see that we have grouped several tests together under one unit, specifying several actions to run in our test suite in a list. Please be aware that the `Test` and `TestMultiple` used here are **not** the same as the `test` and `testMultiple` functions we used in previous examples; the former are enum variants which allow us to register actions to be run as part of a test suite, and the latter are functions which immediately invoke a test.

Test suites not only provide organizational structure to our tests, but they also enable additional functionality for our tests, such as allowing you to specify code to be run before and/or after each test (or before/after the whole suite). This can be useful in the case that we want to run side effects which our code under test relies on to work properly. For example, if we are testing code that mutates a file, we can leverage this functionality to create/initialize the file before each test, and then wipe the file after each test. Here is an example showing all of the additional actions you can specify as part of a test suite:
```
// ...

testSuite("my code which mutates the file works", [
  BeforeEach(() => {
    // this code will run before each test
  }),
  AfterEach(() => {
    // this code will run after each test
  }),
  BeforeAll(() => {
    // this code will run before the entire suite.
  }),
  AfterAll(() => {
    // this code will run after the entire suite.
  }),
  Test("my test", () => {
    // test my code
  }),
  TestMultiple("some more tests", [/* ... */], (/* ... */) => {
    // some more tests
  })
])
```
Note that the places you put the `Before...` and `After...`s in the test suite list does not matter; the only thing to know is that if you have multiple of the same action type, they will be run in the order they appear in the list e.g. the first `BeforeEach` in the list will be run before the second `BeforeEach`.

Side note: you may be wondering: "What is the point of `BeforeAll` and `AfterAll`? Can't I just put code before and after the invokation of the `testSuite` function?" ...Well, yes; however, the existence of these actions allows us to compose our tests in complex ways more easily. For example, if we have multiple test suites across multiple files that all have the same side effects that need to be run to set up the tests, we can easily extract these actions into a list in a shared file and then append them into each of our test suites' action lists.


### More on matchers and assertions

So far, we have only been using the `equals` matcher, but there are several other matchers built in to grain test; both simple matchers that match a value against another value, and compound matchers that take other matchers as inputs! Here is a mishmash of different matchers available out of the box:
```
import * from "./grain-test"

// this test will pass
test("test a bunch of stuff", () => {
  // equality matchers
  assertThat(1, equals(1))
  assertThat("A", notEquals("B"))

  // boolean matchers
  assertThat(false, isFalse)
  assertThat(true, isTrue)

  // Option matchers
  assertThat(None, isNone)
  assertThat(Some(42), isSome)

  // Result matchers
  assertThat(Ok("hi"), isOk)
  assertThat(Err("an error"), isErr)

  // -- Compound matchers --

  // matches opposite of given matcher
  assertThat(Some(42), not(isNone))

  // matches if both matchers succeed
  assertThat(2, both(equals(2), not(equals(3))))

  // matches if either matcher succeeds
  assertThat(true, either(isTrue, isFalse))

  // matches if all matchers succeed
  assertThat(Some("abc"), all([isSome, equals(Some("abc")), notEquals(None)]))

  // matches if any matcher succeeds
  assertThat(None, any([isNone, isSome, equals(Some("abc"))]))
})
```

If the matchers available out of the box do not suit your needs, you can create your own matchers. Here is an example of creating a few custom matchers:
```
import * from "./grain-test"
import Set from "set"

// a custom matcher that checks if two lists have the same elements
let hasSameElementsAs = binaryMatcher((firstList, secondList) => {
  let passed = Set.fromList(firstList) == Set.fromList(secondList)

  // this function must return a record of type AssertionInfo, which is exported from grain-test
  {
    // an indication of whether the matcher succeeded or not
    passed,
    // a function that returns the failure message if the test fails;
    // it is prefixed by "Expected " in the output if the test fails
    computeFailMsg: () => toString(firstList) ++ " to have the same elements as " ++ toString(secondList)
  }
})

// a custom matcher that checks if a number is even
let isEven = unaryMatcher((numValue) => {
  { passed: numValue % 2 == 0, computeFailMsg: () => toString(numValue) ++ " to be an even number" }
})

test("my custom matchers work", () => {
  // we call hasSameElementsAs, a binary matcher, with a second value;
  // in this case, [1, 2, 3] will be passed to the "firstList" parameter,
  // and [2, 3, 1] to the "secondList" parameter of the matching function
  assertThat([1, 2, 3], hasSameElementsAs([2, 3, 1]))

  // we pass isEven, a unary matcher, directly to assertThat;
  // in this case, 4 will be passed to the "numValue" parameter of the matching function
  assertThat(4, isEven)
})
```

We can also add custom messages for our assertions to be displayed if they fail with `assertWithMsgThat`.
```
// ...

test("...", () => {
  assertWithMsgThat("the file is read correctly", MyCode.readFile("hello.txt"), equals(Some("Hello, world!")))
})
```


## The test runner script

`run-tests` is a simple Python script that runs all of your tests. Its default behavior can be changed with various flags.

| Description | Flag | Default | Example |
| ----------- | ---- | ------- | ------- |
| Regex to use to match test files (relative to location of script) | `-r` or `--regex` | `.+\.test\.gr$` | `./run-tests --regex .+_test\.gr` |
| Directory to start searching for tests in (relative to location of script) | `-d` or `--dir` | `.` | `./run-tests --dir ./tests` |
| Directories to exclude when searching for test files | `-e` or `--exclude-dir` | `[]` | `./run-tests --exclude-dir ./target --exclude-dir ./forbidden` |
| A flag to indicate that only failing tests should be shown | `-f` or `--only-failing` | disabled | `./run-tests --only-failing` |
| A flag to indicate that output should be given without any dressing i.e. text coloring, special unicode characters | `-p` or `--plain` | disabled | `./run-tests --plain` |
| A flag to indicate that no more tests should be ran after the first failing test | `-b` or `--bail-upon-failure` | disabled | `./run-tests --bail-upon-failure` |

## API docs
Docs for the `grain-test.gr` API can be found [here](https://github.com/alexsnezhko3/grain-test/blob/main/api-docs.md)

## Contributing
If you feel that some improvement can be made to the documentation or any other artifacts, feel free to open an issue or create a pull request!
