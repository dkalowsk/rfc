# RFC: Identifying Unit Tests for HIL

## Problem Description

The Zephyr Maintainers Forum (ZMF) 2026 voted to pursue hardware in the
loop (HIL) testing at the project level. The ZMF discussion included
proposals to do HIL testing in Zephyr CI, either at pull request (PR)
time or at "merge train" time. (Details about "merge trains" are unclear
at this point.)

Enabling new features, especially those associated with the CI system
like on-PR HIL testing, must be done in a way that allows for gradual
adoption. Otherwise, development could come to a halt as problems with
the new features are identified and worked out.

Running all available twister test suites will not allow the project a
moment to ensure that the tests accurately work with the
infrastructure changes. Therefore, the first problem for the project
is to identify and agree on which test suites should be run depending
on what areas the diff in the PR is changing.

## Proposed Change (Summary)

Allowing the Zephyr project to continue to work at its current pace or faster,
the goal here is to provide a mechanism with which the HIL functionality can
more slowly roll out and catch up.

The idea is that the Testing Working Group will continue to independently
develop the test plans for the current levels available. As the HIL
infrastructure comes online, a tag will be added to those identified tests to
ensure they pass all the available platforms. The method recommended is to add
new tags to the Zephyr unit tests to provide confidence the test has been verified
to run on the HIL setup.

The project already makes use of different levels for testing, specifically the
`smoke`, `acceptance`, `nightly`, and `stress` levels. These levels will
continue to be leveraged and used for the base configurations.

## Proposed Change (Detail)

The addition of a new tag, `hil_verified` or `hil_test`, will assist in slowly
rolling out support for each test by showing what tests have been verified.

As more tests are able to come online for the `smoke` and `acceptance` test levels,
the `hil_verified` tag will be added to them.  At some point the need for the
additional tag will likely no longer exist and the tag can be removed.

## Concerns and Unresolved Questions

The current HIL design focuses on SOC functionality. Assuming the design works,
it is logical to expand testing to more areas, including sensors, execution
performance (stress), and power performance. It may make more sense to come up
with additional test tag heuristics or levels to allow for better coverage as
infrastructure support is improved.

The addition of a cadence level (such as `weekly`) may have some value for HIL
testing allowing the HIL system to do further testing at some cadence.
