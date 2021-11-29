---
layout: "post"
title: "Android automated tests with Robot pattern (in Kotlin)"
date: "2018-11-07 13:00"
categories: [android]
tags: [android, test, kotlin]
description: "Writing powerful automated tests for Android has never been easier. This is how to utilize AndroidTestKTX with Robot pattern."
redirecturl: "https://www.crispy-engineering.com/android-automated-tests-with-robot-pattern-in-kotlin/"
---
In [previous blog post]({{ site.url }}/posts/2018/11/kotlin-dsl-android-automated-tests/), we have discussed and shown a [Kotlin DSL library](https://github.com/codecentric/androidtestktx) we developed for the Espresso and UIAutomator, as we noticed that readability and maintainability are endangered and made a necessary steps. However, the DSL is only one step into the right direction.

The DSL doesn’t solve the separation of concerns at all. Improvement of the readability was still needed. We addressed these issues with the Robot pattern. As a beneficial side-effect, we hide the library and the DSL syntax completely.

Hint: This is what we want to achieve. Through this post I'll guide you one function at a time to it.
```kotlin
@Test
fun shouldLoginToTheApp() {
  withLoginRobot {
    login("john_smith", "p@$$w0rd")
  } andThen {
    acceptTermsOfUse()
  } andThenWithPermissionRobot {
    acceptAllPermissions()
  } andVerifyThat {
    userIsLoggedIn()
  }
}
```

## What is the Robot pattern? And for that matter, what is the robot?

Robot is a simple class dedicated to one screen in the application. It contains the implementation of use cases for the interaction with the UI components of that screen only. We’re making the robot which mimics the user interaction for a single screen.

Let’s take a simple login screen with username and password fields and login button as an example. For this login screen, we would create a `LoginRobot` class. In this class we will implement a single function: 

```kotlin
class LoginRobot {
  fun login(username: String, password: String) { 
    //finds all UI elements, interact and log in user.
  }
}
```

When used in the test, we will instantiate the `LoginRobot` class and call a `login` function, providing the username and password. So we will have something like this:

```kotlin
@Test
fun shouldLoginToTheApp() {
  val loginRobot = LoginRobot()
  loginRobot.login("john_smith", "p@$$w0rd")
}
```

However, the snippet above doesn’t do any assertion, therefore, the test is pointless. Also, implementing the assertion directly in the test doesn’t make too much sense, as we’re trying to extract the logic into Robot-like classes.

## What is a RobotResult?

Usually, each Robot has its own RobotResult class. RobotResult is a class which holds the assertions, per use case, for a single screen. In our case, besides `LoginRobot`, we will have a `LoginRobotResult`.

Our LoginRobotResult will have a function `isLoggedIn()`.

```kotlin
class LoginRobotResult {
  fun isLoggedIn() {
    //contains the assertion for login status
  }
}
```

Similarly, as with LoginRobot, we have to instantiate the LoginRobotResult and use it in the test.

```kotlin
@Test
fun shouldLoginToTheApp() {
  val loginRobot = LoginRobot()
  loginRobot.login("john_smith", "p@$$w0rd")
  
  val loginRobotResult = LoginRobotResult()
  loginRobotResult.isLoggedIn() 
}
```

Comparing to the direct approach, where we find the UI elements in the test function body and implement the interaction and assertion one below the other, this looks much better and understandable. But, we wouldn’t be here just to show you simple separation and wrapping the logic into two classes, right?

Below, we’ll show you how to improve the readability with Kotlin infix notation.

## Kotlin infix, extension and higher-order functions in a mission to boost the readability to max

By moving the interaction logic into the Robot class and by moving the assertion logic into the Robot result class, we made necessary steps to improve the maintainability. We have basically applied the separation of concerns principle on the test.

To improve the readability, as a first step we might avoid direct instantiation of classes in the test example above. Instead, by creating a top level higher-order function, we will shorten the login interaction to single expression only. The function we will create is called `withLoginRobot` (this naming convention increases the semantic of test body). This function creates a LoginRobot instance and accepts the lambda in the context of LoginRobot.

```kotlin
fun withLoginRobot(fn: LoginRobot.() -> Unit): LoginRobot = LoginRobot().apply(fn)
```

The test looks more readable now:

```kotlin
@Test
fun shouldLoginToTheApp() {
  withLoginRobot {
    login("john_smith", "p@$$w0rd")
  }
  
  val loginRobotResult = LoginRobotResult()
  loginRobotResult.isLoggedIn() 
}
```

With the identical approach, we can create a function called `verifyThat` to instantiate a LoginRobotResult class.

```kotlin
fun verifyThat(fn: LoginRobotResult.() -> Unit): LoginRobotResult = LoginRobotResult(fn)
```

Which improves the test a little bit as well:

```kotlin
@Test
fun shouldLoginToTheApp() {
  withLoginRobot {
    login("john_smith", "p@$$w0rd")
  }
  
  verifyThat {
    isLoggedIn()
  }
}
```

Although this looks cool, there is more space for improvement. By using the infix notation and making our `verifyThat` function an extension function of LoginRobot, we’ll be able to make a sequential call and single expression of the whole test content.

```kotlin
infix fun LoginRobot.verifyThat(fn: LoginRobotResult.() -> Unit): LoginRobotResult
  = LoginRobotResult(fn)
```

Finally, we have a desired look of our test.

```kotlin
@Test
fun shouldLoginToTheApp() {
  withLoginRobot {
    login("john_smith", "p@$$w0rd")
  } verifyThat {
    isLoggedIn()
  }
}
```

For the sake of readability, additionally we can rename `verifyThat` into `andVerifyThat` and rename `isLoggedIn` into `userIsLoggedIn()`. This is a subjective decision, however, we can read this test very easily, in a natural, human-readable way:

> “With login Robot, login John Smith and verify that user is logged in”.

On the first read, it’s very understandable what this test do and that’s exactly what we want to achieve.

## UI interaction in multiple steps

UI tests with only one interaction step are very rare. Usually, we have to do multiple navigation steps throughout the app, to put it into the desired state ahead of the assertion.  
For example, first we have to login, then we have to accept the terms of service and then to accept required permissions.

Let me introduce you the `andThen` and `andThenWith` functions.

The role of these functions is to wire the expression into a single body, with the possibility to introduce an additional interaction step with the same Robot or with some other Robot.

```kotlin
infix fun LoginRobot.andThen(fn: LoginRobot.() -> Unit): LoginRobot {
  also(fn)
}
```

or 

```kotlin
infix fun LoginRobot.andThenWithPermissionRobot(fn: PermissionRobot.() -> Unit): LoginRobot {
  PermissionRobot().apply(fn)
  return this
}
```

Whatever option we decide to use, our test will stay readable as it was:

```kotlin
@Test
fun shouldLoginToTheApp() {
  withLoginRobot {
    login("john_smith", "p@$$w0rd")
  } andThen {
    acceptTermsOfUse()
  } andThenWithPermissionRobot {
    acceptAllPermissions()
  } andVerifyThat {
    userIsLoggedIn()
  }
}
```

Isn’t this awesome 🙂 !

## Recap

With this approach, we’ve created a couple of abstraction layers for the UI testing with Robot classes and Kotlin DSL a building blocks.

- Each screen has its own Robot.
- Each robot has its own Robot result.
- `withRobotName` function is used to initialise the Robot.
- `andThen` function is used to wire the calls together and increase the semantic of the expression.
- `verifyThat` function in Robot result is used for the assertion implementation.
- A combination of infix notation with higher-order extension functions helps us create a single readable expression.
- [AndroidTestKTX](https://github.com/codecentric/androidtestktx) is used within the Robot functions to simplify the UI interaction.

Android UI testing should be fun as well and with this approach we’re a couple of steps closer.

I wish you joyful testing!