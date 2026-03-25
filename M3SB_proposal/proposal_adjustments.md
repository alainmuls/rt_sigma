# **Proposal Adjustment: SIGMA Real-Time Edition**

Below is the adjusted proposal for project **SIGMA**, updated to focus on **(near) real-time interference detection** using a live data pipeline rather than offline RINEX processing.

#### **1. Core Concept Shift**

[cite_start]The original proposal relied on **RINEX files** (archival data) stored at the NGI[cite: 383, 390]. The updated project SIGMA will transition to a **streaming architecture**. It will leverage the `gnss_parser` utility to ingest live data from GNSS receivers or RTCM streams, generating real-time CSV data for immediate analysis.

#### **2. Technical Architecture & Connection Strategy**

While your current script uses **REDIS**, for a robust national monitoring system, a **Message Broker** approach is recommended.

* **Proposed Alternative: NATS (or RabbitMQ)**
  * **Why:** Unlike Redis, which is primarily an in-memory data store, **NATS** is a cloud-native messaging system designed for high-throughput, low-latency telemetry. It is "fire-and-forget" by default but supports **JetStream** for persistence if a detection algorithm needs to "catch up" on missed data.
  * **Benefit for SIGMA:** It simplifies the connection between the `gnss_parser` (producer) and the `csv_consumer` (detectors) across different network nodes without managing complex database states.

---

### **3. Adjusted Project Objectives (Section 2.4)**

| Objective | Specific | Time-bound |
| :--- | :--- | :--- |
| **O1: Real-time Data Ingestion** | Replace RINEX batching with a live pipeline using `gnss_parser` and a message broker (e.g., NATS/Redis). | Month 9 |
| **O2: Stream-based Detection** | Adapt SNR-based algorithms to work on sliding windows of live CSV data rather than static files. | Month 18 |
| **O3: Dynamic Coordinate Monitoring** | [cite_start]Implement real-time tracking of station coordinate variations to detect immediate spoofing events[cite: 413, 414]. | Month 24 |
| **O6: Real-time Dashboard & Alerts** | [cite_start]Develop a live Grafana/Web dashboard that triggers alerts (API/Push) within seconds of detection[cite: 461]. | Month 36 |

---

### **4. Updated Work Packages (Section 4.1)**

**WP2: Real-time Data Infrastructure (Months 3–15)**

* **Task 2.1: gnss_parser Integration:** Deploy the `gnss_parser` utility to interface with Belgian CORS stations via RTCM streams.
* **Task 2.2: Message Broker Deployment:** Setup a REDIS or NATS cluster to handle the CSV stream between the parser and the analysis modules.
* **Task 2.3: Stream Validation:** Ensure data integrity and latency are below 1 second for 99% of messages.

**WP3: Live Signal-Based Detection (Months 10–24)**

* [cite_start]**Task 3.1: Adaptive Thresholding:** Instead of historical RINEX baselines, implement "moving average" signal strength (SNR) thresholds to detect sudden interference[cite: 395, 396].
* [cite_start]**Task 3.2: Real-time C/N0 Monitoring:** Process live CSV feeds to identify narrowband or broadband jamming signatures as they happen[cite: 402, 403].

**WP6: Visualization and Alerting (Months 30–48)**

* **Task 6.1: SIGMA Live Dashboard:** Build a dashboard (e.g., using Streamlit or Grafana) that visualizes the current status of the Belgian network.
* [cite_start]**Task 6.2: Automated Alerting API:** Create a REST API or Webhook system to notify the BIPT or Defence authorities immediately upon high-confidence interference detection[cite: 347, 461].

---

### **5. Innovative Character (Section 2.2 Updated)**

* [cite_start]**Zero-Latency Monitoring:** Unlike the Swedish SWEPOS or previous Polish systems that often rely on batch-processed RINEX[cite: 383, 387], SIGMA moves to a "Detect-on-Arrival" model.
* [cite_start]**Edge Compatibility:** By using `gnss_parser` to generate lightweight CSV data, the system can eventually run on low-cost COTS hardware at the "edge" of the network, reducing the need for massive central data transfers[cite: 431, 460].

