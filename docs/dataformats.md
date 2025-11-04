# Data Formats
This chapter introduces data formats that are used in SUNRISE to define systems and exchange content over the [SysAPI](api_specification.md#system-api-sysapi) and [EvalAPI](api_specification.md#evaluation-api-evalapi) interfaces.

JSON is used as the standard format for data handling in the SUNRISE environment. With Python being the preferred programming language for the SUNRISE implementation, the format definitions are implemented as [_Pydantic_](https://docs.pydantic.dev/latest/) models.

![SUNRISE Data Formats](img/diag_dataformats_overview.svg)

!!! NOTE
    In the _SUNRISE demonstrator_, the Pydantic models are found in the [dataformats](https://github.com/boschresearch/sunrise.demonstrator/tree/master/dataformats) directory.

## System Definition File (SysDef)
The _SysDef_ JSON file is the central specification that holds all information that the SUNRISE Runtime Manager uses to _identify_, _build_, _run_ and _parametrize_ a system. It is stored in the root directory of the git repository of every system as _sysdef.json_ file.

As the SysDef is the single source to describe a system, this file is particularly relevant for _system integrators_. Front-end users can retrieve the file over the EvalAPI and extract features and capabilities of a system.

The _SysDef_ contains the following hierarchical elements (an example code can be found in the [API specification section](api_specification.md#preparation)):

- `documentation` holds a sub-object with textual information about the system: The name of a responsible person, a short summary and a more detailed description. The `description` entry can also be a link to a _Markdown file_. This link is relative to the sysdef.json inside the git repository. If the linked Markdown document embeds image files, these must also be stored relative to the Markdown document in the git repository.
- `docker_image` is the full URL of the location of the Docker image in a container registry.
- `build_command`: The command to _build_ the system. This can be any type of command with optional arguments which can be executed inside the system container.
For pre-built systems in which an executable (simulation model) is already available in the systems docker image the build step is not needed.
The SysAPI appends the absolute path to the [SysCfg](#system-configuration-file-syscfg) JSON file at the end of the `build_command` when the build command gets invoked by the Runtime Manager.
- `run_command`: The command to _run_ the system. It has the same structure and behavior as the `build_command`.
- `delete_command`: Optional step to release external resources that were used during the experiment. It has the same structure and behavior as the `build_command`.
- `build_parameters`: Configuration options used when building the system with the `build_command`.
- `run_parameters`: Configuration options for executing the system with the `run_command`.
- `common_parameters`: Configuration options for both, the run and build actions in addition to the action specific parameters. A typical example are license settings for systems based on commercial tools.
- `results` contains sub-objects (`SysDefResult`) that define artifacts produced by the run action.
    - `path`: Is a relative location from sysdef file where the result file is found. It must be inside the working directory (e.g. `example.txt` and `any-sub-dir/example.txt` are allowed, whereas `../outside-dir/example.txt` is not allowed). The current working directory is provided by the invoking instance of the SysAPI and part of the mounted Docker volume to persistently save the result data.
    - `type` of the result object must be one the pre-defined values that are described in section [Results](#results).
    - `enabled_by`: Optional _pointer to a boolean parameter_ that acts as a switch to decide if this result object is generated. If the linked parameter is `true`, it is expected that the result object will be generated during run. As the `enabled_by` field is a list, multiple build or run parameters can be referenced. The concatenation of the list entries is interpreted as logical `AND`.

In addition to the aspects described in the list above, there are **general requirements** for the SysDef:

- A **default value** must be provided for every parameter.
- A system must be **complete and executable** with the content given in the SysDef. The content of the SysDef must satisfy the [System Integration Requirements](api_specification.md#system-integration-requirements).

## System Configuration File (SysCfg)
The _SysCfg_ JSON file is passed from the front-end over the EvalAPI to the Runtime Manager for the creation of an experiment. After checks and adaptions inside the Runtime Manager, the modified SysCfg is forwarded to the system through the SysAPI.
At a minimum, the system identification with name and version are required, so the Runtime Manager can find the system from the library during experiment creation, where the SysCfg is passed to the SysAPI call. The system itself can also use this information as an additional check if the requested system and version over the SysAPI are as expected.

The SysCfg contains user-defined values for the parameters defined in the [SysDef](#system-definition-file-sysdef). If no value is set in the SysCfg for a parameter, the default value from the SysDef is used.
_File Parameters_ are adjusted by the Runtime Manager, so that they hold a valid path to the file object provided for the parameter in the systems runtime environment. This value can be directly used in the system context when the SysCfg is passed to the `run_command` or `build_command`.

An example code of a _SysCfg_ file can be found in the [API specification section](api_specification.md#action-execution).

## Parameters
Parameters are used to configure the system behavior in the _build_ or the _run_ actions of an experiment. They are declared in the _SysDef_ file with a **default value** that allows a _dry run_ of the system. This helps to test a system out-of-the-box without the need to know details about the specific parameters and their impact.

### Simple Parameters
Simple parameters are defined as basic key-value pair in JSON definition. The value can any of the _primitive data types_:

- String
- Number (signed, unsigned, floating point)
- Boolean

!!! NOTE
    Array (list) and Object (dict) parameters are not supported.

### Complex Parameters
Complex parameters provide additional features like constraining of the value to only allow a restricted range of numeric values or a selection from pre-defined distinct choices.
Additionally, a description can be added to describe the purpose of the parameter.

#### Enumeration Parameters
Enumeration parameters allow only specific values which can be strings, booleans, or numbers. The following example demonstrates an enumeration which can have either `instruction_accurate` or `cycle_accurate` as value.

```json
{
  "uarch":{
    "default_value": "instruction_accurate",
    "meta": {
      "values": ["instruction_accurate", "cycle_accurate"]
    },
    "description": "micro architectural representation of core simulator"
  }
}
```

In the SysCfg, just a value is used instead of the sub-object.
The Runtime Manager checks the provided value against the options given in the SysDef and signals an error if the choice is not valid.


#### Range Parameters
The range parameter definition is similar to the enumeration parameters. It allows restricting the values of a numeric parameter to a pre-defined range.

```json
   "timeout": {
      "default_value": 30,
      "meta": {
        "lower": 5,
        "upper": 120
      },
      "description": "Configure a timeout in seconds"
    }
```

In the SysCfg, the sub-object is replaced by a single entry containing the value to be used for the parameter.
The Runtime Manager checks if the provided value is in the range specified in the SysDef and signals an error if this is not the case.


#### File Parameters
A special configuration option are **file parameters** that allow providing a file to an experiment. A common use-case are embedded software binaries that should be executed on a programmable model. Another use-case are stimulus files that are read by the system to test its behavior in different conditions or for fault-injection scenarios.

Since the process to transfer files from the front-end of the SUNRISE Runtime Manager to the system is different from simple parameters, the entry `"is_file": true` must be present in the `meta` block to identify file parameters.
The following example shows a file parameter with the name `app` and its default path in `default_value` (can be a relative path to the _sysdef.json_ file or an absolute path of the system container) to a software binary.

```json
"run_parameters": {
  "app": {
    "default_value": "demo_application/demo.elf",
      "meta": {
        "is_file": true
      },
      "description": "App binary to execute on the CPU"
  }
}
```

In the SysCfg, the sub-object is replaced by a single string that holds the path to the file in the execution context. So when the `run_command` or `build_command` receive the SysCfg, the paths to the files for the file parameters are valid as they are. The SysCfg example in the [API specification section](api_specification.md#action-execution) shows the substitution.


## Results
Results objects are generated during the run action in an experiment should have one of the pre-defined types, to enable re-use of analysis tools in the front-end and comparison of experiments.
Results artifacts always are _files_ with a specific _file type_, defined in the SysDef file of a system.

The following **standard** file types are supported:

- `vcd`: Open and established ASCII-based format for waveform data. This can be used for signal tracing.
- `fst`: Waveform data as Fast Signal Trace (FST), used by the _GTKWave_ tool.
- `junit_xml`: A common format to define and structure test results. It is XML-based and described for example [here](https://en.wikipedia.org/wiki/JUnit).
- `gprof`: Results from the [gprof](https://ftp.gnu.org/old-gnu/Manuals/gprof-2.9.1/html_mono/gprof.html) tool which is a text file with _flat profile_ and _call graph_ sections.

Additionally, **custom types** are defined for SUNRISE (details below):

- `performance` for CPU performance metrics
- `simulation_speed` for simulation performance metrics
- `profile_csv` for function profiling

If a system produces results that are not of any the types specified before, the placeholder for **generic** files an be used:

- `binary`: To represent any binary data, for which a specific tool or application is needed to process or visualize it.
- `text`: For text files, that can contain for example log output in a non-standardized shape.

### Custom Types
#### Performance
The result type `performance` is used to record performance metrics of a CPU. It contains the cycle and instruction count of the executed system. Additionally, a core clock frequency in _Hz_ can be added to captured time or just for reference. All fields are optional.

The file could look like:

```json
{
  "cycles": 601607,
  "instructions": 342413,
  "frequency_hz": 1e+08
}
```

!!! TIP
    Exponential notation is allowed to support large numbers.

#### Simulation Speed
The `simulation_speed` type holds metrics for a simulation execution performance. It contains the simulated time and the execution (wall-clock) time in seconds.

An example is:

```json
{
  "simulated_time_sec": 0.0103042,
  "execution_time_sec": 0.154422
}
```

This would mean a the _real-time factor (RTF)_, execution time divided by simulated time, is approximately 15. The Pydantic model contains a method to calculate the RTF.

#### Function Profiling Table
For custom function profiling tools, it is recommended to save the result in this table format.
The data is structured with one row per function and the profiling metrics in the columns. The rows are not sorted by any column.

The **columns** are:

- _function_: Name of the function
- _address_: Address of the function in the memory (hexadecimal, leading _0x_ will be truncated.)
- _count_: Number of calls of the function
- _percent_: Share of cycles spent in the function in relation to the total cycles
- _self_cycles_: Number of cycles spend in the functions itself (without sub-calls)
- _cumulative_cycles_: Overall number of cycles spend in the function including sub-calls

The overall **table** looks like this:

| function      | address  | count | percent | self_cycles | cumulative_cycles |
|:---|:---|:---|:---|:---|:---|
| some_function | 20001470 | 15    | 25.2    | 177         | 508 |
| another_func  | 20000630 | 1     | 44.0    | 167         | 885 |
| third_fn      | 2000052a | 4     | 59.1    | 59          | 1190 |

The table is stored and transferred as a **CSV file**, so the columns are separated with commas:

```csv
function,address,count,percent,self_cycles,cumulative_cycles
some_function,20001470,15,25.2,177,508
another_function,20000630,1,44.0,167,885
third_fn,2000052a,4,59.1,59,1190
```
