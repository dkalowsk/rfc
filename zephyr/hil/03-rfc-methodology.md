# RFC: Enabling Hardware In the Loop CI

## Problem Description

This problem description assumes that [rfc-tests](rfc-tests.md) and
[rfc-tiers](rfc-tiers.md) have been solved and describes a final step
to enabling Hardware In the Loop (HIL) testing: how should the HIL
testing service run in practice? How should the test harness be administered,
and who should be responsible for individual components within them?

## Proposed Change (Summary)

At a high level, enabling the HIL service is going to require standardizing
upon the location of test boards, a means to administer them, and a means
to enable/disable from the Zephyr CI.

## Proposed Change (Detail)

Now that the project has decided to move forward with HIL testing, the question
becomes where the boards will be housed.  At the time of writing, there are three
options available to the project, each with their advantages and disadvantages. The
next few sections will go into detail of these options.

### Zephyr Hosted

The Zephyr project has dedicated lab space allocated to it in the South Korean datacenter
hosted by Stephanos. Utilizing this space provides the Zephyr project with complete

### 3rd Party Hosted

### Contributor Hosted


Any HIL setup requires a job dispatcher. Typically a job dispatcher has the
following major responsibilities:

- one (to many) worker client(s) connected
- exposes various DUTs (Devices Under Test) connected to a worker
- A centralized scheduling/dispatcher server to associate jobs to workers+DUTs
- the ability to enable/disable DUTs on workers

Importantly, the dispatcher **must not** take over the roles that
`twister` provides today, primarily the test selection, running, and
results collection. The dispatcher must be implemented as a layer on
top of twister, not as a replacement for it.

A single centralized scheduling server allows workers clients to connect,
following a standard hub-and-spoke model. The scheduling server sends incoming
job requests to the workers, and can be used to check status of various jobs
sent out for execution.

A worker can be:

- any device powerful enough to run containers
- any device with at least one USB port
- located remotely from the server
- can have multiple DUTs attached to it
- can have heterogeneous DUTs attached to it
- exposes to the server which DUTs it provides support for

The Zephyr project must control the dispatcher/scheduler, while
allowing access to workers and DUTs to be provided to the project by
community members.

Using the Linaro LAVA system, Zephyr will need a way to host the LAVA
dispatcher code. This can be within Zephyr's current infrastructure or
it can be hosted by a member company.

### Quick LAVA Primer

Because of the dynamic nature of how most dispatchers work, the use of the
Zephyr hardware map becomes extremely difficult if not impossible. Thankfully
much of the same information can be embedded into the dispatcher allowing the
details to move with the device and submit these details to twister via the
command line.

LAVA utilizes a dictionary of device-types that describe the basic capabilities
per device. When a device is added, the device-type information will be
appended to the device information.

An example device-type file for the RaspberryPi Pico:

```jinja2
{# device_type: rpi_pico #}
{% extends 'base.jinja2' %}

{% set board_id = board_id | default('0000000000') %}

{% set serial_port_base = serial_port_base | default('/dev/serial/by-id/usb-Raspberry_Pi_Debug_Probe__CMSIS-DAP__') %}

{% set serial_port_jtag = serial_port_jtag | default('-if01') %}
{% set serial_port_uart = serial_port_uart | default('-if01') %}

{% set serial_port_uart_path = serial_port_uart_path | default(serial_port_base ~ board_id ~ serial_port_uart) %}
{% set serial_port_flash_path = serial_port_flash_path | default(serial_port_base ~ board_id ~ serial_port_jtag) %}

{% set device_info = device_info | default([{'board_id': board_id}]) %}

{% set environment = environment | default({
    "SERIAL_PORT_UART": serial_port_uart_path,
    "SERIAL_PORT_FLASH": serial_port_flash_path,
}) %}

{% block body %}
board_id: '{{ board_id }}'

actions:
  deploy:
    methods:
      image:

  boot:
    connections:
      serial:
      ssh:
    methods:
      jlink:
{% endblock body %}
```

With that information we can see the `serial_port_flash_path` builds the string
needed to uniquely identify the device and allow `twister` to flash the device.

Next is the task of adding the specific device and associating it with a worker.

```jinja2
{% extends "rpi_pico.jinja2" %}

{% set board_id = 'BOARD_SERIAL_NUMBER_GOES_HERE' %}

{% set sync_to_lava = {
    'device_type': 'rpi_pico',
    'worker': 'qcomm-worker-01',
    'tags': [],
    'aliases': [],
    'physical_owner': 'qualcomm'
    }
%}
```

In this example the `board_id` is the UNIQUE identification from the board and
gets set in the device. The `board_id` is then inherited by the device-type,
which allows the configuration of the `serial_port_*` values. The device-type
has default values in case the device forgets to set the values.

