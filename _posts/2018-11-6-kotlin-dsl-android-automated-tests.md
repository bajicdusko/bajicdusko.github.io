---
layout: "post"
title: "Kotlin DSL for Espresso and UIAutomator"
date: "2018-11-06 13:00"
categories: [android]
tags: [android, test, kotlin]
description: "Android automation tests assume Espresso and UIAutomator. This DSL unifies them into unique syntax."
---
On Android there aren’t many choices when it comes to which library to use. Obvious choices are [Espresso](https://developer.android.com/training/testing/espresso/) and [UIAutomator](https://developer.android.com/training/testing/ui-automator). Although the libraries can be used for the same goal, there are crucial differences between them. Shortly:

- Espresso is a white box solution for Android testing, sandboxed for the usage of testing the current application.
- UIAutomator is a black box testing solution, which is running on the device level, giving us the capabilities of navigating outside of the currently tested application.

For the purpose of end-to-end testing, we needed both libraries in order to pull out the best of both worlds and to implement proper automated testing of the app.

When the UI testing is implemented, the library syntax is never simple nor pretty. Having two libraries working together, with completely different syntax, makes readability and maintainability quite hard to achieve.

For example, let’s see how could we execute a click action on the UI component.

## Espresso

In case of Espresso, we’re working with three types of objects. Matchers, ViewActions and ViewAssertions. Following the syntax and combining these three objects, we would implement click action like this:

```java
Espresso.onView(Matchers.withId(R.id.activityLoginBtnSubmit)).perform(ViewActions.click())
```

## UIAutomator

In case of UIAutomator, it’s much more complicated. There are a couple of preconditions to make in order to query the UI hierarchy for the specific object.

1. Getting the Context from the InstrumentationRegistry
2. Converting the resource ID into the resource name (we’ll need this later)
3. Creating the UIDevice object, which is a God object for the UIAutomator. Almost each call requires UIDevice instance.
4. Defining the UISelector. In our case, we want to create a UISelector to query the UI component by resource ID, but such a method doesn’t exist in UIAutomator. There is something similar (via resource name), so we’ll use it (that’s why 2).
5. Instantiating the UIObject by using the UIDevice and UISelector. Having the UIObject gives us the possibility to interact with the UIComponent.

```java
val instrumentation = InstrumentationRegistry.getInstrumentation()
val uiDevice = UiDevice.getInstance(instrumentation)
val appContext = InstrumentationRegistry.getInstrumentation().targetContext

val loginButtonSelector = UiSelector().resourceId(appContext.resources.getResourceName(
        R.id.activityLoginBtnSubmit
    )
)

val loginButton = uiDevice.findObject(loginButtonSelector)
loginButton.click()
```

Now imagine combining these two and making 10-15 navigations through the app in order to check some view located deep in the hierarchy…
Yes, maintainability is equal to **zero**. Readability is equal to **headache**.

## The DSL idea

We’ve recognised this problem right in the beginning and decided to use the power of Kotlin to write a DSL which unifies the syntax for both libraries. An additional benefit we get is an easier handover of the codebase to the new colleague, as the DSL syntax is more logical and simpler to grasp.

```kotlin
click on button(R.id.activityLoginBtnLogin)
```

Just by looking into the existing examples, writing the new tests shouldn’t be a hurdle at all. We proved this by introducing the new colleague into the team, inexperienced in Kotlin, and he started writing production ready tests in a matter of week. Yay!  
Extending the DSL proved to be a simple as well.  
The syntax we come up with should be human readable as much as possible and hopefully we succeeded. It’s identical for both, Espresso and UIAutomator. It just depends of what function you imported on first use in the class.

### The library

During development, we’ve had to use more and more actions and assertions on the UI components, so the DSL grew over time. At one point it became problematic to maintain that collection of functions as well, so we had to organise it and make it independent of the current application we were testing. The library is born.

[Android Test KTX](https://github.com/codecentric/androidtestktx) is a resulting library which is deployed to the GitHub and open sourced under Apache 2.0 license for the public use.

### Internal organisation

As everything goes in pairs now, so does the library organisation:

![Kotlin DSL library organisation]({{ site.url }}/assets/img/posts/2018/2018-11-06-kotlin-dsl-library-structure.png){: width="190px"}

Functions are split into two main packages called espresso and uiautomator. Further, each package has an _Actions.kt_, _Assertions.kt, Matchers.kt_ and {library}_Extensions.kt_.

The Actions file contains functions which execute some action on the UI component. _click_, _typeText_, _scroll_.  
Matchers file, contains functions for finding the UI component on the screen. _viewById_, _viewByText_.  
The Assertions file contains functions for checking the state of the UI component. itIsDisplayed, itIsEnabled.

### infix notation

```kotlin
click on button(R.id.activityLoginBtnSubmit)
```

or

```kotlin
typeText("dummyUsername") into text(R.id.activityLoginEtUsername)
```
are both written in the infix notation, where the `on` and `into` are infix extension functions.

These expressions could be written like this as well:

```kotlin
viewById(R.id.activityLoginBtnSubmit).click()
```

```kotlin
viewById(R.id.activityLoginEtUsername).typeText("dummyUsername")
```

We’ve left it to be a personal choice of the library user.

`on` and into functions have an identical implementation, and we created `onto` function as well. Their purpose is to increase the readability and semantical meaning of the expression.  
We want to click `on` something, type `into` some field or hold `onto` something else.

```kotlin
infix fun ViewAction.on(matcher: Matcher) {
  Espresso.onView(matcher).perform(this)
}
```
`view`, `text`, `field` and `button` are also the functions with the identical implementation, doing the same thing as `viewById` it should improve the semantical meaning of the expression.

This DSL proved to be a working solution, which simplifies and accelerates the testing process. Personal satisfaction shouldn’t be excluded either, as writing the concise and readable code make you feel better. Me at least. Time spent on debugging and understanding the test is brought down to minimum. What’s more important, if you’re using this library, knowing full UIAutomator or Espresso logic isn’t necessary any more, or at least it shouldn’t be mandatory.

### Verification

Finding the UI component and making an interaction with it brings us a half way to the goal. With the verification of the UI component state, we’re completing the test. We’ve introduced the `verifyThat` infix extension function to cover the readability of the Assertion part.

```kotlin
infix fun Matcher.verifyThat(func: () -> ViewAssertion) {
  onView(this).check(func())
}
```

This function influenced the naming of the assertion functions greatly, as we constantly have a semantical meaning of the expression in mind.

```kotlin
val usernameField = viewById(R.id.activityLoginEtUsername)
typeText("dummyUsername") into usernameField
click on button(R.id.activityLoginBtnSubmit)

usernameField verifyThat { itIsNotEnabled() }
```

### Usage

This library is deployed onto the JCenter and can be added into the project by adding the line below into the `build.gradle` file.

```gradle
androidTestImplementation 'de.codecentric:androidtestktx:$latestVersion'
```

### Friendly advice
 
- Although the UIAutomator worked quite well for us, it was the cause of most headaches as well. Espresso proved to be a better solution. So the future library development will follow the Espresso cheat sheet and implement the same actions for the UIAutomator. Library version 1.0.0 should bring separate artefacts for Espresso and UIAutomator.
- Using the DSL in combination with the Robot pattern is strongly encouraged. The power of the Robot pattern combined with Kotlin is described in a separated blog post, [Automated tests in Android with Robot pattern]({{ site.url }}/posts/2018/11/android-automated-tests-with-robot-pattern). Here is a sneak peek:

```kotlin
@Test
fun shouldLoginTest() {
  withLoginRobot {
    initiateTheLogin()
  } andThen {
    acceptThePermissions()
  } andThenVerifyThat {
    userIsLoggedIn()
  }
}
```
