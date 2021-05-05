# The Sysops Squad Architectural Kata

Please note that all views are documented presented [C4 model](https://c4model.com). All diagrams are supplied with a key explaining meaning of each shape on the diagram.

## Business Case

The business case is described [here](BusinessCase.md)

## Requirements

Functional and quality requirements are described in [this page ](Requirements.md)

## Baseline Architecture

This section describes the architecture of the current ticket system.

The current ticket system demonstrates very poor characteristics of availability, maintainability, deployability and performance.

The following diagram depicts the containers diagram of the current ticket system:

![Baseline Architecture](images/baseline.jpg "Baseline Architecture")

## Target Architecture

This section describes the target system architecture

### Use Case Model

The following diagram shows mapping of architecture characteristics requirements on use cases based on discovered requirements:

![Use Case Model](images/use-case-model.jpg "Use Case Model")


### System Context

The system context is depicted in the following diagram:

![System Context](images/system-context.jpg "System Context")

### Containers

The containers diagram that follows shows the high-level shape of the software architecture and how responsibilities are distributed across containers. It also shows the major technology choices and how the containers communicate with one another.

Note that small stickers mark which containers have highest requirement on the specified quality.

![Containers](images/containers.jpg "Containers")

### Sequences

### Deployment
