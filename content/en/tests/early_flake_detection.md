---
title: Early Flake Detection
kind: documentation
description: Detect flakiness before it reaches the default branch
aliases:
- /continuous_integration/tests/early_flake_detection/
further_reading:
  - link: "/tests"
    tag: "Documentation"
    text: "Learn about Test Visibility"
  - link: "/quality_gates"
    tag: "Documentation"
    text: "Learn about Quality Gates"
---

## Overview

Early Flake Detection is Datadog's mechanism to tackle test flakiness. By identifying tests that are newly added and running them multiple times we can detect a high percentage of flaky tests. This can help detect up to [75% of flaky tests][1] before they’re merged to the default branch.

## How it works

* Test Visibility’s backend will automatically store unique tests for a given test service. This is what we'll call **known tests**.
* Whenever a test session is about to run, the Datadog libraries will fetch known tests.
* If a test appears in this list, the execution will continue as normal.
* If a test does not appear in this list, the Datadog library will automatically retry it.

The objective of running a test multiple times is detecting potential issues. If any of the attempts fail, Test Visibility will automatically tag the test as flaky. You may then choose to block the feature branch from being merged with a [Quality Gate][2].

### Why would running a test multiple times help detect issues?

A [flaky test][3] is a test that exhibits both a passing and failing status across multiple test runs for the same commit. This behavior is normally caused by some random component, such as a race condition. By running the test multiple times we increase the likelihood of this random component to show up.


### How does Test Visibility define the list of known tests?

The list of known tests is generated from tests run in default and default-like branches. This what we call the list of [**Excluded Branches**][4].

Early Flake Detection will normally work as follows:

* A new test is added in a feature branch. It will be retried automatically for the current commit and for every commit in the feature branch until it’s merged.
* Once the feature branch is merged, that new test will be considered part of the known tests for the service.

{{< img src="continuous_integration/early_flake_detection_commits.png" alt="How Early Flake Detection works in your commits.">}}

### Excluded branches

The excluded branches will not execute Early Flake Detection, that is, no test will be retried. Additionally, all the tests that run in those branches will update our storage of known tests. This means that what Early Flake Detection considers a new test is based on the tests that run in these branches: if a test has run in any of the excluded branches, it will *not* be considered new anymore.

## Set up a Datadog library
Before setting up Intelligent Test Runner, you must configure [Test Visibility][5] for your particular language. If you are reporting data through the Agent, use v6.40 or 7.40 and later.

## Configuration
Once you have set up your Datadog library for Test Visibility, configure Early Flake Detection from the [Test Service Settings page][6]:

{{< img src="continuous_integration/early_flake_detection_test_settings.png" alt="Early flake Detection in Test Service Settings.">}}

When clicking on **Configure** in the Early Flake Detection column you will see a modal such as this one:

{{< img src="continuous_integration/early_flake_detection_configuration_modal.png" alt="Early flake Detection configuration modal.">}}

You will be able to toggle the status of Early Flake Detection for that test service and add a list of [**Excluded Branches**][4].

## Further Reading

{{< partial name="whats-next/whats-next.html" >}}

[1]: https://2020.splashcon.org/details/splash-2020-oopsla/78/A-Large-Scale-Longitudinal-Study-of-Flaky-Tests
[2]: /quality_gates/
[3]: /glossary/#flaky-test
[4]: /tests/early_flake_detection/#excluded-branches
[5]: /continuous_integration/tests
[6]: https://app.datadoghq.com/ci/settings/test-service
