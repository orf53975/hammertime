# README #

**Hammertime**: a software suite for testing, profiling and simulating the rowhammer DRAM defect.

### What does this project contain? ###

While still a work in progress, the following components make up Hammertime:

* `libramses`: a library that handles address translation for the entire memory stack.
* `libperfev-util`: a library providing a more human-friendly interface to Linux's performance event API.
* **Probes** for monitoring memory access behaviour of running programs.
* **Predictors** that decide whether a certain memory access behaviour triggers rowhammer.
* Glue code to tie all this together and effect bit flips in memory.
* **Fliptables**: example profiles of rowhammer-vulnerable DRAM chips, usable by a dedicated predictor.
* Various cool tools:
	* `tools/profile`: a tool to test a running system's vulnerability to rowhammer.
For more information, check out its own README file under `tools/profile/README`.
**NOTE:** this tool is still an experimental bundle of spaghetti code which will get rewritten at some point.
Therefore do not consider any part of it a stable API until this notice is removed.
	* `tools/py/prettyprofile.py` converts a `profile` output into something more human-friendly.
	* `tools/py/hammerprof.py` converts a `profile` output into a fliptable.
	* `ramses/tools/msys_detect.py` interactive tool for detecting current system memory configuration.

For an in-depth view of how these components fit together and the overall architecture of Hammertime check out `DESIGN.md` and browse the source code.
Header files are particularly explanation-rich.

### What can it *do*? ###

**NOTE:** a singular application providing user-friendly access to Hammertime's features is planned.
Until then, Hammertime is intended to be more like a toolbox you use from your own programs.

The main intended use case is as a bit flip simulator for arbitrary programs.
That would, in principle, involve the following sequence of activities:

1. Setup and attach a **probe** to a task / PID / system
2. Collect **probe** output and feed it into a **predictor**
3. Respond to events generated by the **predictor**
4. Produce bit flips when instructed by the **predictor**
5. Go back to step 2 for as long as you feel like

The `demo` program presents a proof-of-concept implementation of a rowhammer bit flip simulator for various run scenarios.

In addition `tools/profile` is a powerful rowhammer testing and profiling tool.

### How do I get set up? ###

#### Dependencies ####

* Linux kernel >= 3.18
* `glibc` with pthread support
* `libpfm4`
* Python 3 --- used by tools

#### Building ####

Run `make` in the root directory to build all Hammertime components and tools, except for the demo binary.

`make demo` will build the demo binary.

`make all` will build everything there is to build.

`make clean` removes all previously built files.

### Getting started ###

#### Detecting your system's memory configuration ####

A memory configuration (i.e. `.msys`) file includes information about the memory controller, physical address router, DRAM geometry and optional on-chip remapping.
Figuring these out by hand is tedious; here's where a tool comes in.

Run `ramses/tools/msys_detect.py`.
It will try to auto-detect most parameters and ask you for the others.

The output file it produces can now be used by other Hammertime components.

#### Testing for rowhammer ####

The profile tool requires elevated permissions.
Either run it as root or run `make cap` as root in its directory to set the necessary capabilities on the binary.

Example run with only the essential arguments:

`tools/profile/profile -s 256m spam.msys`

will run a basic double-sided rowhammer attack over 256MiB of RAM using `spam.msys` as memory configuration file.

The output may seem a bit cryptic. To remedy this, use the prettifying script:

`tools/profile/profile -s 256m spam.msys | tools/py/prettyprofile.py -`

as a shell pipeline or

```
tools/profile/profile -s 256m spam.msys myprof.res
tools/py/prettyprofile.py myprof.res
```

by using a temporary file (the `.res` extension stands for "result"; it's the default one expected by the fliptable build system. More on that later)

Check out `profile`'s own README file for more options and how to read its raw output.

#### Simulating bit flips ####

The `demo` program is a good place to start.
The program is not intended as a proper component of Hammertime, rather more like a runnable "What's new" showcase.

1. Read the source code in `demo.c` and `glue.c` to get an idea of how to get Hammertime's components working together.
2. (Optionally) Edit `demo.c` to suit your needs (tweaking thresholds, input file paths, etc.)
3. Run `make demo`
4. Run `demo` as root:
	* `./demo <PID>` attaches to the main thread of a running process specified by PID
	* `./demo -e <PROGRAM> [ARGS]` launches PROGRAM with ARGS as arguments and attaches to it; this also monitors all spawned threads
	* `./demo -s` attaches to the entire running system. **NOTE:** Requires an unrestricted `/dev/mem`, which means you most probably have to recompile your kernel with `CONFIG_STRICT_DEVMEM=n`

The `fliptables/` directory contains some example fliptables based on profile runs of real DRAM chips.
Rowhammer attack patterns of single-sided, double-sided, adjacent (a.k.a. "amplified") and double-sided under memory pressure are provided.
`.msys` files describing the memory layout of tested modules are also included.

#### Brewing your own fliptables ####

Say you have found bit flips running `profile` and want to use them with the simulator.
Simply run

`tools/py/hammerprof.py myprof.res myprof.fliptbl`

to convert the `profile` output `myprof.res` into a binary fliptable.

Alternatively just place your `.res` file(s) under `fliptables/` and run `make fliptables`.
The build system will take care of the rest.

### How can I contribute? ###

#### I have a new memory address translation / routing / mapping / remapping scheme ####

Great! Include your changes under `ramses/` and send a pull request.
Take care not to break the ramses ABI though.

Currently, AMD memory controller mapping support is missing and would be greatly appreciated.
The specs are out there in AMD's BIOS and Kernel Developer's Guide (BKDG): [AMD Developer Manuals](http://developer.amd.com/resources/developer-guides-manuals/).
Search for "DRAM Address Mapping".

#### I wrote a new probe / predictor ####

Include your changes under the appropriate directory and send a pull request.
Try to stick to the generic interface for your component so it won't require custom glue code.

#### I found a bug! ####

Report it on the bug tracker [here](https://github.com/andreittr/hammertime/issues).

#### I have an interesting profile result I'd like to share ####

Get in touch with me. See e-mail below.

### Who do I talk to? ###

Have a question, suggestion, comment? Drop me an e-mail at andrei.ttr@gmail.com .
