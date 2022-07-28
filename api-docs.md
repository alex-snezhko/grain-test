---
title: grain-test
---

A simple testing framework for grain

version v0.1.0

### Grain-test.**TestSuiteItem**

```grain
enum TestSuiteItem<a> {
  BeforeEach((() -> Void)),
  AfterEach((() -> Void)),
  BeforeAll((() -> Void)),
  AfterAll((() -> Void)),
  Test(String, (() -> Void)),
  TestMultiple(String, List<a>, (a -> Void)),
}
```

An `enum` of all of the possible values that can be included in a test suite.

Use `BeforeEach` to run a function before each test in the suite.

Use `AfterEach` to run a function after each test in the suite.

Use `BeforeAll` to run a function before running the tests in the test suite. Note: `BeforeAll` will run before the first `BeforeEach` if both are given.

Use `AfterAll` to run a function after running all the tests in the test suite. Note: `AfterAll` will run after the last `AfterEach` if both are given.

Use `Test` to run a test as part of a test suite. Behavior is similar to the standalone `test` function.

Use `TestMultiple` to run a test function against multiple inputs. Behavior is similar to the standalone `testMultiple` function.

### Grain-test.**testSuite**

```grain
testSuite : (String, List<TestSuiteItem<a>>) -> Void
```

Run a test suite, defined by a list of `TestSuiteItem`s that are run as part of the suite.

Parameters:

|param|type|description|
|-----|----|-----------|
|`testSuiteName`|`String`|the name of the test suite|
|`testSuiteItem`|`List<TestSuiteItem<a>>`|a list of test suite items that encompass the test suite|

### Grain-test.**test**

```grain
test : (String, (() -> a)) -> Bool
```

Run a single test case, consisting of zero or more assertions

Parameters:

|param|type|description|
|-----|----|-----------|
|`testName`|`String`|a description of what the test is doing|
|`runTest`|`() -> a`|a function that runs the test case|

### Grain-test.**testMultiple**

```grain
testMultiple : (String, List<a>, (a -> b)) -> Bool
```

Run a test with a number of different inputs and expected outputs

Parameters:

|param|type|description|
|-----|----|-----------|
|`testMultName`|`String`|a description of what the test is doing|
|`runs`|`List<a>`|a list of data to run the test against. Each item will get passed to the test function and can then be referenced in the test|
|`runTest`|`a -> b`|a function that runs the test case|

### Grain-test.**AssertionInfo**

```grain
record AssertionInfo {
  passed: Bool,
  computeFailMsg: () -> String,
}
```

Info about the status of an assertion made during a test

### Grain-test.**equals**

```grain
equals : a -> a -> AssertionInfo
```

A matcher creator function that checks if two values are equal to each other

Parameters:

|param|type|description|
|-----|----|-----------|
|`other`|`a`|the value the matcher will compare against|

Returns:

|type|description|
|----|-----------|
|`a -> AssertionInfo`|a matcher that succeeds if the value being matched against is equal to the value given to create the matcher|

### Grain-test.**notEquals**

```grain
notEquals : a -> a -> AssertionInfo
```

A matcher creator function that checks if two values are not equal to each other

Parameters:

|param|type|description|
|-----|----|-----------|
|`other`|`a`|the value the matcher will compare against|

Returns:

|type|description|
|----|-----------|
|`a -> AssertionInfo`|a matcher that succeeds if the value being matched against is not equal to the value given to create the matcher|

### Grain-test.**isTrue**

```grain
isTrue : Bool -> AssertionInfo
```

A matcher function that checks if a value is `true`

Returns:

|type|description|
|----|-----------|
|`AssertionInfo`|a matcher that succeeds if the value being matched against is `true`|

### Grain-test.**isFalse**

```grain
isFalse : Bool -> AssertionInfo
```

A matcher function that checks if a value is `false`

Returns:

|type|description|
|----|-----------|
|`AssertionInfo`|a matcher that succeeds if the value being matched against is `false`|

### Grain-test.**isNone**

```grain
isNone : Option<a> -> AssertionInfo
```

A matcher function that checks if an `Option` is `None`

Returns:

|type|description|
|----|-----------|
|`AssertionInfo`|a matcher that succeeds if the `Option` value being matched against is `None`|

### Grain-test.**isSome**

```grain
isSome : Option<a> -> AssertionInfo
```

A matcher function that checks if an `Option` is contentful i.e. the `Some` variant

Returns:

|type|description|
|----|-----------|
|`AssertionInfo`|a matcher that succeeds if the `Option` value being matched against is contentful|

### Grain-test.**isOk**

```grain
isOk : Result<a, b> -> AssertionInfo
```

A matcher function that checks if a `Result` is the `Ok` variant

Returns:

|type|description|
|----|-----------|
|`AssertionInfo`|a matcher that succeeds if the `Result` value being matched against is the `Ok` variant|

