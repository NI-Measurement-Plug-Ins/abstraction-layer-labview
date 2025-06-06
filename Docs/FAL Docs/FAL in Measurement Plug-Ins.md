# FAL in Measurement Plug-In

- [FAL in Measurement Plug-In](#fal-in-measurement-plug-in)
  - [What is FAL?](#what-is-fal)
  - [Pre-requisites](#pre-requisites)
  - [Steps to create new FAL based measurement](#steps-to-create-new-fal-based-measurement)
  - [Steps to migrate FAL implementations from other frameworks](#steps-to-migrate-fal-implementations-from-other-frameworks)

## What is FAL?

The Functional Abstraction Layer (FAL) is a higher-level abstraction layer that provides a more functional view of the system. It focuses on abstracting the functionality of the instrument rather than the instrument type, allowing software components to interact with various instrument models through well-defined interfaces without modifying the implementation.

## Pre-requisites

- Intermediate-level expertise in [LabVIEW - Object-Oriented Programming (OOP)](https://www.ni.com/en/support/documentation/supplemental/06/labview-object-oriented-programming--the-decisions-behind-the-de.html).
- Understanding of the [session management](https://www.ni.com/docs/en-US/bundle/measurementplugins/page/session-management.html) in the measurement plug-ins.
- Fundamentals of FAL.

## Steps to create new FAL based measurement

1. Create a measurement plug-in by following the steps mentioned in the [Developing a measurement plug-in with LabVIEW](https://github.com/ni/measurement-plugin-labview?tab=readme-ov-file#developing-a-labview-measurement) guide.

2. Ensure to follow the recommended directory structure for FAL shown below, to efficiently organize the files related to the FAL implementation.

    ``` bash

    <fal_root_directory>
        ├───Functions
        │   ├───<function_1>
        │   │   ├───Methods
        │   │   └───Utility
        │   ├───<function_2>
        │   │    ├───Methods
        │   │    └───Utility
        │   .
        │   .
        │   .
        │   └───<function_n>
        │        ├───Methods
        │        └───Utility
        ├───Instruments
        │   ├───Abstract_Instrument
        │   │   ├───Accessors
        │   │   ├───controls
        │   │   ├───Methods
        │   │   └───Utility
        │   └───Instrument_Models
        │       ├───<instrument_model_1>
        │       │   ├───controls
        │       │   ├───Methods
        │       │   │   ├───<function_1>
        │       │   │   ├───<function_2>
        │       │   │   └───Session Methods
        │       │   └───subVIs
        │       ├───<instrument_model_2>
        │       │   ├───controls
        │       │   └───Methods
        │       │       ├───<function_1>
        │       │       └───Session Methods
        │       .
        │       .
        │       .
        │       └───<instrument_model_n>
        │           ├───controls
        │           └───Methods
        │               ├───<function_1>
        │               ├───<function_2>
        │               └───Session Methods
        └───Reusables
        
    ```

   **Example:**

    ![Recommended Directory Structure](<./FAL Images/Directory Structure.png>)

3. **Copy the Abstract_Instrument class and Reusables**  
   1. Copy the [`Abstract_Instrument`](https://github.com/NI-Measurement-Plug-Ins/abstraction-layer-labview/tree/main/Source/FAL%20Implementation/FAL/Instruments/Abstract_Instrument) folder into the `Instruments` directory, and the [`Reusables`](https://github.com/NI-Measurement-Plug-Ins/abstraction-layer-labview/tree/main/Source/FAL%20Implementation/FAL/Reusables) folder parallel to the `Instruments` directory. Make sure to preserve all contents of both folders.
   2. Add `Abstract_Instrument.lvclass` from the `Abstract_Instrument` folder and `Get Pin Information.lvlib` from the `Reusables` folder to your LabVIEW project that contains the created measurement plug-in.
   3. Ensure that the `Abstract_Instrument` class inherits the **ISession Factory** interface located at `<vi.lib>\Plug-In SDKs\Sessions\Instrument\ISession Factory\ISession Factory.lvclass`.
   4. Open `Get Instrument Path.vi` from the `Reusables` folder and replace the term **Instrument_Models** in the base path with the actual name of the directory that contains your instrument child classes. This VI takes the `instrument_type_id` as input and returns the path to the corresponding child class directory.

      ![Get Instrument Path](<./FAL Images/Get Instrument Path.png>)

   > **Note:**  
   > The expected folder structure for any FAL-based measurement plug-in should have the **Reusables** folder placed parallel to the **Instruments** folder, where the **Instruments** folder contains the abstract instrument class and the child classes created for different instrument models.

4. **Create function interfaces**  
   1. In your LabVIEW project, create interfaces for each required functionality (for example, `Measure_Voltage.lvclass`). These are called Function Interfaces.
   2. Save the created Function Interfaces in the `Functions` folder.

   ![Base class and Function Interface](<FAL Images/Base and Function class.png>)

5. Right-click each created interface and select `New` → `VI from Dynamic Dispatch Template` to create the necessary dynamic dispatch VIs that implement the required functionality for the interface (for example, `Measure_Voltage.vi`).

6. For each function interface, implement [Utility](https://github.com/NI-Measurement-Plug-Ins/abstraction-layer-labview/tree/main/Source/FAL%20Implementation/FAL/Functions/Measure_Voltage/Utility) VIs that typecast the `Abstract_Instrument` object to the corresponding function interface object as shown below:

    ![Get 1 Object](<./FAL Images/Get 1 Object.png>)

7. **Create instrument child classes**  
    1. Create child classes for instruments of different types and models, inheriting the `Abstract_Instrument.lvclass` as parent class and required function interface(s) as parent interface(s), as shown in the class hierarchy image below. Refer to this [example](https://github.com/NI-Measurement-Plug-Ins/abstraction-layer-labview/tree/main/Source/FAL%20Implementation) for more information.
    2. **The name of each child class and its directory must match the instrument type ID configured for that instrument model in the pin map file.**
    3. Below are the standard directory names followed for NI instruments:

        | Instrument type     | Directory name      |
        |---------------------|---------------------|
        | NI-DCPower          | niDCPower           |
        | NI-DMM              | niDMM               |
        | NI-Digital Pattern  | niDigitalPattern    |
        | NI-SCOPE            | niSCOPE             |
        | NI-FGEN             | niFGEN              |
        | NI-DAQmx            | niDAQmx             |
        | NI-SWITCH           | niRelayDriver       |

    ![FAL Class Hierarchy](<./FAL Images/FAL Class Hierarchy.png>)

8. **Override session and function methods in child classes**  
    1. For NI instruments, override only the **Initialize Session.vi** session method and the dynamic dispatch methods of the function interfaces inherited by the child class.
    2. For custom instruments, override all session methods and the dynamic dispatch methods of the function interfaces inherited by the child class. Example session methods for a Keysight DMM include:
        - **Initialize Session.vi** - Initializes driver session for the instrument.
        - ***Initialize MeasurementLink Session.vi*** - Creates a new session using the session initialization parameters. If the session represents a remote session, initialize and close session behavior determines whether creating the local session creates a new session on the server or attaches to an existing session on the server.

            ![Initialize MeasurementLink Session](<FAL Images/KeysightDmm Initialize MeasurementLink Session.png>)

        - ***Get Provided Interface and Service Class.vi*** - Returns the provided interface and service class that will be used to query the NI Discovery service for the address and port of the instrument's gRPC server. Do not implement/override this VI if the instrument does not use a gRPC server.

            ![Get Provided Interface and Service Class](<FAL Images/KeysightDmm Get Provided Interface and Service Class.png>)

        - ***Close MeasurmentLink Session.vi*** - Closes the local session. If the session represents a remote session, initialize and close session behavior determines whether closing the local session closes the server session or detaches from the server session.

            ![Close MeasurementLink Session](<FAL Images/KeysightDmm Close MeasurementLink Session.png>)

    > **Note**  
    > The function methods can vary based on the required functionality and hence users can have their own function methods defined for both the function interfaces and child classes.

9. **Define plug-in I/O and update logic**  
    1. Define the inputs and outputs in the measurement plug-in and update `Get Type Specializations.vi` to populate pin information from the pin map file.
    2. Use the following APIs in your measurement logic:  
        - **Initialize Pin.vi** – Polymorphic VI for reserving and initializing instrument sessions. Returns an array of `Abstract_Instrument` objects based on the provided `pin or relay name(s)`.
        - **Get \<function-name\> Object.vi** – Polymorphic VI for typecasting each `Abstract_Instrument` object to the appropriate function interface object.
        - **Function interface dynamic dispatch API** – Use the relevant function interface methods (such as `Measure_Voltage.vi`) to perform the required measurement operations.
        - **Close Session** and **Unreserve Session** – Close and unreserve all instrument sessions created with the session reservation object.

    ![Measurement Logic](<FAL Images/Measurement Logic.png>)

## Steps to migrate FAL implementations from other frameworks

1. Create a measurement plug-in by following the steps mentioned in the [Developing a measurement plug-in with LabVIEW](https://github.com/ni/measurement-plugin-labview?tab=readme-ov-file#developing-a-labview-measurement) guide.
2. Add your existing FAL classes to the LabVIEW project that contains the created measurement plug-in.
3. Update the abstract instrument base class of your instrument classes to inherit from the **ISession Factory** interface, located at `<vi.lib>\Plug-In SDKs\Sessions\Instrument\ISession Factory\ISession Factory.lvclass`.
4. Follow steps 3-9 of [Steps to create new FAL based measurement](#steps-to-create-new-fal-based-measurement) for migrating your existing FAL implementations to a measurement plug-in compatible, class-based FAL.
