# HAL in Measurement Plug-In

- [HAL in Measurement Plug-In](#hal-in-measurement-plug-in)
  - [What is HAL?](#what-is-hal)
  - [Pre-requisites](#pre-requisites)
  - [Steps to create new HAL based measurement](#steps-to-create-new-hal-based-measurement)
  - [Migrate the existing instrument class to Measurement Plug-In](#migrate-the-existing-instrument-class-to-measurement-plug-in)

## What is HAL?

Hardware Abstraction Layer (HAL) enables users to develop software applications agnostic of instrument models of a type. HAL in measurement plug-ins allows users to work with various instrument models without modifying the implementation. This HAL implementation leverages pins from the pin map.

## Pre-requisites

- Intermediate-level expertise in [LabVIEW - Object-Oriented Programming (OOP)](https://www.ni.com/en/support/documentation/supplemental/06/labview-object-oriented-programming--the-decisions-behind-the-de.html).
- Understanding of the [session management](https://www.ni.com/docs/en-US/bundle/measurementplugins/page/session-management.html) in the measurement plug-ins.
- Fundamentals of HAL.

## Steps to create new HAL based measurement

1. Create a measurement plug-in by following the steps mentioned in [Developing a measurement plug-in with LabVIEW](https://github.com/ni/measurement-plugin-labview?tab=readme-ov-file#developing-a-labview-measurement).
2. Create base class for a specific instrument type (ex: DMM) with `LabVIEW Object` as the parent class and `NI Session Management Instrument:ISession Factory` interface located at `<vi.lib>\Plug-In SDKs\Sessions\Instrument\ISession Factory\ISession Factory.lvclass` as the parent interface of the base class.  
    ![Session Factory Inheritance](<HAL Images/Session Factory Inheritance.png>)
3. Right click on the base class and select `New` -> `VI for Override...`.
   1. Create dynamic dispatch VIs for the required session methods of ISession Factory interface in the instrument base class that includes:
      - ***Initialize MeasurementLink Session.vi*** - Initializes the measurement plug-ins session for the instrument selected.
      - ***Get Instrument Type ID.vi*** - Gets the instrument type ID mentioned in the pin map file for the selected instrument.
      - ***Get Provided Interface and Service Class.vi*** - Returns the provided interface and service class that will be used to query the NI Discovery service for the address and port of the instrument's gRPC server.
      - ***Close MeasurementLink Session.vi*** - Closes the local measurement plug-ins session.
   2. In the `Get Instrument Type ID.vi` session method of the instrument base class, implement the logic to extract `instrument type id` from the class name of the provided `session factory in` object as shown below.  
        ![Get Instrument Type ID](<./HAL Images/Get Instrument Type ID.png>)
        > **Note:**  
        > This logic is common for all child classes of a specific instrument type and does not need to be overridden in individual child classes.  
4. Again, right click on the base class and select `New` -> `VI from Dynamic Dispatch Template`.
   1. Create dynamic dispatch VIs for the required measurement methods.
      - ***Initialize*** - Initializes the instrument session.
      - ***Configure*** - Configures the input parameters for the selected instrument.
      - ***Measure*** - Takes measurement output from the instrument.
    > **Note:**  
    > 1. The dynamic dispatch method for closing any instrument session is not implemented here because the `Close Sessions.vi` of the measurement plug-in will close all driver sessions that are reserved in the `Measurement Logic.vi`.  
    > 2. Users can create additional measurement methods as needed. The methods mentioned above are provided as examples.
5. Edit the base class cluster control elements to store the session reservation object of ISession Factory interface, pin name and the channel name as below.  
    ![Base Class Private Data Control](<./HAL Images/Base Class Private Data Control.png>)
6. Create accessors to read and write the private class data elements of the base class.
7. Add the methods under [Utility](https://github.com/NI-Measurement-Plug-Ins/abstraction-layer-labview/tree/main/Source/HAL%20Implementation/HAL/Instruments/DMM_Base/Utility) folder to your instrument base class and update the missing object constants/controls with your base class object.
8. Create child classes that inherit from the instrument base class and implement the overriding methods.
   1. **The instrument child class name should match with the instrument type id in the pin map file and the directory name of the instrument child class.**
   2. The directory names for different NI instrument types are:

      Instrument type | Directory name
      --- | ---
      NI-DCPower | niDCPower
      NI-DMM | niDMM
      NI-Digital Pattern | niDigitalPattern
      NI-SCOPE | niSCOPE
      NI-FGEN | niFGEN
      NI-DAQmx | niDAQmx
      NI-SWITCH | niRelayDriver

9. For NI instruments, override the measurement methods of the instrument base class.
10. For custom instruments, override both the session and measurement methods present in the instrument base class. The session methods for a Keysight DMM include:
    - ***Initialize MeasurementLink Session.vi*** - Creates a new session using the session initialization parameters. If the session represents a remote session, initialize and close session behavior determines whether creating the local session creates a new session on the server or attaches to an existing session on the server.

    ![Initialize MeasurementLink Session](<HAL Images/KeysightDmm Initialize MeasurementLink Session.png>)

    - ***Get Provided Interface and Service Class.vi*** - Returns the provided interface and service class that will be used to query the measurement plug-ins discovery service for the address and port of the instrument's gRPC server. Do not implement/override this VI if the instrument does not use a gRPC server.

    ![Get Provided Interface and Service Class](<HAL Images/KeysightDmm Get Provided Interface and Service Class.png>)

    - ***Close MeasurementLink Session.vi*** - Closes the local session. If the session represents a remote session, initialize and close session behavior determines whether closing the local session closes the server session or detaches from the server session.

    ![Close MeasurementLink Session](<HAL Images/KeysightDmm Close MeasurementLink Session.png>)

11. Copy the VIs under [Reusables](https://github.com/NI-Measurement-Plug-Ins/abstraction-layer-labview/tree/main/Source/HAL%20Implementation/HAL/Reusables) folder.
    1. Update the base path in the `Get Instrument Path.vi` to specify the actual instrument type instead of `DMM_Models`. This VI is used to get the path to the child class directory by providing the `instrument_type_id` of the child class.  
        ![Get Instrument Path](<./HAL Images/Get Instrument Path.png>)
    > **Note:**  
    > The expected folder structure for any HAL-based measurement plug-in should have the `Reusables` folder parallel to the `Instruments` folder, which contains the instrument base and child classes.
12. Define the inputs and outputs in the measurement plug-ins and update the `Get Type Specialization.vi` to populate the pin information from pin map file.
13. Update the Measurement logic with the below APIs:
    - ***Reserve Session.vi*** - a polymorphic VI for reserving either a single pin or multiple pins together.
    - ***Get Pin Object.vi***, a polymorphic VI for getting the `session reservation` and getting either a single or an array of base class instrument objects from `pin or relay name(s)`.
    - ***Measurement APIs*** from the instrument base class that includes `Initialize`, `Configure` and the `Measure` for measuring the instrument.
    - ***Close Session*** and ***Unreserve Session*** for closing and unreserving the measurement plug-ins sessions.
    ![Measurement Logic](<HAL Images/Measurement Logic.png>)

## Migrate the existing instrument class to Measurement Plug-In

1. Create a measurement plug-in by following the steps mentioned in [Developing a measurement plug-in with LabVIEW](https://github.com/ni/measurement-plugin-labview?tab=readme-ov-file#developing-a-labview-measurement)
2. Copy the existing instrument classes to the project and follow from step 3-13 of [Steps to create new HAL based measurement](#steps-to-create-new-hal-based-measurement) for migrating the existing instrument classes to measurement plug-in.
