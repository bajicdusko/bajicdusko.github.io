---
layout: "post"
title: "Android testing: AWS Device Farm vs Firebase TestLab"
date: "2018-09-27 13:00"
categories: [android]
tags: [android, test, aws, devicefarm, firebase]
description: "Tips and tricks on the AWS DeviceFarm and Firebase TestLab."
---

A whole year passed since I started working on Android automation testing solution on the project for a large corporation. The project is handed over to another team now, and it’s time to share the valuable experience.

Our job was to automate E2E tests only. And one of the first decisions we had to make, is to choose a serious enough company which provides a service of "renting" all kind of physical devices for the testing purposes. First, we thought of self-hosting solution which could be wired to the Cl pipeline, but we could never provide a device diversity granular enough. Therefore, we started looking for cloud solutions.

Since we needed a solution which supports both, Android and iOS platforms, with a large number of various devices, AWS DeviceFarm singled out, as a solution we could trust to be stable enough, with the responsive support and general ease of use.

### AWS DeviceFarm

When using it for the first time, you will probably try out the service through the web UI. There are just a couple of mandatory steps to go through in order to start the test run:

1. Choose a test type: Instrumentation
2. Upload test apk
3. Upload app apk
4. Choose devices (create a so-called device-pool)
5. If you don’t need any extra data package to provide, then click run.

And basically, that’s it. The tests will run on a chosen devices and if everything goes well, you’ll see the cumulative pass/fail statistic per device and detailed list of passed and failed tests.

For each test, you’ll be able to get the instrumentation log, logcat and video recording by default.

However, web UI isn’t much of use when the CI pipeline is used, so we have to use either the AWS CLI or some plugin for the build server. We were using the Jenkins which has the support for AWS DeviceFarm communication (through the plugin of course).

It worked very well, at least when it comes to the tests execution. A first huge issue we stumbled upon was the lack of reporting. There is no option to add an email or emails which should receive the testing report. Actually, there is no report at all, not even a digested one which could be forwarded to the client. We were left with the option to allow the access to our AWS project so that the testing results could be checked through the web Ul.

#### JUnit4 support - Deal breaker

On the Android side, the testing procedure was complicated enough and we had to make a couple of compromises. One of them was to force a strict order of the test execution due to complex and long login procedure in the app.

In order to do so, as a first step, we’ve created a precise test suites. A handy behaviour of test suite definition on Android is that test classes will be executed in the order they are defined in the `@SuiteClasses` annotation.

As a second part, we had to order the tests inside the test classes as well, which we did with the only option available: `@FixMethodOrder` annotation.

And that was the end of the journey for us with AWS DeviceFarm because they implement `JUnit4` only partially, without any support for test suites definition, nor for the FixMethodOrder! Since we were left out of options, we had to abandon the service as we could not run the tests as we wanted to.

###Firebase TestLab
Ahead of abandoning the AWS, we had to make sure that we can find a service, still serious enough and with good support which doesn’t have those `JUnit4` limitations. We tried the Firebase and it worked.

Through the web UI the setup procedure steps are almost identical to AWS:

1. Choose a test type: (also instrumentation)
2. Upload both apk’s
3. Choose a device
4. Run.
5. Observe the test results per device and per test with access to video recording and logs.

Of course, we cannot use the web UI, so we end up using the CLI solution for Firebase: `gcloud`.

With the `gcloud` we’re able to define the type of the test, paths to the apk files, result directory and bucket on Google Cloud Storage, and the `test-target` which besides all standard options like test package or individual test, also accepts the test suite as a target, respecting the order of execution.

Although it works on a similar way as AWS DeviceFarm, Firebase TestLab relies on Google stack and therefore saves all test results into the bucket on Google Cloud Storage, which is awesome as we’re able to access the files directly.

>In order to define the custom bucket and path per test execution, a payed access to Google Cloud Storage is required. In the case of free storage usage, the test results will be saved under the testlab/random-hash directory, and removed after 90 days.
Since we could access the test results directly, we could build the testing report as we wanted to, which is something our client really liked. Still, I’d like to mention that Firebase also doesn’t provide a system reporting solution where we could create only the mailing list in order to have the results delivered. It does have a digested results per device in the console output.

#### Timeouts:

Although the Firebase gives us the possibility of test suites run, it didn’t come for free. A maximum timeout for test execution is **30 minutes**. This is more than enough for 90% of use cases, but in our case having one test suite containing all test classes proved to be a problematic solution as our E2E tests are executing 60+ minutes.

The reason behind this 30 min limit, according to the discussions on Google forums and Slack, is the system stability, so they found the compromise on 30 min. Executing anything longer than that would be interrupted without any results.

We solved this problem by creating many small test suites which are executed one after another, with separate `gcloud` calls. As a consequence, report generation was even more complex, but possible at least, giving us the working solution at the end.

### Comparison:
Here we’ll try to sum up the pros and cons from both services:

1. CLI simplicity: Firebase
2. Plugin accessibility: AWS
3. System reports(mailing list): None
4. Reports accessibility: Firebase
5. Digested report: Firebase
6. Devices choice: AWS (Firebase have a 15-20 different devices)
7. Up-to-date compatibility: Firebase
8. Support accessibility: Firebase (almost instant via Slack)
9. Device remote control: AWS
10. System stability: AWS (On Firebase we had a lot of “infrastructure failures" on certain devices)
11. IDE integration: Firebase
12. iOS support: Both(with the slight advantage to AWS as Firebase XCUITest support is in closed beta at the time of writing)

This list could go on and on, but its goal is not to tell you _"You should never use service X"_. I wanted to point out to the issues and advantages from the real world example.

### Conclusion
A general feeling I get as a user of these services is that there is no large freedom of choice. As our requests and expectations are higher, the walls we hit are higher as well, and it happens much often. The worst thing about it is that you cannot be aware of all these tiny issues when making a decision. Who would think of `JUnit4` issues on AWS… but it happens.

>These services are used on unlimited payed plans, including the traffic generated on Google Cloud Storage. There was no limitations of service due to free or trial usage.

Stay cautious!
