# Mini Conveyor Control & Sorting System (CODESYS V3.5)

A professional industrial automation project demonstrating a complete PLC and Web-Based HMI implementation for a conveyor belt control system. Developed using CODESYS V3.5, this system integrates robust control logic, interlocking safety features, and a modern user interface designed following ISA-101 HMI visualization standards.

---

##  Tech Stack & Tools
* **PLC Development Environment:** CODESYS V3.5 SP22
* **Programming Language:** IEC 61131-3 Ladder Diagram (LD)
* **PLC Runtime Simulation:** CODESYS Control Win V3 x64
* **HMI Solution:** CODESYS WebVisu (HTML5/SVG Web Server)

---

##  Key Features & Control Logic

The control program manages a standard industrial mini-conveyor with the following core functionalities:

1. **Dual-Mode Operation:** * **Auto Mode:** Governed by a dedicated Start/Stop latching logic with self-retaining contact design.
   * **Manual Mode:** Provides physical control using Forward (FW) and Reverse (REV) Jog pushbuttons.
2. **Safety Interlocking:** High-reliability protection implemented in the Ladder Diagram prevents simultaneous FW and REV motor excitation, eliminating short-circuit risks.
3. **Box Jamming Detection (Safety Timer):** Integrates a **TON Timer (4 seconds)** that continuously monitors the photo-electric sensor. If a package blocks the sensor for too long, the system automatically shuts down to prevent motor burnout.
4. **Automated Batch Counting:** Uses a Up-Counter (**CTU**) to track sorted products. Upon reaching the batch limit of **5 items**, the conveyor safely stops and signals the operator.

---

##  Web-Based HMI Interface

The HMI is rendered via standard web browsers using CODESYS WebVisu, allowing remote terminal operations. 

* **Clean Layout:** Designed with a realistic light-gray control panel texture, separating control zones (Auto/Manual) from monitoring zones.
* **Industrial Indicators:** Features a standard 3-tier Tower Lamp (Red for Batch Complete/Alarm, Yellow for Standby/Ready, Green for Auto Running).
* **Live Counter Display:** Dynamically displays real-time counting status (`COUNT : %d`) synced directly with the PLC backend variables.

###  Interface Preview
*(Optional: Delete this line and drag/drop your HMI screenshot here in GitHub to show your work!)*

---

##  How to Run and Simulate

1. Open the project file `Mini_Conveyor_Sorting_System.project` in CODESYS V3.5.
2. Go to the **Online** menu and check **Simulation**.
3. Click **Login** and download the program to the virtual PLC controller.
4. Press **F5** (or go to Debug -> **Start**) to run the PLC.
5. Open Google Chrome or Microsoft Edge and navigate to:
   ```text
   http://localhost:8080/webvisu.htm

   ## 📊 System Operational Flowchart

The sequential logic of the PLC program separates into two main operational paths based on the Selector Switch:

```mermaid
graph LR
    %% Main Control Split
    Start([Start]) --> Mode{Selector Switch?}

    %% MANUAL MODE PATH
    Mode -- Manual Mode --> ManStandby[Standby / Yellow ON]
    ManStandby --> Button{Jog Button?}
    Button -- Press FW --> MotorFW[Motor FORWARD]
    Button -- Press REV --> MotorREV[Motor REVERSE]
    Button -- Release --> MotorStop[Motor STOP]
    MotorStop --> ManStandby

    %% AUTO MODE PATH
    Mode -- Auto Mode --> AutoReady[Ready / Yellow ON]
    AutoReady --> StartBtn{Press START?}
    StartBtn -- Yes --> AutoRun[Conveyor RUN / Green ON]
    StartBtn -- No --> AutoReady
    
    AutoRun --> Sensor{Sensor Status?}
    Sensor -- Blocked > 4s --> Alarm[Jam Alarm / Red ON & STOP]
    Sensor -- Normal Pass --> Count[COUNT = COUNT + 1]
    
    Count --> Target{COUNT = 5?}
    Target -- No --> AutoRun
    Target -- Yes --> Complete[Batch Complete / Red ON & STOP]
    
    %% Reset Action
    Alarm & Complete --> Reset{Press RESET?}
    Reset -- Yes --> Clear[Clear COUNT & Alarm]
    Clear --> AutoReady
    Reset -- No --> Alarm & Complete

    - **Emergency Stop (E-STOP) Integration:** Features a dedicated safety cutoff. Pressing the E-STOP instantly de-energizes the conveyor motor in both Auto and Manual modes, requiring a manual Reset to restart operations.