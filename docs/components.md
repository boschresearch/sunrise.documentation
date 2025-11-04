# Components

## Overview
The SUNRISE framework consists of the following components:

- **Runtime Manager:** The core component that operates a server, which is accessed by users via the REST-based [_Evaluation API_ (EvalAPI)](api_specification.md#evaluation-api-evalapi) and interacts with the _systems_ through the [_System API (SysAPI)_](api_specification.md#system-api-sysapi).
- **System Storage:** A library of containerized systems, including simulation platforms and their execution dependencies.
- **Front-End:** The interface for users. Can be graphical visualizations, headless scripts or other applications the support the EvalAPI.
- **Back-End:** The environment where _systems_ are executed as containers. Runs are organized in _experiments_ with an underlying state-machine for each, reflecting the generic concept of the [simulation workflow](workflow.md).

![Components Overview](img/sunrise_overview.svg)

## Terminology
Since some terms have multiple meanings in the industry. Refer to the following definitions in the context of SUNRISE:

- **System**: An application, in general any kind of bundled simulation platform with its simulation tool and all required dependencies in form of a container image. A system is operated in a non-interactive way (headless) inside a container to make it manageable by the Runtime Manager. Systems are configurable with _parameters_. More details in the [SysAPI specification](api_specification.md#system-components).
- **Model**: A representation of a hardware component like e.g. a CPU, memory, bus, peripheral or physical plant description in a simulation platform. A system is typically comprised of one or multiple models.
- **Experiment**: A unique temporary session, that holds a working environment, owned by a user to evaluate one use-case.

### Roles
The SUNRISE methodology distinctly separates the roles of three perspectives. Separating these responsibilities allows each specialist to focus on their specific domain.

The **user** implements application-specific front-ends against the [EvalAPI](api_specification.md#evaluation-api-evalapi) with no need to know about the technical details related to the setup of the simulation model, tools and environment. The realization of a front-end can be rather generic, providing options to modify any configuration parameter available in the system and run through the execution workflow, or it can be designed specifically for a use case by only accessing a subset of parameters and visualizing the relevant experiment results and metrics. Separate solutions for front-ends can be used in parallel for multiple users with different scopes.

The system **integrator** is responsible for implementing the [SysAPI](api_specification.md#system-api-sysapi) to make simulations accessible and executable in a defined manner that is fully independent of the application it is used for. This role involves structuring the systems in the [specified way](api_specification.md#system-api-sysapi) and setting up a container image with all runtime dependencies, allowing a system to be seamlessly integrated into the SUNRISE framework. As the interface is standardized and common, there is no need for the integrator to (re-)train users on specific simulation technologies.

The overall hosting of the framework is managed by an **operator**, who is responsible for enabling containerized workload deployment and ensuring that the necessary computation resources are available. In this perspective, neither the details of the simulation tools nor the specific use case are relevant, as all components are encapsulated within containers.
