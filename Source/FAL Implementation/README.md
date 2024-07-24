# Source Measure DC Voltage using LabVIEW FAL

- [Source Measure DC Voltage using LabVIEW FAL](#source-measure-dc-voltage-using-labview-fal)
  - [Source Measure DC Voltage Measurement](#source-measure-dc-voltage-measurement)
  - [Features](#features)
  - [Required Software and Drivers](#required-software-and-drivers)
  - [Required Hardware](#required-hardware)
  - [FAL Class Hierarchy](#fal-class-hierarchy)
  - [How to simulate NI-DMM, NI-DCPower and Keysight DMM?](#how-to-simulate-ni-dmm-ni-dcpower-and-keysight-dmm)
    - [To simulate NI instruments](#to-simulate-ni-instruments)
    - [To simulate Keysight DMM via VISA-TCP\IP simulation](#to-simulate-keysight-dmm-via-visa-tcpip-simulation)
  - [Note](#note)

## Source Measure DC Voltage Measurement

This measurement plug-in example sources DC voltage value using NI-DCPower and measures back the DC voltage level from either NI-DCPower, NI-DMM or Keysight DMM instruments through Functional Abstraction Layer (FAL).

Select the **Source Pin** name as `VDD_NI_DCPower` to source the NI-DCPower instrument and select the **Measure Pin** name either as `VDD_NI_DMM`, `VDD_NI_DCPower` or `VDD_Keysight_DMM` to measure from NI-DMM, NI-DCPower or Keysight DMM instrument respectively. Select the voltage level, source delay and range. Specify the resolution in digits of precision. The measured value will be displayed in the measurement indicator.

## Features

- Uses the NI-DMM LabVIEW API, NI-DCPower LabVIEW API and Agilent_34401A-DMM API.
- Pin-aware, supporting multiple sessions, one Source Pin and one Measure Pin.
  - Uses the same selected measurement function and range for all selected pin/site combinations.
- Includes InstrumentStudio project files to run a measurement.
- Uses the NI gRPC Device Server to allow sharing instrument sessions with other measurement services when running measurements from TestStand.

## Required Software and Drivers

- LabVIEW 2021 SP1 64-bit or later
- InstrumentStudio 2024 Q3 or later
- TestStand 2021 SP1 or later
- NI-DMM 2023 Q1 or later
- NI-DCPower 2023 Q1 or later
- [Agilent 34401 LabVIEW Instrument Driver](https://sine.ni.com/apps/utf8/niid_web_display.download_page?p_id_guid=014E7F05D12C6F8BE0440003BA7CCD71)

## Required Hardware

This example requires :

- NI-DMM (e.g. PXIe-4081).
- NI-DCPower (e.g. PXIe-4141).
- Keysight 34401A or compatible DMM.
  - By default, the pin map included with this example uses the instrument name
  `VISA-DMM`. Please update the alias of Keysight 34401A DMM to `VISA-DMM`in NI MAX.

## FAL Class Hierarchy

The object-oriented FAL library can be used to implement new instrument models.

The FAL library implementation involves the following modules or classes:

1. Abstract_Instrument - The base class for the instrument models.
2. Function Interface - Functionally abstracts the instruments by providing function interfaces for the instrument child classes to inherit.
3. Instrument Model - The implementations of various categories of instrument model classes ([niDMM](../../labview_fal/FAL/Instruments/niDMM), [KeysightDmm](../../labview_fal/FAL/Instruments/KeysightDmm) and [niDCPower](../../labview_fal/FAL/Instruments/niDCPower)).
    ![FAL Class Hierarchy](<FAL Class Hierarchy.png>)

## How to simulate NI-DMM, NI-DCPower and Keysight DMM?

### To simulate NI instruments

- Launch NI MAX.
- Right-click on Devices and Interfaces in the MAX configuration tree and select `Create New`.
- In the Create New dialog, `Simulated NI-DAQmx Device or Modular Instrument` and click `Finish`.
- Search and Select the device model `PXIe-4141` for NI-DCPower and `PXIe-4081` for NI-DMM and click `OK`.
- The simulated device will now appear in NI MAX under My System > Devices and Interfaces.

### To simulate Keysight DMM via VISA-TCP\IP simulation

- Run the `<repo>\Source\Shared\KeysightDmm Simulation\Simulate Keysight 34401a TCP.vi` with port 50000 and desired timeout in ms.
- Open NI MAX application.
- Expand Devices and Interfaces and Network Devices will be displayed.
- Right click Network Devices and click create a new VISA TCP/IP Resource.
- Select manual entry of raw socket and click next
- Now enter the host address as localhost and port 50000 (consistent with port number in simulation vi).
- Verify Validate opens the port.
- Enter the alias name as `VISA-DMM`
- In pin map verify if the custom instrument is named as `VISA-DMM`

## Note

- This measurement uses the `SourceMeasureDCVoltageFAL.pinmap` file, which includes one DC-Power
  instrument and two custom DMM instruments: `GPIB0::3::INSTR (simulated)` and `VISA-DMM(physical)`,
  both identified with the instrument type ID `KeysightDmm`. Currently, the Initialize and Register Sessions.vi and Unregister and Close Sessions.vi
  only support initializing a single session of a specific instrument type ID,  ensure to remove the simulated instrument (GPIB0::3::INSTR) if you have a
  physical instrument (VISA-DMM) connected, or vice versa.
