# Dmm Measurement using LabVIEW HAL

- [Dmm Measurement using LabVIEW HAL](#dmm-measurement-using-labview-hal)
  - [Dmm Measurement](#dmm-measurement)
  - [Features](#features)
  - [Required Software and Drivers](#required-software-and-drivers)
  - [Required Hardware](#required-hardware)
  - [HAL Class Hierarchy](#hal-class-hierarchy)
  - [How to simulate NI-DMM and Keysight DMM?](#how-to-simulate-ni-dmm-and-keysight-dmm)
    - [To simulate NI-DMM](#to-simulate-ni-dmm)
    - [To simulate Keysight 34001A via VISA-TCP\IP simulation](#to-simulate-keysight-34001a-via-visa-tcpip-simulation)
  - [Note](#note)

## Dmm Measurement

This measurement plug-in example acquires a single measurement from either **NI-DMM** or **Keysight DMM** using the Hardware Abstraction Layer (HAL).

Select the pin_name as `NI_DMM_PIN` to measure from NI-DMM or select `Keysight_DMM_Pin` to measure from Keysight DMM. Select the measurement function and range. Specify the resolution in digits of precision. The measured value will be displayed in the measurement indicator.

## Features

- Uses the NI-DMM LabVIEW API and Agilent_34401A-DMM API.
- Pin-aware, supporting one session and one pin
  - Uses the same selected measurement function and range for all selected pin/site combinations.
- Includes InstrumentStudio project files to run a measurement.
- Uses the NI gRPC device server to allow sharing instrument sessions with other
  measurement services when running measurements from TestStand.

## Required Software and Drivers

- LabVIEW 2021 SP1 64-bit or later
- InstrumentStudio 2024 Q3 or later
- TestStand 2021 SP1 or later
- NI-DMM 2023 Q1 or later
- [Agilent 34401 LabVIEW Instrument Driver](https://sine.ni.com/apps/utf8/niid_web_display.download_page?p_id_guid=014E7F05D12C6F8BE0440003BA7CCD71)

## Required Hardware

This example requires :

- NI-DMM (e.g. PXIe-4081).
- Keysight 34401A or compatible DMM.
  - By default, the pin map included with this example uses the instrument name
  `VISA-DMM`. If this doesn't match your instrument name in NI Max, update alias of Keysight 34401A DMM to `VISA-DMM`in NI Max.

## HAL Class Hierarchy

- The DmmMeasurementHAL example in abstraction-layer repository uses HAL for different DMM instruments.
- The HAL example implementation involves the following modules or classes:
  - DMM_Base - The base class for the instrument models.
  - Instrument Model - The implementations of various categories of Dmm instrument classes ([niDMM](../../labview_hal/HAL/Instruments/niDMM) and [KeysightDmm](../../labview_hal/HAL/Instruments/KeysightDmm)).
  ![HAL Class Hierarchy](<HAL Class Hierarchy.png>)

## How to simulate NI-DMM and Keysight DMM?

### To simulate NI-DMM

- Open NI MAX and navigate to My System > Devices and Interfaces.
- Right-click in the Devices and Interfaces section and select Create New.
- Choose Simulated NI-DAQmx Device or Modular Instrument.
- Search for the niDMM model (e.g., PXIe-4081).
- Select the model and click OK
- The simulated niDMM will appear under My System > Devices and Interfaces.

### To simulate Keysight 34001A via VISA-TCP\IP simulation

- Run the `<repo> labview_fal\Utilities\KeysightDmm Simulation\Simulate_Keysight_34401a_TCP.vi` with port 50000 and desired timeout in ms.
- Open NI Max application.
- Create a new VISA TCP/IP Resource under Devices and Interfaces -> NetworkDevices.
- Select manual entry of raw socket and click next
- Now enter the host address as localhost and port 50000 (consistent with port number in simulation vi).
- Verify Validate opens the port.
- Enter the alias name as VISA-DMM
- In pin map verify if the custom instrument is named as VISA-DMM

## Note

- This measurement uses the `DmmMeasurementHAL.pinmap` file, which includes two custom DMM instruments:
  `GPIB0::3::INSTR (simulated)` and `VISA-DMM (physical)`, both identified with the instrument
  type ID `KeysightDmm`. Currently, the Initialize and Register Sessions.vi and Unregister and Close Sessions.vi
  only support initializing a single session of a specific instrument type ID. Therefore, before
  executing the TestStand sequence, ensure to remove the simulated instrument (GPIB0::3::INSTR) if
  you have a physical instrument (VISA-DMM) connected, or vice versa.
