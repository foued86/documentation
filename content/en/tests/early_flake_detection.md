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
---

## Overview

By identifying tests that are newly added and running them multiple times we can detect a high percentage of flaky tests. This can help detect up to [75% of flaky tests][1] before they’re merged in the default branch.



## How it works

* Test Visibility’s backend will automatically store unique tests for a given test service. This is what we'll call **known tests**.
* Whenever a test session is about to run, the DataDog libraries will fetch known tests.
* If a test appears in this list, the execution will continue as normal.
* If a test does not appear in this list, the DataDog library will automatically retry it.

The objective of running it multiple times is to detect potential issues. If any of the retries fail, Test Visibility will automatically tag a test as flaky.

### How does Test Visibility define the list of known tests?

The list of known tests is generated from tests run in default and default-like branches. This what we call the list of excluded branches.

Early Flake Detection will normally work as follows:

* A new test is added in a feature branch. It will be retried automatically for the current commit and for every commit in the feature branch until it’s merged.
* Once the feature branch is merged, that new test will be considered part of the known tests for the service.

{{< img src="continuous_integration/early_flake_detection_commits.png" alt="How Early Flake Detection works in your commits.">}}


## Configuration
[1]: https://2020.splashcon.org/details/splash-2020-oopsla/78/A-Large-Scale-Longitudinal-Study-of-Flaky-Tests