### Grain-test.**isErr**

```grain
isErr : Result<a, b> -> AssertionInfo
```

A matcher function that checks if a `Result` is the `Err` variant

Returns:

|type|description|
|----|-----------|
|`AssertionInfo`|a matcher that succeeds if the `Result` value being matched against is the `Err` variant|

### Grain-test.**not**

```grain
not : (a -> AssertionInfo) -> a -> AssertionInfo
```

A matcher creator that checks the opposite of the given matcher

Parameters:

|param|type|description|
|-----|----|-----------|
|`matcher`|`a -> AssertionInfo`|a matcher to check the success of|

Returns:

|type|description|
|----|-----------|
|`a -> AssertionInfo`|a matcher that succeeds if the given matcher fails|

### Grain-test.**both**

```grain
both : ((a -> AssertionInfo), (a -> AssertionInfo)) -> a -> AssertionInfo
```

A matcher creator function that checks if two matchers both succeed

Parameters:

|param|type|description|
|-----|----|-----------|
|`first`|`a -> AssertionInfo`|the first matcher to check the success of|
|`second`|`a -> AssertionInfo`|the second matcher to check the success of|

Returns:

|type|description|
|----|-----------|
|`a -> AssertionInfo`|a matcher that succeeds if both matchers succeed|

### Grain-test.**either**

```grain
either : ((a -> AssertionInfo), (a -> AssertionInfo)) -> a -> AssertionInfo
```

A matcher creator function that checks if either of two matchers succeed

Parameters:

|param|type|description|
|-----|----|-----------|
|`first`|`a -> AssertionInfo`|the first matcher to check the success of|
|`second`|`a -> AssertionInfo`|the second matcher to check the success of|

Returns:

|type|description|
|----|-----------|
|`a -> AssertionInfo`|a matcher that succeeds if either of the two matchers succeed|

### Grain-test.**all**

```grain
all : List<a -> AssertionInfo> -> a -> AssertionInfo
```

A matcher creator function that checks if all of the given matchers succeed

Parameters:

|param|type|description|
|-----|----|-----------|
|`matchers`|`List<a -> AssertionInfo>`|the list of matcher to check the success of|

Returns:

|type|description|
|----|-----------|
|`a -> AssertionInfo`|a matcher that succeeds if all of the given matchers succeed|

### Grain-test.**any**

```grain
any : List<a -> AssertionInfo> -> a -> AssertionInfo
```

A matcher creator function that checks if any of the given matchers succeed

Parameters:

|param|type|description|
|-----|----|-----------|
|`matchers`|`List<a -> AssertionInfo>`|the list of matcher to check the success of|

Returns:

|type|description|
|----|-----------|
|`a -> AssertionInfo`|a matcher that succeeds if any of the given matchers succeed|

### Grain-test.**binaryMatcher**

```grain
binaryMatcher : ((a, a) -> AssertionInfo) -> a -> a -> AssertionInfo
```

A matcher creator creator (yes, you read that right) that defines a matcher creator based on custom matching logic


 receives both the value being matched against and the value the returned matcher creator is invoked with.
 This function should return an AssertionInfo object

Parameters:

|param|type|description|
|-----|----|-----------|
|`runFn`|`(a, a) -> AssertionInfo`|a function to run to determine the success of the matcher;|

Returns:

|type|description|
|----|-----------|
|`a -> a -> AssertionInfo`|a matcher creator that can be given a value to match against|

### Grain-test.**unaryMatcher**

```grain
unaryMatcher : (a -> AssertionInfo) -> a -> AssertionInfo
```

A matcher creator that defines a matcher based on custom matching logic


 receives the value being matched against. This function should return an AssertionInfo object

Parameters:

|param|type|description|
|-----|----|-----------|
|`runFn`|`a -> AssertionInfo`|a function to run to determine the success of the matcher;|

Returns:

|type|description|
|----|-----------|
|`a -> AssertionInfo`|a matcher creator that can be given a value to match against|

### Grain-test.**assertThat**

```grain
assertThat : (a, (a -> AssertionInfo)) -> Void
```

Asserts that a value fulfills a given matcher (note: does not use the native grain `assert`)

Parameters:

|param|type|description|
|-----|----|-----------|
|`value`|`a`|a value to match against|
|`matcher`|`a -> AssertionInfo`|a matcher to apply to the value|

### Grain-test.**assertWithMsgThat**

```grain
assertWithMsgThat : (String, a, (a -> AssertionInfo)) -> Void
```

Asserts that a value fulfills a given matcher, with a message to display upon failure (the message is prepended with `"Expected that "` in output).
(note: does not use the native grain `assert`)

Parameters:

|param|type|description|
|-----|----|-----------|
|`message`|`String`|a message to be printed in the case that the assertion fails|
|`value`|`a`|a value to match against|
|`matcher`|`a -> AssertionInfo`|a matcher to apply to the value|

