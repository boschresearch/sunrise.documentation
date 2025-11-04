# Workflow
SUNRISE facilitates the configuration and execution of _systems_ to generate results in a straightforward and reproducible manner. Users follow a logical sequence of steps, independent of simulation technology:

1. Setup
1. Configure
1. Build (optional)
1. Run
1. Analyze
1. Archive

![Execution Workflow](img/sunrise_workflow.svg)


## The Experiment Concept
The workflow is built around the concept of _experiments_. Each experiment is a unique temporary session, that holds a working environment owned by a user to evaluate one use-case. The lifetime of an experiment is typically in the range of minutes to few days and can contain multiple simulation runs with modified parameters.

In an experiment, the [build](#the-build-action) of the simulation model inside a system and the [run](#the-run-action) of the simulation are separate actions. For both, distinct parameters can be configured.
This separation allows two different loop concepts to iteratively improve parameter settings:

- A _full loop_ that re-runs build and run actions
- A _fast loop_ that only repeats the run step and keeps the previous build result


## The Build Action
For systems that contain models which are available in source code of a compiled programming language (e.g. C++ / SystemC), the build comprises the compilation and the generation of the binaries.
After the action, a runnable simulation must be available. As systems can be pre-compiled, _the build step is optional_.

_Build parameters_ have an impact on the source code or the build process itself (e.g. preprocessor options). Adjusting the parameters will require a re-compilation.

## The Run Action
The run action executes the system, so it starts the executable that is created in the build action using the _run parameters_ set by the user. Changing these parameters does not require re-building the binary (e.g.command line arguments of the simulator, embedded software binaries to be loaded on a instruction set simulator or stimulus data files which are read at run-time).

The system is executed until it terminates or a timeout is reached.
In the SUNRISE workflow, the run is a _non-interactive_ action, so user interaction e.g. with a graphical simulation tool, is neither intended nor possible.

## Result Analysis
After completing the run action, the next step involves evaluating the results that have been generated.
The type of results is highly system and use-case specific. Typical are log files, waveform traces, CPU performance metrics (number of instructions and cycles) or embedded software profiling. In general, a result object could be any type of file.