---

### **Summary of Key Changes for the Proposal**

1. **Terminology:** Change "RINEX-based analysis" to **"Real-time Stream Analysis."**
2. **Input Source:** Replace "NGI Archive" with **"Live RTCM/NTRIP Streams."**
3. **Deliverables:** Include the **Real-time CSV Consumer script** and the **Message Broker configuration** as core technical outputs of the project.

---

Based on your transition from offline RINEX processing to a real-time streaming architecture using `gnss_parser` and a message broker, here are the adjusted Work Packages (WPs).

These revisions focus on the live data pipeline, the ingestion of RTCM/Serial data, and the shift toward "Detect-on-Arrival" algorithms.

---

# Adjusted Work Packages: Project SIGMA (Real-Time)

## WP 1: Literature Study and Requirements Analysis


**Goal:** Define the technical framework for real-time monitoring and review existing streaming GNSS interference methodologies.

* **Task 1.1: Review of Real-Time Detection Methods:** Study existing "Detect-on-Arrival" systems and stream-processing architectures (e.g., NATS, Kafka, RabbitMQ, or Redis-based pipelines).
* **Task 1.2: Technical Requirements Definition:** Specify the requirements for the `gnss_parser` output (CSV schema) and the latency thresholds required for alerting.
* **Task 1.3: Analysis of Multi-Constellation Streams:** Define how GPS, Galileo, GLONASS, and BeiDou signals will be synchronized within the live data bus.

## WP 2: Real-Time Data Ingestion and Infrastructure


**Goal:** To establish a robust, low-latency pipeline that replaces static file processing with live GNSS data streams.

* **Task 2.1: Integration of `gnss_parser`**
    * Deploy and configure the `gnss_parser` utility to interface directly with GNSS receivers (via Serial/USB) and reference station streams (via RTCM/NTRIP).
* **Task 2.2: Message Broker Implementation (Data Bus)**
    * Set up the communication backbone. While a **REDIS** setup (as used in `csv_consumer.py`) is functional, this task will evaluate and deploy a production-grade broker like **NATS**, **Kafka** or **RabbitMQ** for high-throughput CSV telemetry.
* **Task 2.3: Stream-to-CSV Standardization**
    * Define the real-time schema for PVT (Position, Velocity, Time), DOP, and RTCM MSM7 messages to ensure compatibility between the parser and downstream detection repos.
* **Task 2.4: Quality of Service (QoS) Monitoring**
    * Develop heartbeat and latency tracking to ensure the "near" real-time delay remains under 500ms.

## WP 3: Real-Time Interference Detection Algorithms


**Goal:** To adapt SNR and position-based detection logic to work on streaming data buffers rather than archival files.

* **Task 3.1: Sliding-Window SNR Analysis**
    * Develop algorithms to monitor Carrier-to-Noise density (C/N0) in real-time. Implement adaptive thresholding to distinguish between natural signal fading and intentional jamming.
* **Task 3.2: Live Coordinate Variation Monitoring**
    * Utilize the live PVT feed to detect instantaneous "jumps" or drifts in the known position of core stations, serving as a primary indicator for spoofing attacks.
* **Task 3.3: Multi-Station Stream Correlation**
    * Cross-reference real-time CSV data from multiple Belgian stations to localize the source of interference as it moves or expands.

## WP 4: CSV Consumer and Detection Engine Integration


**Goal:** To finalize the `csv_consumer.py` logic into a headless detection engine.

* **Task 4.1: Automated Detection Logic**
    * Transition the `csv_consumer` from a display utility to an automated decision engine that flags anomalies without human intervention.
* **Task 4.2: Real-Time Data Persistence**
    * Implement a "Ring Buffer" storage system where live data is kept in memory for analysis but only written to long-term storage when an interference event is triggered.

