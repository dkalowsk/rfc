# RFC: Updates to Board Tier

## Problem Description

When the TSC [voted](https://lists.zephyrproject.org/g/tsc/message/1464) to add
tiered board support for testing, the resulting vote left implementation
details open.  Specifically the vote does not define which tiers map to the
defined testing levels, when each tier will be used in the development process,
and did not clearly provide a means to identify which boards are in which
tiers.

## Proposed Change (Summary)

Expanding the tiered board support concept beyond 3 tiers will help clarify
which board tier maps to specific testing levels. Adding an in-tree mechanism
to record and discover which boards are in which tiers ensures that the
information is able to be queried and expanded for the future lifetime of
releases.

This introduces the ground work infrastructure needed to move towards allowing
hardware testing to block the merge of a PR.

## Proposed Change (Detail)

The TSC voted and approved the creation of a [tiered
hardware](https://docs.zephyrproject.org/latest/project/release_process.html#hardware-support-tiers)
support system policy. A brief summary (see link for full details):

- Tier 0: Running twister tests on emulation platforms
- Tier 1: Running a subset of publicly available platforms with twister device
testing on a regular basis. These platforms are typically paid support.
- Tier 2: Platforms supported by the community for which no promise of testing
is made yet likely work due to the efforts of the community support.
- Tier 3: Platforms that are generally deprecated or not supported.

The policy can be broken into a chart:

| Tier    | Per PR | Per Cadence | Supported not tested | Not Supported |
|---------|--------|-------------|----------------------|---------------|
| Tier 0  |    x   |             |                      |               |
| Tier 1  |        |      x      |                      |               |
| Tier 2  |        |             |           x          |               |
| Tier 3  |        |             |                      |       x       |

Enabling Hardware in the Loop PR support proposes expanding the definitions of
the tiered support. As before each tier continues to encompass the support of
the previous tier.

- Tier 0: The testing of all PRs with the support of emulation and simulators
to verify the logic. This is no change from what we have today. Failure on this
tier blocks merge for the PR.
- Tier 1: The use of a subset of hardware platforms for which passing testing
of PRs through twister on device testing becomes a requirement.
- Tier 2: An expanded set of hardware platforms and Zephyr unit tests to be run
on a regular cadence basis (i.e. weekly). Failures in the cadence test are by
definition not blocking, as the code has already been merged to the tree.
- Tier 3: The running the complete list of boards expected to be supported by
the a release, provided there is hardware available. The tests expected to run
include both the fast and long run tests. Results will be uploaded for review
by the Release Working Group in preparation for the pending Zephyr release
event.
- Tier 4: Replaces what was formerly called Tier 3. These are boards no longer
actively supported. They will not be run through any testing.

A chart version of the above descriptions to make it easier to understand.

| Tier    | Per PR | Per Cadence | Per RC or Release | Not Supported |
|---------|--------|-------------|-------------------|---------------|
| Tier 0  |   x    |      x      |         x         |               |
| Tier 1  |   x    |      x      |         x         |               |
| Tier 2  |        |      x      |         x         |               |
| Tier 3  |        |             |         x         |               |
| Tier 4  |        |             |                   |       x       |

### PR vs Merge Train

During the Zephyr Maintainers Forum (ZMF), it was suggested that Tier 1 not be
used on a per PR basis, rather on a per Merge Train basis. The reasoning behind
the suggestion is that as a project we do not need or want to test the
iterations of the PR. That is a requirement on the developer to ensure.
Functional testing on hardware by the Zephyr project is a verification gate
only to be used on the last version about to be merged in to `main`. The
proposed change to use the merge train sounds reasonable and the RFC author
agrees with the approach.

At least one edge case exists when limiting hardware testing to a merge train
that should be explored.  In talking with several developers, it was pointed
out that there do exist changes where it is useful to have the tests run per PR
iteration. For example, the picolibc maintainer has mentioned how useful it
would be to ensure changes made work on as many platforms as possible before
moving a PR out of Draft form. The challenge for this area of development is
having access to all the boards.  As such it is recommended that, if the
project decides to run HIL on a per-merge-train, there also be a mechanism for
a PR to be run on hardware before the merge train event. It is likely this
feature can be limited in who can enable it.

### Mapping Tiers and Levels Expectations

Currently the Zephyr project has
[levels](https://github.com/zephyrproject-rtos/zephyr/blob/d1b8eec76bfd14058854245d09aa1e413e355c4a/scripts/schemas/twister/testsuite-schema.yaml#L78)
defined. For example `smoke` and `acceptance`. Mapping these test levels to
tiered board support ensures all areas of testing are covered and clearly
documented.

#### Tier 1

Requirement(s): Boards in the Tier 1 classification are expected to always
work. An incoming PR to Zephyr that triggers a run on a Tier 1 board is
identified to have changes in code with known hardware sensitive areas.

Test Level(s): `smoke`

Test Length: 5-10 minutes total in length (includes flashing time)

Passing Signature: PR passes CI check.

Failure Signature: Blocking of the merge of the PR. By blocking the merge, with
platform champion(s) agreeing to assist in debugging the failure to unblock the
PR.

Result(s): At this level, passing and failing of the checks accepts or blocks a
PR from being merged.

#### Tier 2

Requirement(s): Boards in the Tier 2 classification run at a some regularly
defined cadence and include the boards for Tier 1 in the run. For example the
cadence can be defined as daily or weekly. The key distinction between Tier 1
and Tier 2 is that for Tier 2 the code has already been merged to the tree.

Test Level(s): `acceptance` and `unit`

Test Length: Cadence based test must be less than the start of the next cadence
cycle.

Passing Signature: A clean run of all `twister` tests.

Failure Signature: If a `twister` test does not pass, the creation of an issue
that will be dispensed through the current issue review process.

Result(s): The collection of test results per board will be used to generate a
dashboard reflecting the current health of the project at this cadence check.

#### Tier 3

Requirement(s): This Tier is really a specialization of the Tier 2 for use in
the release process. The specialization allows it some additional length of
time and boards to include per run.

Test Level(s): `system`

Test Length: Several hours.

Passing Signature: A clean run of all `twister` tests.

Failure Signature: A not clean run of all `twsiter` tests.

Result(s): The collection of test results per board will be used to generate a
dashboard reflecting the current readiness of the project to tag a release or
release candidate.

#### Tier 4

Requirement(s): Unsupported tier.

Test Level(s): N/A

Test Length: N/A

Passing: N/A

Failure: N/A

Result(s): These boards are provided as-is with no promise of working or
tracking by the project.

### Board Identification

This proposal includes extending the boards.yml file in the boards directory to
include a new mapping dictating the tier level for each board. Adding the
information here has the advantage of allowing out-of-tree boards to also be
easily picked up for testing using the same CI patterns.

Examples of the suggested addition:

```diff
board:
  name: rpi_pico
  full_name: Raspberry Pi Pico
  vendor: raspberrypi
+  tier: 2
  socs:
    - name: rp2040
      variants:
        - name: w
          variants:
            - name: mcuboot
        - name: mcuboot
```

and

```diff
board:
  name: qemu_cortex_m3
  full_name: QEMU Emulation for ARM Cortex-M3
  vendor: qemu
+  tier: 0
  socs:
  - name: ti_lm3s6965
```

Further, the `west boards` command will need to be extended to add support for
identifying and displaying only those boards with a specific tier setting. An
initial proposal of the command in question would be `west boards --tier X`,
where `X` can be the numerical value of the tier.

### Dynamic Tiers

During the Zephyr Maintainers Forum (ZMF) the suggestion was made to have the
tier marker be dynamically generated instead of statically set. The idea is
that any board can be made a Tier 1 provided it passes some set of heuristics
that would need to be determine. Examples used at ZMF were availability,
reliability, and speed.

While this idea appears to be a very good change, it leaves two areas open to
question.

1. As the tier status is developed at runtime, any release will lose access to
   that data making it hard, if not impossible, to reproduce the release CI. This
   becomes especially troublesome when considering an LTS and a need to ensure
   testing environments persist. One suggested action to address this short
   coming is to freeze the boards tier status as RC1 is tagged. This keeps the
   proposed addition in boards.yml, and requires the Release Team to add this
   item to the tracking list.  
2. The dynamic tier calculation will limit the ability of out-of-tree boards to
   utilize the same framework for their own testing. Given one of the talks
   at ZMF pushed the idea that not everything needs to be in the main tree, and
   for out-of-tree be seen as just as important, cutting out the ability for
   these users seems counter to the overall project goals.

## Concerns and Unresolved Questions

Merge Trains appear to be the future direction for merging PRs of the project
and add several complications that cannot yet be answered. Their implementation
is still a work in progress. It is not yet clear if it will be possible and how
to add stages to it's workflows.

The design of a mechanism to allow specific PRs to run on hardware tests before
the merge train will need to be designed.  This steps into areas of the Zephyr
CI and GitHub actions that the author of the RFC is not completely confident on
the ability to make the changes and may require community assistance.

### Unmapped Test Levels

The design leaves the following test levels unmapped: `integration` and
`regressions`. It is not currently known if these levels are in use and if so
where they would fit into the recommended tier structuring.

## Terminology

Tier - a hierarchical ranking of boards and their value.

Platform Champion - This is a company or person who has agreed to provide
support for a specific set of hardware platforms.  This support includes
assisting in the debug of PR failures on the platform.

Merge Train - a deployment queue mechanism that combines multiple PRs into a
single into a single group. This allows for testing in the exact state that the
commits will land, preventing integration failures.
