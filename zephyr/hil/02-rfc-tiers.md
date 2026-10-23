# RFC: Updates to Board Tier

## Problem Description

The current policy developed for the Zephyr projects tiered board support lays
the ground work for starting Hardware In the Loop (HIL) testing. With a few
tweaks it can be expanded to support HIL testing in a per-PR and regular
cadence testing work flow, while aligning to the defined testing levels.

## Proposed Change (Summary)

Adding more granularity to tiered board support definition will allow the Zephyr
project's CI to expand to include HIL support. The proposal does not remove
the currently defined tiered support. Instead it adds another layer to make it clear which boards
are needed and when.

Additionally the proposal adds an in-tree mechanism to record the project level
decisions on which boards are in the tiered support.

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
  to verify the logic. This is no change from what we have today. Failure on
  this tier blocks merge for the PR.
- Tier 1: The use of a subset of hardware platforms for which testing of PRs
  through twister on device testing becomes a requirement. These PRs contain
  changes to code with known hardware sensitive areas as identified by the
  project. These are to be fast unit tests that taken as a whole require no
  more than 5-10 minutes to run total. Failure on this tier blocks the merge of
  the PR. By blocking the merge, platform owners agree to assist in debugging
  the failure to unblock the PR.
- Tier 2: An expanded set of hardware platforms and Zephyr unit tests to be run
  on a regular cadence basis (i.e. weekly). These longer run tests are to check
  known hardware sensitive areas as identified by the project and may take
  several hours to run as a whole set. Test results will be uploaded for presentation in a dashboard. Review of the HIL dashboard will be part of the weekly Release Working Group.
  Failures in the weekly HIL test will then be assigned a priority and assigned an owner.
- Tier 3: The running the complete list of boards expected to be supported by
  the the release provided there is hardware available. The tests expect to run
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
the suggestion is that as a project we do not need to test the iterations of
the PR, the functional testing should only be on the last version about to be merged in to `main`. The proposed change to use the merge train sounds reasonable.

In talking with several developers, it was pointed out that there do exist
changes where it would be useful to have the tests per PR. For example the picolibc maintainer has mentioned how useful it would be to ensure changes they have made work on as many platforms as possible before moving a PR
out of Draft form. As such it is recommended that, if the project decides to run
HIL on a per-merge-train, there also be a mechanism for a PR to be run on
hardware before the merge train event. It is likely this feature can be limited
in who can enable it.

### Tiers and Levels

Currently the Zephyr project has levels defined. For example `smoke` and `acceptance`. Many of these
can be mapped one-to-one if preferred.

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
