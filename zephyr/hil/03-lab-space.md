# RFC: Enabling Hardware In the Loop Labs

## Problem Description

Any Hardware In the Loop (HIL) service requires access to the hardware on which
to run the tests. Access to the hardware comes with challenges on where those
devices shall exist to include in testing. How the boards are shared and
exposed to the Zephyr project for use in the CI/CD system is an essential step
for adding HIL support.

## Proposed Change (Summary)

At a high level, enabling the HIL service is going to require standardizing
upon the location of test boards, a means to administer them, and a means
to enable/disable from the Zephyr CI.

## Proposed Change (Detail)

At the time of writing, there are three options available to the project, each
with their advantages and disadvantages. The next few sections will go into
detail of these options.

Each case shares similar concerns about the boards.

- The boards must be made available by the project contributor(s) for exlusive
  use to the Zephyr project.  The Zephyr project will not be able to purchase
  the boards needed and will not be able to time share the boards.
- Any additional equipment needed for flashing or testing the boards must be
  provided by the board submitter(s).
- All boards must be funcitonal and able to flash unattended, without a need
  for additional rework.
- Boards will fail. Reasons can range from electrical to hitting MTBF values.
  Replacement of those failing boards will be required by the project
  contributor(s), otherwise the failed boards will be removed from the
  HIL support.

Any lab space utilized will also have the same requirements:

- Reliable and consistent high speed network connectivity
- Reliable and consistent power
- Temperature and humidity control
- On premise support 24-hours a day for network and board support
- Each lab will host a board coordinator that exposes the boards
  to the Zephyr CI/CD for use.

### Self-Hosted Lab

In this scenario, the Zephyr project would look to acquire secure lab space
to host the infrastructure needed for HIL support. Contributors with boards to
include in the HIL test will be required to physically send their boards to the
project for inclusion. The Zephyr project will host the board coordinator and
be responsible for ensuring the infrastructure to connect all the boards to
the coordinator is correct.

When boards are slow or malfunctioning, a member of the Zephyr project will be
required to remove the board from the device queue.  If the board requires
replacement, it will not be re-added until a physical replacement can be
aquired. If a project contributor sends multiple boards ready for a backup, the
Zephyr project will need to include space to store these backup boards.

The Self-Hosted lab puts the cost of space, electrical, thermal, and support on
to the Zephyr project. Movement forward with this configuration will require
financial approval from the Governing Board.

### 3rd Party Hosted Lab

In a 3rd Party hosted lab, the Zephyr project would contract with an independent
3rd party to provide the functionality described in the self-hosted.

The 3rd Party lab puts the cost of space, electrical, thermal and support into
a contact that the Zephyr project will agree to. It is possible a 3rd part lab
will also donate their services to the project as well. Movement forward with
this configuration will require financial approval from the Governing Board.

### Distributed Hosted Lab

A distributed lab scenario follows a more traditional spoke-and-wheel design.
The Zephyr project hosts only the board coordinator that exposes various boards
available and schedules the boards for HIL tests.

The boards themselves are located at various project contributors sites. The
contributor(s) agree to provide the support necessary. The boards will be
connected to a local "worker" that shares the supported boards with Zephyr
Project's hosted board coordinator in a secure manner. Each worker may have one
to many boards connected to it that will be exposed for project to utilize. It
is at project contributor's discretion for the quantity to share.

The Zephyr project maintains the ability to disable boards that are too slow or
not functioning correctly. Re-enabling these board can also be handled by the
Zephyr project when coorections from the project contributor(s) is made.

## Recommendation

The recommendation for a HIL setup on the Zephyr project is to utilize the
Distributed Lab scenario.  There are several reasons for this:

- The cost of running a lab is not cheap. In many cases, those submitting
  boards for inclusion in a HIL test system already have lab space allocated
  for their own device testing. The distributed method allows the Zephyr
  project to piggy-back off this existing infrastructure.
- Every board has quirks to it. These quirks can include boot processes,
  flashing steps, or general electrical interference challenges. Those
  submitting the boards for inclusion already are aware of these challenges and
  are better equiped to maintain them.
- The distributed lab does not preclude the use of a 3rd party or self-hosted
  lab infrastructure also existing if desired in the future.