In addition to this, LAVA has the concept of a health check that can be run.
The health check is designed to ensure that the device is still available and
functional. In cases where that is not true, the device is placed into
"Maintenance" mode and is no longer available to take on jobs. The health check
can be configured to automatically shut down the USB port, resetting the DUT,
and bringing it back online as a basic first pass to fix DUTs.

### Outline of CI Flow Changes

The basic flow needing to be followed per PR will look like so:

1. Generate the test plan for the Tier 1 boards at the test level of `smoke` and
   the tag `hil_verified` tests the are associated with the incoming change.
1. Have the Zephyr CI build all the unit tests needed to run with a
   `build-only` flag set.
1. Bundle the unit tests and the `tests` directory into a binary and upload
   these to a location that the workers can download from. The `tests` directory
   is necessary in cases where new tests may have been added for the later
   `twister` runs to work.
1. Generate LAVA job.yaml files from a template in the tree with the following
   details:
   1. Locations for bundled artifacts
   1. Name of the LAVA device-type the job needs to run on
   1. Set the steps needed for `twister` to flash this particular platform
1. Submit the generated job files to the LAVA dispatching server for running.

After that, the following steps run on each LAVA worker, with the
details specified in the LAVA job.yaml:

1. Ensure the worker's container has everything needed to run twister.
1. Ensure the worker can download the bundled artifacts.
1. Download the bundled artifacts, and explode them in the correct location
1. Run twister in `device-test` mode.
1. Upload the results for checking.

### The Zephyr CI

The Zephyr CI `.github/workflow/twister.yml` will need to now submit the list
of platforms to run tests on to the `scripts/ci/test_plan_v2.py` script.

The `test_plan_v2.py` script already supports multiple platforms being submitted
via the `-p` flag, leaving the work to be done as a collection step before
calling it. For example, the proposed `west boards --tier 1` could show all the
tier 1 boards associated with the per-PR test suite. That list will need to be
converted into `--platform` arguments for `test_plan_v2.py` to generate the
correct test plan(s).

The additional change `test_plan_v2.py` will need is to allow passing of specific
tags to `twister` when generating the testplan. Specifically the CI will want to
pass the `hil_verified` tag and have the ability to set the level to generate the
correct HIL test plans. The selection of which level to run depends on an
automated run or a per-PR style run. It is proposed that the level support be
added following the same pattern that `twister` currently supports.

### Bundled Artifacts

It is not yet clear what options are available to the project for uploading
temporary bundled artifacts. One suggestion is an Amazon S3 cloud where
pre-authenticated URLs can be generated and auto-expiration of the artifacts
can happen.

### Containers

The CI will need at least one container. It's possible to use the `ci_repo_cache`
container already maintained by the project, although it may be preferred to generate
a smaller container for download.

Any container will need all the pieces to run `twister` installed.

It may make sense to have multiple of these smaller containers with
different host tools required for flashing installed. Each could be named
for the specialization needed. For example, the following containers could exist:

- `worker-segger`
- `worker-openocd`
- `worker-rpi-openocd`
- etc.

It is an open question how much container golf the project wants to play here.
In this case, if smaller containers are the desire, each device-type that
utilizes a specific flashing tooling will need to have a templated job file
that can be used.

### Results Uploading

This is a future TBD section. In short, the project will need a place for
HIL results to be uploaded and scanned for status. The results will be `twister`
log files, which the project is already good at parsing.

## Concerns

The choice of the scheduling harness is polarizing. It is not clear how many
vendors may simply not contribute due to the choice.

LAVA has been around for a long time and is used primarily with Linux system
testing and for the Linux [kernelci.org](https://kernelci.org) testing. The
primary focus fo LAVA is on testing, limiting the uses for which the workers
can be used.

Other options considered:

- [LabGrid](https://github.com/labgrid-project/labgrid) - a very close second
  although not selected as the primary focus for LabGrid is on remote
  development. The biggest takeaway here is that the Baylibre folks report
  that they can use LAVA device files to run `labgrid-client` commands. The
  RFC author has not tested this configuration to verify the functionality,
  but if it works it would give more flexibility to the system.
- [TCF](https://github.com/intel/tcf) - not selected as it would not work with
  twister as documented today.
- [FC](https://github.com/NXP/fc) - not selected as the last updates to the
  code are over two years ago, suggesting the project is no longer actively
  maintained.
- [SLAV](https://github.com/SamsungSLAV/slav/tree/doc) by Samsung - not
  selected as the last updates to the code are over 7 years ago, suggesting
  the project is no longer actively maintained.
- [r4d](https://github.com/ci-rt/r4d) - not selected as I could not get it
  to work, and the project's last code update was over 4 years go suggesting
  the project is no longer actively maintained.
