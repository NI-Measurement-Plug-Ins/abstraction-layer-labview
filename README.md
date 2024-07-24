
# Abstraction Layer in Measurement Plug-In for LabVIEW

This repository contains the workflow and measurement plug-in examples showcasing how to implement
Hardware Abstraction Layer (HAL) and Functional Abstraction Layer (FAL) in the measurement plug-ins.

**Table of Contents**

- [Abstraction Layer in Measurement Plug-In for LabVIEW](#abstraction-layer-in-measurement-plug-in-for-labview)
  - [Overview](#overview)
    - [Hardware Abstraction Layer](#hardware-abstraction-layer)
    - [Functional Abstraction Layer](#functional-abstraction-layer)
  - [Software Dependencies](#software-dependencies)
  - [Hardware Dependencies](#hardware-dependencies)
  - [Getting Started](#getting-started)
  - [Create and Update NI Package Manager Feeds](#create-and-update-ni-package-manager-feeds)

## Overview

### Hardware Abstraction Layer

Hardware Abstraction Layer (HAL) enables users to develop applications agnostic of instrument models
of a type (like DMM). HAL in measurement plug-ins allows users to work with various instrument
models without modifying the implementation. This HAL implementation leverages pins from the pin map.

### Functional Abstraction Layer

The Functional Abstraction Layer (FAL) is a higher-level abstraction layer that provides a more
functional view of the system. It focuses on abstracting the functionality rather than the
hardware, allowing software components to interact with each other through well-defined interfaces.

## Software Dependencies

- LabVIEW 2021 SP1 or later
- InstrumentStudio Professional 2024 Q3 or later
- NI-DCPower 2024 Q2 or later
- NI-DMM 2023 Q1 or later
- NI-VISA 2024 Q1 or later
- NI-488.2 and/or NI-Serial
- Recommended: TestStand 2021 SP1 or later

## Hardware Dependencies

Supported instrument models:

- NI-DCPower (e.g. PXIe-4141)
- NI-DMM (e.g. PXIe-4081).
- Keysight 34401A DMM.

## Getting Started

- You can refer to the [HAL in Measurement Plug-in](https://github.com/NI-Measurement-Plug-Ins/abstraction-layer-labview/blob/main/Source/HAL%20Implementation/README.md) to understand the workflow for implementing HAL for measurement plug-ins.
- You can refer to the [FAL in Measurement Plug-in](https://github.com/NI-Measurement-Plug-Ins/abstraction-layer-labview/blob/main/Source/FAL%20Implementation/README.md) to understand the workflow for implementing FAL for measurement plug-ins.


## Create and Update NI Package Manager Feeds

- Packages for various measurement plugins are incorporated into an NI Package Manager feed,
  allowing users to install new packages or receive updates to existing ones by subscribing to the
  feed.

- The feeds for Measurement plugins are maintained under the attached repo
  [`package-manager-feeds`](https://github.com/NI-MeasurementLink-Plug-Ins/package-manager-feeds).

- Please follow the procedure mentioned in attached document for adding new packages or updating new
  versions of existing packages to the feed
  [`README.md`](https://github.com/NI-MeasurementLink-Plug-Ins/package-manager-feeds/blob/main/package-feed-updater/README.md).
