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

1. Create a measurement plug-in by following the steps mentioned in the [Developing a measurement plug-in with LabVIEW](https://github.com/ni/measurement-plugin-labview?tab=readme-ov-file#developing-a-labview-measurement) guide.

2. Ensure to follow the recommended directory structure for HAL shown below, to efficiently organize the files related to the HAL implementation.

   ``` bash

   <hal_root_directory>
      ├───Instruments
      │   ├───<instrument_type>_Base
      │   │   ├───Accessors
      │   │   ├───Methods
      │   │   │   ├───Measurement Methods
      │   │   │   └───Session Methods
      │   │   ├───Utility
      │   │   └───<other related files and folders>
      │   │
      │   └───<instrument_type>_Models
      │       ├───<NI_instrument_type>
      │       │    └───Methods
      │       │       └───Measurement Methods
      │       ├───<instrument_model_1>
      │       │    ├───Methods
      │       │    │   ├───Measurement Methods
      │       │    │   └───Session Methods
      │       │    ├───subVIs
      │       │    └───controls
      │       .
      │       .
      │       .
      │       └───<instrument_model_n>
      │            └───Methods
      │                ├───Measurement Methods
      │                └───Session Methods
      └───Reusables
      
   ```

   **Example:**

   ![Recommended Directory Structure](<./HAL Images/Directory Structure.png>)

3. In `<instrument type>_Base` folder, create a base class for your instrument type that inherits from **LabVIEW Object** and **ISession Factory** interface (located at `<vi.lib>\Plug-In SDKs\Sessions\Instrument\ISession Factory\ISession Factory.lvclass`).

4. **Override ISession Factory methods**  
   1. Right-click the base class and select `New` -> `VI for Override...`.
   2. In the `New Override` dialog, select the following methods from the ISession Factory interface and click `OK` to generate overriding methods:
      - ***Initialize MeasurementLink Session.vi*** - Initializes the measurement plug-ins session for the instrument selected.
      - ***Get Instrument Type ID.vi*** - Gets the instrument type ID mentioned in the pin map file for the selected instrument.
      - ***Get Provided Interface and Service Class.vi*** - Returns the provided interface and service class that will be used to query the NI Discovery service for the address and port of the instrument's gRPC server.
      - ***Close MeasurementLink Session.vi*** - Closes the local measurement plug-ins session.
   3. Save the generated dynamic dispatch VIs in the `Session Methods` folder.
   4. In the generated `Get Instrument Type ID.vi`, implement logic that takes the instrument class object (`session factory in`) as input and returns its corresponding instrument type ID, as shown below:

      ![Get Instrument Type ID](<./HAL Images/Get Instrument Type ID.png>)

   > **Note**  
   > The implementation of **`Get Instrument Type ID.vi`** is common to all child classes of a specific instrument type and does not need to be overridden in individual child classes.

5. **Create measurement operation methods**  
   1. Right-click the base class and select `New` -> `VI from Dynamic Dispatch Template`.
   2. Create dynamic dispatch VIs for required measurement operations and save them in the `Measurement Methods` folder:
      - ***Initialize*** - Initializes the instrument session.
      - ***Configure*** - Configures the input parameters for the selected instrument.
      - ***Measure*** - Takes measurement output from the instrument.

   > **Note**  
   > 1. The dynamic dispatch method for closing any instrument session is not implemented here, as **`Close Sessions.vi`** in the measurement plug-in will close all driver sessions reserved in **`Measurement Logic.vi`**.
   > 2. You can create additional measurement methods as needed; the above are provided as examples.

6. **Edit the private data cluster control of base class**  
   1. Add controls to store the session reservation object, pin name, and channel name as shown below.
   2. The session reservation object can be found at `<vi.lib>\Plug-In SDKs\Clients\Session Management V1\Session Reservation\Session Reservation.lvclass`.

   ![Base Class Private Data Control](<./HAL Images/Base Class Private Data Control.png>)

7. Create accessor VIs to read and write elements of the private data cluster control and save them in the `Accessors` folder.

8. **Copy Reusables**  
   1. Copy the [Reusables](https://github.com/NI-Measurement-Plug-Ins/abstraction-layer-labview/tree/main/Source/HAL%20Implementation/HAL/Reusables) folder and place it parallel to the `Instruments` folder, preserving its contents.
   2. Add the copied `Get Pin Information.lvlib` to your LabVIEW project.
   3. In `Get Instrument Path.vi`, replace the term **DMM_Models** in the base path with the actual name of the directory that contains your instrument child classes. This VI takes the `instrument_type_id` as input and returns the path to the corresponding child class directory.

      ![Get Instrument Path](<./HAL Images/Get Instrument Path.png>)

   > **Note:**  
   > The expected folder structure for any HAL-based measurement plug-in should have the **Reusables** folder parallel to the **Instruments** folder, which contains the instrument base and child classes.

9. **Copy Utility VIs**  
   1. Copy the [Utility](https://github.com/NI-Measurement-Plug-Ins/abstraction-layer-labview/tree/main/Source/HAL%20Implementation/HAL/Instruments/DMM_Base/Utility) folder and place it parallel to the instrument base class, preserving its contents.
   2. Add the copied VIs from the `Utility` folder to your instrument base class under a virtual folder of the same name.
   3. Open each copied VI and replace any missing constants or controls of the class object with your base class object.

10. **Create instrument child classes**  
    1. In `<instrument type>_Models` folder, create child classes that inherit from the instrument base class, as shown in the class hierarchy image below. Refer to this [example](https://github.com/NI-Measurement-Plug-Ins/abstraction-layer-labview/tree/main/Source/HAL%20Implementation) for more information.
    2. **The name of each child class and its directory must match the instrument type ID configured for that instrument model in the pin map file.**
    3. Below are the standard directory names followed for NI instruments:

       | Instrument type       | Directory name    |
       |-----------------------|-------------------|
       | NI-DCPower            | niDCPower         |
       | NI-DMM                | niDMM             |
       | NI-Digital Pattern    | niDigitalPattern  |
       | NI-SCOPE              | niSCOPE           |
       | NI-FGEN               | niFGEN            |
       | NI-DAQmx              | niDAQmx           |
       | NI-SWITCH             | niRelayDriver     |

    ![HAL Class Hierarchy](<./HAL Images/HAL Class Hierarchy.png>)

11. **Override methods in child classes**  
    1. For NI instruments, override only the measurement methods of the instrument base class in the child class.
    2. For custom instruments, override both the session and measurement methods in the child class. For example, session methods for a Keysight DMM include:
       - ***Initialize MeasurementLink Session.vi*** - Creates a new session using the session initialization parameters. If the session represents a remote session, initialize and close session behavior determines whether creating the local session creates a new session on the server or attaches to an existing session on the server.

          ![Initialize MeasurementLink Session](<HAL Images/KeysightDmm Initialize MeasurementLink Session.png>)

       - ***Get Provided Interface and Service Class.vi*** - Returns the provided interface and service class that will be used to query the measurement plug-ins discovery service for the address and port of the instrument's gRPC server. Do not implement/override this VI if the instrument does not use a gRPC server.

          ![Get Provided Interface and Service Class](<HAL Images/KeysightDmm Get Provided Interface and Service Class.png>)

       - ***Close MeasurementLink Session.vi*** - Closes the local session. If the session represents a remote session, initialize and close session behavior determines whether closing the local session closes the server session or detaches from the server session.

          ![Close MeasurementLink Session](<HAL Images/KeysightDmm Close MeasurementLink Session.png>)

12. **Define plug-in I/O and update logic**  
    1. Define the inputs and outputs in the measurement plug-in and update `Get Type Specializations.vi` to populate pin information from the pin map file.
    2. Update the measurement logic with the following APIs:
       - **Reserve Sessions.vi** – Polymorphic VI for reserving single or multiple pins.
       - **Get Pin Object.vi** – Polymorphic VI for retrieving instrument base class objects using `pin or relay name(s)` and `session reservation` object.
       - **Measurement APIs** – Use the instrument base class methods (`Initialize`, `Configure`, `Measure`) to perform measurements.
       - **Close Sessions.vi** and **Unreserve Sessions.vi** – Close and unreserve all instrument sessions created with the session reservation object.

       ![Measurement Logic](<HAL Images/Measurement Logic.png>)

## Migrate the existing instrument class to Measurement Plug-In

1. Create a measurement plug-in by following the steps mentioned in the [Developing a measurement plug-in with LabVIEW](https://github.com/ni/measurement-plugin-labview?tab=readme-ov-file#developing-a-labview-measurement) guide.
2. Add your existing instrument class to the LabVIEW project that contains the created measurement plug-in.
3. Update your instrument base class to inherit the **ISession Factory** interface, located at `<vi.lib>\Plug-In SDKs\Sessions\Instrument\ISession Factory\ISession Factory.lvclass`.
4. Follow steps 4-12 of [Steps to create new HAL based measurement](#steps-to-create-new-hal-based-measurement) for migrating your existing instrument class-based HAL to measurement plug-in compatible, class-based HAL.
