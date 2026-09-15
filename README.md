# AI-Based AR Guidance and VR Training Platform for Dental Surgery

## Project Overview

This project is a graduation project that combines **Artificial Intelligence, Augmented Reality (AR), and Virtual Reality (VR)** to support dental surgery and dental education.

The project focuses on two main problems.

First, during dental surgery, dentists often work with CBCT scans on a computer screen and then have to mentally translate the 2D information into the patient's actual 3D anatomy. This can increase cognitive load and create a gap between preoperative planning and what happens during the procedure.

Second, dental students need a safe environment where they can practice surgical procedures without putting real patients at risk.

To address these problems, the system is divided into two main components:

* **AR Surgical Guidance:** Segmented CBCT information is aligned and displayed directly over the patient's anatomy during surgery.
* **VR Dental Training:** Students can practice dental procedures in a virtual environment and receive feedback on their performance.

The four VR training scenarios are:

1. Guided Drilling
2. Retained Root Removal
3. Tumor Removal
4. Impacted Tooth Extraction

For the AR system, the project uses the **Viture AR headset**, a U-Net model for segmentation, and a YOLO-based model for registration. The AI models are deployed through Hugging Face Spaces using Gradio APIs.

For VR training, the system uses an **HTC Vive headset** and a Unity application. The dental meshes are generated using the **Marching Cubes algorithm**, allowing the models to be used for interactive drilling and simulation.

The AI models are developed using **TensorFlow and Keras**, with data coming mainly from the ToothFairy3 2025 challenge and supplemented with datasets from Kaggle and Roboflow.

## System Architecture

The project consists of two connected workflows:

```text
                    Dental Surgery Platform
                            |
             ┌──────────────┴──────────────┐
             |                             |
             ↓                             ↓
       AR Surgical System             VR Training System
             |                             |
        CBCT Scan                     Scenario Selection
             ↓                             ↓
       U-Net Segmentation            Mesh Generation
             ↓                             ↓
       YOLO Registration             Dental Simulation
             ↓                             ↓
       AR Overlay                    Performance Tracking
             ↓                             ↓
     Intraoperative Guidance          Training Report
```

The AR side is mainly focused on **surgical guidance and safety**, while the VR side focuses on **training and performance evaluation**.

---

# AR Surgical Guidance

The AR workflow starts when the dentist loads a CBCT scan and ends when the procedure is completed and the session is saved.

## 1. Load CBCT Scan

The dentist selects a DICOM file through the application.

Before processing begins, the system checks whether the file is valid. If the file is invalid, the dentist is asked to select another file.

## 2. Preoperative Planning

The dentist can view and manipulate the 3D scan before the procedure.

Important parameters such as:

* Entry point
* Target depth
* Angle limits

can be defined during planning.

The measurements are updated while the dentist works, and the final plan is saved with the session.

## 3. AI Segmentation

The AR application sends the CBCT volume to the **U-Net segmentation model** through a Hugging Face API.

The request is handled asynchronously, so the application can display a loading state while waiting for the model.

If the API fails or returns an invalid mesh, the dentist can retry the request. After three unsuccessful attempts, the session can be flagged for manual follow-up.

## 4. Registration and Alignment

After segmentation, the generated 3D mesh needs to be aligned with the patient's real anatomy.

The **YOLO registration model** uses the AR headset camera to perform this alignment in real time.

The target registration error is:

```text
Registration Error < 1 mm
```

If the error is too high, the system can retry the registration or switch to marker-based registration.

## 5. Activate AR Overlay

Once the registration is accepted, the segmented anatomy is displayed over the patient's mouth.

This allows the dentist to view the CBCT information together with the real anatomy instead of constantly switching between the patient and an external screen.

## 6. Intraoperative Monitoring

During the procedure, the system continuously monitors:

* Drill angle
* Drill depth
* Safety thresholds

A visual color indicator provides feedback, while audio and visual alarms are triggered when a safety threshold is exceeded.

The alarm is cleared only after the dentist corrects the deviation.

## 7. Session Logging

When the procedure is finished, the system stores the session information.

The log contains information such as:

* Drill angle
* Depth
* Deviations
* Number of alarms
* Other procedure events

Once the session is closed, the recorded data becomes immutable.

---

# VR Dental Training

The VR component provides students with a safe environment to practice dental procedures.

## 1. Launch the VR Environment

The trainee starts the Unity application using the **HTC Vive headset**.

A scenario selection menu is displayed.

## 2. Select a Training Scenario

The trainee chooses one of four available procedures:

```text
Guided Drilling
Retained Root Removal
Tumor Removal
Impacted Tooth Extraction
```

Each scenario has its own objectives and evaluation criteria.

## 3. Load the Dental Model

If a patient CBCT scan is available, it can be loaded into the system.

If there is no patient scan, the system uses a synthetic jaw model.

Sub-region voxelization is applied during loading to prepare the model for real-time deformation.

The **Marching Cubes algorithm** is then used to generate the mesh.

