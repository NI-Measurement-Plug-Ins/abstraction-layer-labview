# FAL in Measurement Plug-In

- [FAL in Measurement Plug-In](#fal-in-measurement-plug-in)
  - [What is FAL?](#what-is-fal)
  - [Pre-requisites](#pre-requisites)
  - [Steps to create new FAL based measurement](#steps-to-create-new-fal-based-measurement)
  - [Steps to migrate FAL implementations from other frameworks](#steps-to-migrate-fal-implementations-from-other-frameworks)

## What is FAL?

The Functional Abstraction Layer (FAL) is a higher-level abstraction layer that provides a more functional view of the system. It focuses on abstracting the functionality of the instrument rather than the instrument type, allowing software components to interact with various instrument models through well-defined interfaces without modifying the implementation.

## Pre-requisites

- Intermediate working experience in LabVIEW - Object-Oriented Programming (OOP).
- Understanding of the session management in the measurement plug-ins.
- Fundamental knowledge of FAL.

## Steps to create new FAL based measurement

1. Create a measurement plug-in by following the steps mentioned in [Developing a measurement plug-in with LabVIEW](https://github.com/ni/measurement-plugin-labview?tab=readme-ov-file#developing-a-labview-measurement)
2. Copy the `Abstract_Instrument` base class and its methods into the project. Ensure to inherit the Abstract_Instrument class from `ISession Factory` interface.
3. Create function interface and implement the dynamic dispatch method for the required functionality.
    ![Base class and Function Interface](<FAL Images/Base and Function class.png>)
4. Implement a VI in the function interface that typecasts the `Abstract_Instrument` class object to function interface object. Refer [Utility](https://github.com/NI-Measurement-Plug-Ins/abstraction-layer-labview/tree/main/Source/FAL%20Implementation/FAL/Functions/Measure_Voltage/Utility) for more details.
5. Create instrument child class `<instrument_type_id>` inherited from `Abstract_Instrument` class and the `function interface`. The instrument child class name should match with the instrument type id in the pin map file and the directory name of the instrument child class. The directory names for different NI instrument types are:

   Instrument type | Directory name
   --- | ---
   NI-DCPower | niDCPower
   NI-DMM | niDMM
   NI-Digital Pattern | niDigitalPattern
   NI-SCOPE | niSCOPE
   NI-FGEN | niFGEN
   NI-DAQmx | niDAQmx
   NI-SWITCH | niRelayDriver
6. For NI instruments, override the **Initialize Session.vi** session method in the instrument child class.
7. For custom instruments, override all the session methods present in `Abstract_Instrument` class. The session methods for a Keysight DMM include:
    - **Initialize Session.vi** - Initializes driver session for the instrument.
    - ***Initialize MeasurementLink Session.vi*** - Creates a new session using the session initialization parameters. If the session represents a remote session, initialize and close session behavior determines whether creating the local session creates a new session on the server or attaches to an existing session on the server.

        ![Initialize MeasurementLink Session](<FAL Images/KeysightDmm Initialize MeasurementLink Session.png>)

    - ***Get Provided Interface and Service Class.vi*** - Returns the provided interface and service class that will be used to query the NI Discovery service for the address and port of the instrument's gRPC server. Do not implement/override this VI if the instrument does not use a gRPC server.

        ![Get Provided Interface and Service Class](<FAL Images/KeysightDmm Get Provided Interface and Service Class.png>)

    - ***Close MeasurmentLink Session.vi*** - Closes the local session. If the session represents a remote session, initialize and close session behavior determines whether closing the local session closes the server session or detaches from the server session.

        ![Close MeasurementLink Session](<FAL Images/KeysightDmm Close MeasurementLink Session.png>)

8. Create the overriding method for the dynamic dispatch function interface method in the instrument child class. The function methods can vary based on the function type and hence, the users can have their own function methods defined for both their function interface and child classes respectively.
9. Copy the VIs under [Reusables](https://github.com/NI-Measurement-Plug-Ins/abstraction-layer-labview/tree/main/Source/FAL%20Implementation/FAL/Reusables) into the project. Make sure to update the logic of `Get Instrument Path.vi` to get the path of the child class directory.
10. Define the inputs and outputs in the measurement plug-ins and update the `Get Type Specialization.vi` to populate the pin information from pin map file.
11. Update the measurement logic with the below APIs:
    - ***Initialize Pin.vi*** - a polymorphic VI for reserving measurement plug-ins and driver sessions. Returns an array of `Abstract_Instrument` objects based on pin input.
    - ***Get \<function-name\> Object.vi***, a polymorphic VI to typecast the `Abstract_Instrument` object to function interface object.
    - ***Function interface dynamic dispatch API*** for the instrument child class to override the function methods.
    - ***Close Session*** and ***Unreserve Session*** for closing and unreserving the measurement plug-ins sessions.
    ![Measurement Logic](<FAL Images/Measurement Logic.png>)

## Steps to migrate FAL implementations from other frameworks

1. Create a measurement plug-in by following the steps mentioned in [Developing a measurement plug-in with LabVIEW](https://github.com/ni/measurement-plugin-labview?tab=readme-ov-file#developing-a-labview-measurement)
2. Copy the existing FAL classes to the project and follow from step 2-10 of [Steps to create new FAL based measurement](#steps-to-create-new-fal-based-measurement) for migrating the existing FAL implementations to measurement plug-in.
