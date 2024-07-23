# HAL in Measurement Plug-In

- [What is HAL?](#what-is-hal)
- [Pre-requisites](#pre-requisites)
- [Steps to create new HAL based measurement](#steps-to-create-new-hal-based-measurement)
- [Migrate the existing instrument class to Measurement Plug-In](#migrate-the-existing-instrument-class-to-measurement-plug-in)

## What is HAL?

Hardware Abstraction Layer (HAL) enables users to develop software applications agnostic of instrument models of a type. HAL in measurement plug-ins allows users to work with various instrument models without modifying the implementation. This HAL implementation leverages pins from the pin map.

## Pre-requisites

- Intermediate working experience in LabVIEW - Object-Oriented Programming (OOP).
- Understanding of the session management in the measurement plug-ins.
- Fundamental knowledge of HAL.

## Steps to create new HAL based measurement

1. Create a measurement plug-in by following the steps mentioned in [Developing a measurement plug-in with LabVIEW](https://github.com/ni/measurement-plugin-labview?tab=readme-ov-file#developing-a-labview-measurement).
2. Create a base class for a single instrument type and inherit from the `ISession Factory` interface.
    ![Session Factory Inheritance](<HAL Images/Session Factory Inheritance.png>)
3. Implement the below dynamic dispatch methods in the instrument base class that includes:
    - Session methods:
      - ***Initialize MeasurementLink Session.vi*** - Initializes the measurement plug-ins session for the instrument selected.
      - ***Get Instrument Type ID.vi*** - Gets the instrument type ID mentioned in the pin map file for the selected instrument.
      - ***Get Provided Interface and Service Class.vi*** - Returns the provided interface and service class that will be used to query the NI Discovery service for the address and port of the instrument's gRPC server.
      - ***Close MeasurementLink Session.vi*** - Closes the local measurement plug-ins session.
    - Measurement methods:
      - ***Initialize*** - Initializes the instrument session.
      - ***Configure*** - Configures the input parameters for the selected instrument.
      - ***Measure*** - Takes measurement output from the instrument.
4. Implement the VIs under [Utility](../../labview_hal/HAL/Instruments/DMM_Base/Utility) in the instrument base class.
5. Create child classes that inherits from the instrument base class and implement the overriding methods. The instrument child class name should match with the instrument type id in the pin map file and the directory name of the instrument child class. The directory names for different NI instrument types are:

   Instrument type | Directory name
   --- | ---
   NI-DCPower | niDCPower
   NI-DMM | niDMM
   NI-Digital Pattern | niDigitalPattern
   NI-SCOPE | niSCOPE
   NI-FGEN | niFGEN
   NI-DAQmx | niDAQmx
   NI-SWITCH | niRelayDriver
6. For NI instruments, override the measurement methods of the instrument base class.
7. For custom instruments, override both the session and measurement methods present in the instrument base class. The session methods for a Keysight DMM include:
    - ***Initialize MeasurementLink Session.vi*** - Creates a new session using the session initialization parameters. If the session represents a remote session, initialize and close session behavior determines whether creating the local session creates a new session on the server or attaches to an existing session on the server.

    ![Initialize MeasurementLink Session](<HAL Images/KeysightDmm Initialize MeasurementLink Session.png>)

    - ***Get Provided Interface and Service Class.vi*** - Returns the provided interface and service class that will be used to query the measurement plug-ins discovery service for the address and port of the instrument's gRPC server. Do not implement/override this VI if the instrument does not use a gRPC server.

    ![Get Provided Interface and Service Class](<HAL Images/KeysightDmm Get Provided Interface and Service Class.png>)

    - ***Close MeasurementLink Session.vi*** - Closes the local session. If the session represents a remote session, initialize and close session behavior determines whether closing the local session closes the server session or detaches from the server session.

    ![Close MeasurementLink Session](<HAL Images/KeysightDmm Close MeasurementLink Session.png>)

8. Copy the VIs under [Reusables](../../labview_hal/HAL/Reusables). Make sure to update the logic of `Get Instrument Path.vi` to get the path of the child class directory.
9. Define the inputs and outputs in the measurement plug-ins and update the `Get Type Specialization.vi` to populate the pin information from pin map file.
10. Update the Measurement logic with the below APIs:
    - ***Reserve Session.vi*** - a polymorphic VI for reserving either a single pin or multiple pins together.
    - ***Get Pin Object.vi***, a polymorphic VI for getting the `session reservation` and getting either a single or an array of base class instrument objects from `pin or relay name(s)`.
    - ***Measurement APIs*** from the instrument base class that includes `Initialize`, `Configure` and the `Measure` for measuring the instrument.
    - ***Close Session*** and ***Unreserve Session*** for closing and unreserving the measurement plug-ins sessions.
    ![Measurement Logic](<HAL Images/Measurement Logic.png>)

## Migrate the existing instrument class to Measurement Plug-In

1. Create a measurement plug-in by following the steps mentioned in [Developing a measurement plug-in with LabVIEW](https://www.ni.com/docs/en-US/bundle/measurementplugins/page/labview-measurements.html)
2. Copy the existing instrument classes to the project and follow from step 3-10 of [Steps to create new HAL based measurement](#steps-to-create-new-hal-based-measurement) for migrating the existing instrument classes to measurement plug-in.
