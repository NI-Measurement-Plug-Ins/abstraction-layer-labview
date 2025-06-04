# FAL in Measurement Plug-In

- [FAL in Measurement Plug-In](#fal-in-measurement-plug-in)
  - [What is FAL?](#what-is-fal)
  - [Pre-requisites](#pre-requisites)
  - [Steps to create new FAL based measurement](#steps-to-create-new-fal-based-measurement)
  - [Steps to migrate FAL implementations from other frameworks](#steps-to-migrate-fal-implementations-from-other-frameworks)

## What is FAL?

The Functional Abstraction Layer (FAL) is a higher-level abstraction layer that provides a more functional view of the system. It focuses on abstracting the functionality of the instrument rather than the instrument type, allowing software components to interact with various instrument models through well-defined interfaces without modifying the implementation.

## Pre-requisites

- Intermediate working experience in [LabVIEW - Object-Oriented Programming (OOP)](https://www.ni.com/en/support/documentation/supplemental/06/labview-object-oriented-programming--the-decisions-behind-the-de.html).
- Understanding of the [session management](https://www.ni.com/docs/en-US/bundle/measurementplugins/page/session-management.html) in the measurement plug-ins.
- Fundamental knowledge of FAL.

## Steps to create new FAL based measurement

![Block Diagram](<FAL Images/Block Diagram.png>)

1. Create a measurement plug-in by following the steps mentioned in [Developing a measurement plug-in with LabVIEW](https://github.com/ni/measurement-plugin-labview?tab=readme-ov-file#developing-a-labview-measurement)
2. Copy the [`Abstract_Instrument`](https://github.com/NI-Measurement-Plug-Ins/abstraction-layer-labview/tree/main/Source/FAL%20Implementation/FAL/Instruments/Abstract_Instrument) class along with its `Accessors`, `controls`, `Methods` and `Utility` files into the LabVIEW project containing the measurement plug-in. Ensure the `Abstract_Instrument` class inherits the `ISession Factory` interface located at `<vi.lib>\Plug-In SDKs\Sessions\Instrument\ISession Factory\ISession Factory.lvclass` as the parent interface.
3. In the LabVIEW project, create interfaces for the required functionality (ex: Measure_Voltage.lvclass).
4. Right click on the created interface and select `New` -> `VI from Dynamic Dispatch Template` to create dynamic dispatch VIs for the methods required to perform the function (ex: Measure_Voltage.vi).
    ![Base class and Function Interface](<FAL Images/Base and Function class.png>)
5. For all created function interfaces, implement the [Utility](https://github.com/NI-Measurement-Plug-Ins/abstraction-layer-labview/tree/main/Source/FAL%20Implementation/FAL/Functions/Measure_Voltage/Utility) functions. The `Utility` functions typecast the `Abstract_Instrument` class object into the required function interface object as shown.  
    ![Get 1 Object](<./FAL Images/Get 1 Object.png>)
6. Create instrument child classes that inherit `Abstract_Instrument` class as the parent class and the required function interface(s) as the parent interface(s).
   1. **The instrument child class name should match with the `instrument type id` in the pin map file and the directory name of the instrument child class.**
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

7. For NI instruments, override the **Initialize Session.vi** session method of the `Abstract_Instrument` class and the dynamic dispatch methods of the function interfaces in the child class.
8. For custom instruments, override all the session methods present in `Abstract_Instrument` class and the dynamic dispatch methods of the function interfaces. The session methods for a Keysight DMM include:
    - **Initialize Session.vi** - Initializes driver session for the instrument.
    - ***Initialize MeasurementLink Session.vi*** - Creates a new session using the session initialization parameters. If the session represents a remote session, initialize and close session behavior determines whether creating the local session creates a new session on the server or attaches to an existing session on the server.

        ![Initialize MeasurementLink Session](<FAL Images/KeysightDmm Initialize MeasurementLink Session.png>)

    - ***Get Provided Interface and Service Class.vi*** - Returns the provided interface and service class that will be used to query the NI Discovery service for the address and port of the instrument's gRPC server. Do not implement/override this VI if the instrument does not use a gRPC server.

        ![Get Provided Interface and Service Class](<FAL Images/KeysightDmm Get Provided Interface and Service Class.png>)

    - ***Close MeasurmentLink Session.vi*** - Closes the local session. If the session represents a remote session, initialize and close session behavior determines whether closing the local session closes the server session or detaches from the server session.

        ![Close MeasurementLink Session](<FAL Images/KeysightDmm Close MeasurementLink Session.png>)

    > [!NOTE]
    > The function methods can vary based on the required functionality and hence, users can have their own function methods defined for both the function interfaces and child classes respectively.
9. Copy the VIs under [Reusables](https://github.com/NI-Measurement-Plug-Ins/abstraction-layer-labview/tree/main/Source/FAL%20Implementation/FAL/Reusables) folder.
    1. Update the base path in the `Get Instrument Path.vi` to specify the actual name of the directory containing the child class folders instead of `Instrument_Models`. This VI is used to get the path to the child class directory by providing the `instrument_type_id` of the child class.  
        ![Get Instrument Path](<./FAL Images/Get Instrument Path.png>)
    > [!NOTE]
    > The expected folder structure for any FAL-based measurement plug-in should have the `Reusables` folder parallel to the `Instruments` folder, which contains the instrument base and child classes.
10. Define the inputs and outputs in the measurement plug-ins and update the `Get Type Specialization.vi` to populate the pin information from pin map file.
11. Update the measurement logic with the below APIs:
    - ***Initialize Pin.vi*** - a polymorphic VI for reserving measurement plug-ins and driver sessions. Returns an array of `Abstract_Instrument` objects based on pin input.
    - ***Get \<function-name\> Object.vi***, a polymorphic VI to typecast the `Abstract_Instrument` object to function interface object.
    - ***Function interface dynamic dispatch API*** for the instrument child class to override the function methods.
    - ***Close Session*** and ***Unreserve Session*** for closing and unreserving the measurement plug-ins sessions.
    ![Measurement Logic](<FAL Images/Measurement Logic.png>)

## Steps to migrate FAL implementations from other frameworks

1. Create a measurement plug-in by following the steps mentioned in [Developing a measurement plug-in with LabVIEW](https://github.com/ni/measurement-plugin-labview?tab=readme-ov-file#developing-a-labview-measurement)
2. Copy the existing FAL classes to the project and follow from step 2-11 of [Steps to create new FAL based measurement](#steps-to-create-new-fal-based-measurement) for migrating the existing FAL implementations to measurement plug-in.