If mesh generation takes too long, the system automatically reduces the voxel resolution in the affected region and retries.

## 4. Perform the Simulation

The trainee performs the selected procedure using the VR controllers.

The system tracks different aspects of the procedure, including:

* Entry point
* Drill angle
* Drill depth
* Contact with critical anatomy

If the trainee gets dangerously close to important structures, such as a nerve, or removes too much healthy tissue, the system displays a warning.

The trainee can correct the mistake or restart the scenario.

## 5. Complete the Training Session

When the procedure is successfully completed, a performance report is generated.

If the trainee exits the simulation before completing it, the session is marked as **aborted**, and no performance report is generated.

This prevents incomplete attempts from affecting the student's performance history.

## 6. Review Performance

For completed sessions, the trainee can review metrics such as:

* Entry point deviation
* Angular error
* Depth error
* Total procedure time

These values can be compared with the benchmarks identified in the project's literature review.

---

# System Actors

The system contains both human users and external digital services.

| Actor             | Role                                                                |
| ----------------- | ------------------------------------------------------------------- |
| Dentist / Surgeon | Uses the AR system, plans procedures, and manages surgical guidance |
| Trainee / Student | Performs VR simulations and reviews performance                     |
| Supervisor        | Manages training scenarios and monitors student progress            |
| U-Net Model       | Segments CBCT volumes into 3D anatomical meshes                     |
| YOLO Model        | Performs real-time anatomical registration                          |
| CBCT File System  | Provides the original DICOM scan data                               |

The AI models are treated as external actors so that the main application remains separated from the cloud-based AI infrastructure.

---

# Main Use Cases

## AR Subsystem

The clinical AR subsystem is designed around four main activities:

### Planning

The dentist loads a DICOM scan, identifies the required entry point, and defines depth and angle safety limits.

### AI Processing

The system sends the scan to the AI models for segmentation and registration.

### Live Guidance

The aligned anatomy is displayed through AR while drill movement is monitored in real time.

### Audit and Logging

The system records deviations, alarms, and other events for later review.

## VR Subsystem

The educational VR subsystem focuses on:

### Simulation Selection

Students select a procedure such as tumor removal or impacted tooth extraction.

### Dynamic Mesh Generation

The system generates interactive meshes using Marching Cubes for drilling and deformation.

### Feedback

The trainee receives real-time warnings and a detailed report after completing the procedure.

### Administration

Supervisors can assign training scenarios and monitor student progress over time.

---

# Data Model

The system is designed with **clinical traceability** in mind.

The goal is to make it possible to connect a surgical or training action back to the original patient scan and the AI model version that produced the corresponding 3D model.

## Main Entities

### Patient

Stores information that identifies the patient and connects the patient's medical data.

### CBCT Scan

Contains information about the original 3D X-ray, including:

* Scan ID
* Date
* Dimensions

### Segmented Mesh

Represents the AI-generated anatomical model.

Important attributes include:

* Mesh ID
* OBJ file
* Dice score

### Voxel Mesh

Represents the drillable model used in VR.

Its attributes include:

* Voxel ID
* Resolution

### AR Session

Stores information about an actual surgical procedure, including the dentist and recorded alarms.

### Drill Log

Stores detailed information about drill movement, such as angle and depth.

### VR Session

Represents an individual student training attempt.

### VR Report

Contains the final performance measurements, including error and time.

---

# Data Relationships

The main data flow can be summarized as:

```text
CBCT Scan
    |
    ├──> Segmented Mesh
    |
    └──> Voxel Mesh
             |
             ↓
        VR Training

Segmented Mesh
    |
    ↓
Registration
    |
    ↓
AR Session
    |
    ↓
Drill Logs
```

One scan can be used to create the models needed for both AR and VR.

Registration can also be repeated without running segmentation again, because the same generated mesh can simply be realigned.

A single surgical session can generate many drill-log records because the system tracks movement continuously.

---

# State Management

## AR Session

The AR workflow follows a controlled sequence:

```text
IDLE
  ↓
CBCT Loaded
  ↓
Planning
  ↓
Segmentation
  ↓
Registration
  ↓
Overlay Active
  ↓
Monitoring
  ↓
COMPLETED
```

The system does not allow unsafe steps to be skipped.

For example, the AR overlay cannot be activated before successful registration.

The workflow also includes failure states for segmentation and registration so that errors are handled explicitly.

After completion, the session data becomes immutable.

## VR Session

The VR workflow follows:

```text
Scenario Selection
        ↓
Mesh Generation
        ↓
Simulation
        ↓
Completed / Aborted
        ↓
Performance Report
```

If mesh generation takes too long, the system optimizes the affected region.

Critical errors during the simulation trigger warnings.

Only completed sessions generate performance reports.

---

# Business Rules

## Validation

Before the system proceeds, it checks:

* DICOM validity
* Mesh validity
* Registration error
* AI segmentation quality
* Alarm response time
* Mesh generation latency

The main AI
