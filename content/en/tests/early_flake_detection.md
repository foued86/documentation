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

{{< site-region region="gov" >}}
<div class="alert alert-warning">Early Flake Detection is not available in the selected site ({{< region-param key="dd_site_name" >}}) at this time.</div>
{{< /site-region >}}

{{< callout url="#" btn_hidden="true" >}}
Early Flake Detection is in public beta.
{{< /callout >}}

## Overview

Early Flake Detection is Datadog's mechanism to tackle test flakiness. By identifying tests that are newly added and running them multiple times we can detect a high percentage of flaky tests. This can help detect up to [75% of flaky tests][1] before they’re merged to the default branch.

## How it works

* Test Visibility’s backend will automatically store unique tests for a given test service. This is what we'll call **known tests**.
* Whenever a test session is about to run, the Datadog library will fetch the list of known tests.
* If a test appears in this list, it will be executed as normal.
* If a test does not appear in this list, the Datadog library will automatically retry it a number of times.

The objective of running a test multiple times is detecting potential issues. If any of the attempts fail, Test Visibility will automatically tag the test as flaky. You may then choose to block the feature branch from being merged with a [Quality Gate][2].

### Why would running a test multiple times help detect issues?

A [flaky test][3] is a test that exhibits both a passing and failing status across multiple test runs for the same commit. This behavior is normally caused by some random component, such as a race condition. By running the test multiple times we increase the likelihood of this random component to show up.


### How does Test Visibility define the list of known tests?

The list of known tests is generated from tests run in default and default-like branches. This what we call the list of [**Excluded Branches**][4].

Early Flake Detection will normally work as follows:

* A developer working in a feature branch writes a new test, commits and pushes the changes.
* This test will be retried automatically for this commit and every new commit in the feature branch, until the branch is merged.
* Once the feature branch is merged, that new test will be considered part of the known tests for the test service.

{{< img src="continuous_integration/early_flake_detection_commits.png" alt="How Early Flake Detection works in your commits.">}}

### Excluded branches

The excluded branches will not execute Early Flake Detection, that is, no test will be retried. Additionally, all the tests that run in those branches will update our storage of known tests. This means that what Early Flake Detection considers a new test is based on the tests that run in these branches: if a test has run in any of the excluded branches, it will *not* be considered new anymore.

## Set up a Datadog library
Before setting up Early Flake Detection, you must configure [Test Visibility][5] for your particular language. If you are reporting data through the Agent, use v6.40 or 7.40 and later.

## Configuration
Once you have set up your Datadog library for Test Visibility, configure Early Flake Detection from the [Test Service Settings page][6]:

{{< img src="continuous_integration/early_flake_detection_test_settings.png" alt="Early flake Detection in Test Service Settings.">}}

When clicking on **Configure** in the Early Flake Detection column you will see a modal such as this one:

{{< img src="continuous_integration/early_flake_detection_configuration_modal.png" alt="Early flake Detection configuration modal.">}}

You will be able to toggle the status of Early Flake Detection for that test service and add a list of [**Excluded Branches**][4].

### How do I know that new tests are being detected?

There a set of facets you can use to query sessions that are running Early Flake Detection and tests that are being detected as new.

#### Test session

Test sessions that are running Early Flake Detection will have the tag `@test.early_flake.is_enabled` set to `true`.

#### Test

Tests that are considered new will have the `@test.is_new` tag set to `true`.

Additionally, retries for this test will have the `@test.is_retry` set to `true`.


## Troubleshooting

If you suspect there are issues with Early Flake Detection, disabling it is as simple as enabling it. Simply go to the [Test Service Settings][6], look for your test service, and click **Configure**.

### This new test is not being retried

This could be caused by a couple of reasons:

* This test has already run in a excluded branch, such as `staging`, `main` or `preprod`.
* This test is slower than 5 minutes. There is a mechanism not to run Early Flake Detection on tests that are too slow, since these tests being retried could cause significant delays in CI pipelines.


### A test was retried that is not new

If there's a problem in Test Visibility's backend and the library can't successfully fetch the full list of known tests, it could cause the library to retry tests that are not new.

We have a mechanism in place to prevent this error from slowing down your pipeline, but if it happens, please contact our support team at [Datadog Help][7].

## Further Reading

{{< partial name="whats-next/whats-next.html" >}}

[1]: https://2020.splashcon.org/details/splash-2020-oopsla/78/A-Large-Scale-Longitudinal-Study-of-Flaky-Tests
[2]: /quality_gates/
[3]: /glossary/#flaky-test
[4]: /tests/early_flake_detection/#excluded-branches
[5]: /continuous_integration/tests
[6]: https://app.datadoghq.com/ci/settings/test-service
[7]: https://docs.datadoghq.com/help/