## WP 4: Real-Time Position-Based Detection


**Goal:** Detect spoofing and jamming by monitoring instantaneous coordinate variations in the live stream.

* **Task 4.1: Live Coordinate Stability Monitoring**
    * Adapt wavelet analysis and STFT techniques to process coordinate feeds from `gnss_parser` in real-time.
* **Task 4.2: Real-Time Spoofing Indicators**
    * Identify "jumps" or rapid drifts in station coordinates that signify a spoofing takeover.
* **Task 4.3: Edge Processing Validation**
    * Test if position-based detection can be performed locally at the station level to reduce central bandwidth.

## WP 5: Stream Classification and Event Management


**Goal:** Classify the type of interference (Jamming vs. Spoofing) as it occurs.

* **Task 5.1: Real-Time Interference Classification**
    * Apply machine learning or statistical signatures to the live CSV stream to categorize threats (e.g., Narrowband, Broadband, or Meaconing).
* **Task 5.2: Event Correlation Engine**
    * Automatically link anomalies across multiple NGI stations to confirm the geographic extent of an attack.

## WP 6: Triggered IQ Raw Data Capture and Forensics


**Goal:** To implement a high-fidelity "Black Box" recorder that captures raw RF (IQ) samples upon detection of an interference event for deep post-processing.

* **Task 6.1: Circular IQ Buffer Implementation**
    * Develop a background process on the producer side (connected to the SDR/Receiver) that maintains a continuous N-minute circular buffer of raw IQ samples in RAM or high-speed NVMe storage.
* **Task 6.2: Trigger-Based Event Logging**
    * Implement a "Snapshot" logic: When the real-time detection engine (from WP3/WP4) flags an event, the system automatically freezes the circular buffer and saves the previous $x$ minutes (pre-trigger) and the subsequent $y$ minutes (post-trigger) to a permanent log file.
* **Task 6.3: IQ Metadata Tagging**
    * Ensure each IQ dump is cryptographically signed and timestamped with the specific detection criteria (e.g., "Triggered by 15dB SNR drop") to maintain a chain of evidence for regulatory authorities (BIPT).
* **Task 6.4: Post-Processing Toolset**
    * Develop a basic utility to export these triggered IQ logs into standard formats (e.g., SigMF) for analysis in professional tools like MATLAB or GNSS-SDR.

## WP 7: Visualization, Alerting, and Reporting


**Goal:** To provide the end-user (Defense/NGI) with immediate situational awareness.

* **Task 7.1: SIGMA Real-Time Dashboard**
    * Develop a web-based dashboard (e.g., Grafana or Streamlit) that visualizes live SNR levels and station health across the Belgian network.
* **Task 7.2: Immediate Alerting System**
    * Integrate an automated notification system (API, Email, or SMS) that triggers the moment the detection engine identifies a high-probability interference event.

## WP X: Project Coordination and Management


**Goal:** Ensure administrative coherence and regular reporting to the RMA and NGI.


* **Task X.1: Project Planning & Tracking**
    * Manage the 48-month timeline and milestone progress.
* **Task X.2: Stakeholder Communication**
    * Conduct quarterly meetings between RMA and NGI and yearly briefings for the RHID and BIPT.
* **Task X.3: Reporting**
    * Delivery of yearly progress reports and the final project evaluation.

---

### **Technical Recommendation: Why NATS over REDIS?**
In your `csv_consumer.py`, you are currently watching the file system for CSV changes. For a professional project like SIGMA:
1.  **Latency:** Writing to a disk (CSV) and then reading it with a `watchdog` adds I/O overhead.
2.  **Scalability:** If you move to **NATS (Message Broker)**, the `gnss_parser` can publish data directly to a "subject." Your consumer simply subscribes to it in memory.
3.  **Resilience:** NATS is designed for "Cloud Native" environments and handles re-connections and high-frequency GNSS telemetry more gracefully than file-system polling.
