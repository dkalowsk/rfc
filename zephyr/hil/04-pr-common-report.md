# Common HIL PR Reporting Format

## Problem Description

Vendors and random contributors have begun enabling their own Hardware In the
Loop (HIL) testing reports and sharing the results on the PR itself. In each case
the data being shared is of various quality. In some, the results are extremely 
verbose. In others, the results are more streamlined and presentation friendly.
While in yet others the results are barely noticable.

Development of a common reporting structure for the project on a per PR basis
becomes essential to ensure all community members are able to easily identify
key information and the results.

Example PR reports:
 - [Espressif](https://github.com/zephyrproject-rtos/zephyr/pull/117493#issuecomment-6748815326)
 - [Infineon](https://github.com/zephyrproject-rtos/zephyr/pull/101608#issuecomment-3760063560)
 - [NXP](https://github.com/zephyrproject-rtos/zephyr/pull/122426#issuecomment-6463427636)
 - [Contributor](https://github.com/zephyrproject-rtos/zephyr/pull/118411) see description
 - [Contributor](https://github.com/zephyrproject-rtos/zephyr/pull/120620#issuecomment-6088742399)


## Proposed Change (Summary)

Proposal is to develop a common template for reporting HIL results to the
project on a per PR basis.  Specifically focusing on format and which pieces of
data to include or not include.

## Proposed Change (Detail)

Development of an extendable template to present the test results in a
consistent and readable manner.

There are two routes this can work, listing test passes, failures, and skipped
by platform.  Or listing each test with which platforms passed or failed to it.

Either works as an extensible format. The pick of organizing the data by
platform first is selected by the belief that more users will care about
specific platforms than specific tests passing first. If this assertation
proves to be incorrect, the template can be inverted.

### Full Test Summary

The top of the template will contain a summary of the HIL test results for
quick glance checking.

```
# Test Results HIL

**Platforms Tested:** 
1. Platform A
1. Platform B
1. ... etc ...

**Test SHA:**  <GIT_SHA_HERE>

## Results Summary by Platform

| Platform | Passed | Failed | Skipped | Total |
|----------|--------|--------|---------|-------|
|          |        |        |         |       |
|----------|--------|--------|---------|-------|
| Total(s) |   X    |   Y    |   Z     |   N   |

```

### Platform Test Details - Repeated

The lower portion of the template is organized by platform to contain specific
tests further organized by their results.  This section is expected to be repeated
per platform.

```
## Run Details

### <PLATFORM_NAME>
#### Test Suite Failures
<details><summary>TestSuite Failures</summary>

<details>
<summary>TEST_NUMBER: TEST_NAME :: reason: TEST_ERROR</summary>

    ```sh
    *** Booting Zephyr OS build v4.5.0 ***
    Running TESTSUITE ...
    etc... details here ...
    ```

</details>
</details>

<details><summary>TestCases Skipped</summary>


</details>

#### Test Suite Success
<details><summary>Test Suite Success</summary>

<details>
<summary>TEST_NUMBER: TEST_NAME</summary>

    ```sh
    *** Booting Zephyr OS build v4.5.0 ***
    Running TESTSUITE ...
    etc...
    ```

</details>
</details>

#### Test Cases Skipped
<details><summary>Test Cases Skipped</summary>

<details>
<summary>TEST_NUMBER: TEST_NAME :: reason: TEST_SKIPPED_REASON</summary>
<summary>TEST_NUMBER+1: TEST_NAME :: reason: TEST_SKIPPED_REASON</summary>
</details>
</details>
```

