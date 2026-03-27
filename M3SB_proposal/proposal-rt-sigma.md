
---
title: "Real-time Signal Interference GNSS-network Monitoring and Analysis"
---



# Proposal summary `rt-sigma`


## **Context and Problem Statement**
Global Navigation Satellite Systems (GNSS), such as Galileo and GPS, are fundamental to both national security and the continuous operation of critical infrastructure. From synchronized power grids and financial transaction timing to the precise navigation of autonomous systems and military assets, the reliance on these signals is absolute. However, GNSS signals are inherently weak when they reach the Earth's surface, making them highly susceptible to both intentional and unintentional Radio Frequency Interference (RFI).

Traditional monitoring often relies on post-processing archival data, which creates a "detection gap"—a window of time where interference can cause significant operational disruption before it is identified. Project `rt-sigma` addresses this vulnerability by pivoting to a high-fidelity, real-time monitoring architecture.

## **Project Objectives**
The primary goal of project `rt-sigma` is to develop a fully automated, (near) real-time detection and classification system for GNSS interference across a network of reference stations. The proposal focuses on three technological pillars:

* **Continuous Stream Processing:** Utilizing live data ingestion from GNSS receivers to analyze signal health (SNR/CNR) and positional stability the moment the data is received. Due to the large number of CORS (Continuous Operating Reference Stations) in Belgium a *Clustered Edge Architecture* based on regional clusters of stations will be implemented to reduce latency and bandwidth requirements.
* **Low-Latency Alerting:** Implementing a high-speed data backbone to ensure that anomalies are detected and reported to relevant authorities within seconds, enabling immediate tactical or civil response.
* **Automated RF Forensics:** Deploying a "triggered capture" mechanism that preserves raw radio frequency (IQ) samples during an active interference event, providing the necessary evidence for signal characterization and source localization.

## **Proposed Methodology**
The project utilizes a modular software-defined architecture. A dedicated utility converts live receiver streams into a standardized data format. This data is broadcast via a high-performance message broker (such as NATS or Redis) to a suite of analytical "consumers" deployed in Regional Processing Hubs. Each hub acts as a "Cluster Controller" for approximately five stations. The hub processes the incoming streams and detection algorithms analyze these streams for SNR degradation or coordinate instability.

By processing data in regional clusters, `rt-sigma` minimizes the bandwidth load on the central backbone while maintaining the "Black Box" recording capability (IQ capture) close to the data source. 

The consumers at each cluster apply statistical algorithms to monitor:


1.  **Signal Quality:** Detecting narrowband or broadband jamming through sudden drops in Signal-to-Noise ratios.
2.  **Coordinate Integrity:** Identifying spoofing attempts by detecting instantaneous, non-physical shifts in the known fixed positions of the reference stations.

## **Impact and Application**
This clustered approach ensures a robust, cost-effective deployment. The result is a national **"Situational Awareness Map"** where regional hubs report localized threats to a central command center with immediate, actionable intelligence on the state of the Belgian L-band spectrum. The project results provide  a state-of-the-art situational awareness dashboard, transforming GNSS monitoring from a retrospective analysis tool into a proactive defense asset.

# Work Packages `rt-sigma`

## WP 1: Literature Study and Requirements Analysis


**Goal:** Define the technical framework for real-time monitoring and review existing streaming GNSS interference methodologies.

* **Task 1.1: Review of Real-Time Detection Methods:** Study existing "Detect-on-Arrival" systems and stream-processing architectures (e.g., NATS, Kafka, RabbitMQ, or Redis-based pipelines).
* **Task 1.2: Technical Requirements Definition:** Specify the requirements for the GNSS parser output (CSV schema) and the latency thresholds required for alerting.
* **Task 1.3: Analysis of Multi-Constellation Streams:** Define how GPS, Galileo, GLONASS, and BeiDou signals will be synchronized within the live data bus.

## WP 2: Real-Time Data Ingestion and Infrastructure


**Goal:** To establish a robust, low-latency pipeline that replaces static file processing with live GNSS data streams in a regional processing hub.

- **Task 2.1: Regional Hub Setup** 
    - Develop the Cluster Controller software environment capable of multiplexing GNSS data streams for multiple incoming station streams.
* **Task 2.2: Message Broker Implementation (Data Bus)**
    * Set up the communication backbone. While a **REDIS** setup  is functional, this task will evaluate and deploy a production-grade broker like **NATS**, **Kafka** or **RabbitMQ** for high-throughput GNSS data telemetry.
* **Task 2.3: GNSS Stream Standardization**
    * Define the real-time schema for PVT (Position, Velocity, Time), DOP, and RTCM MSM5/7 messages to ensure compatibility between the parser and downstream detection repos.
* **Task 2.4: Quality of Service (QoS) Monitoring**
    * Develop heartbeat and latency tracking to ensure the "near" real-time delay remains under a threshold.

## WP 3: Real-Time Interference Detection Algorithms


**Goal:** To adapt SNR and position-based detection logic to work on streaming data buffers rather than archival files.

* **Task 3.1: Sliding-Window SNR Analysis**
    * Develop algorithms to monitor Carrier-to-Noise density (C/N0) in real-time. Implement adaptive thresholds to distinguish between natural signal fading and intentional jamming.
* **Task 3.2: Live Coordinate Variation Monitoring**
    * Utilize the live PVT feed to detect instantaneous "jumps" or drifts in the known position of core stations, serving as a primary indicator for spoofing attacks.
* **Task 3.3: Multi-Station Stream Correlation**
    * Cross-reference real-time GNSS data from multiple Belgian stations to localize the source of interference as it moves or expands.

## WP 4: GNSS Data Consumer and Detection Engine Integration


**Goal:** To finalize the processing logic into a headless detection engine.

* **Task 4.1: Automated Detection Logic**
    * Create an automated decision engine that flags anomalies without human intervention.
* **Task 4.2: Real-Time Data Persistence**
    * Implement a "Ring Buffer" storage system where live data is kept in memory for analysis but only written to long-term storage when an interference event is triggered.
    - Determine whether the I/Q data ring is at the CORS reference station or at the regional hub.

## WP 5: Real-Time Position-Based Detection


**Goal:** Detect spoofing and jamming by monitoring instantaneous coordinate variations in the live stream.

* **Task 5.1: Live Coordinate Stability Monitoring**
    * Adapt wavelet analysis and STFT techniques to process coordinate feeds from GNSS parser in real-time.
* **Task 5.2: Real-Time Spoofing Indicators**
    * Identify "jumps" or rapid drifts in station coordinates that signify a spoofing takeover.
* **Task 5.3: Edge Processing Validation**
    * Test if position-based detection can be performed locally at the station level to reduce central bandwidth.

## WP 6: Stream Classification and Event Management


**Goal:** Classify the type of interference (Jamming vs. Spoofing) as it occurs.

- **Task 6.1: Local CORS or Regional Cluster IQ Buffer**
    - Implement a multi-channel circular buffer at either the local reference station or at  the Regional Hub to store raw IQ samples for all associated stations, triggered by a detection event at any single station in the cluster.
* **Task 6.2: Real-Time Interference Classification**
    * Apply machine learning or statistical signatures to the live GNSS data stream to categorize threats (e.g., Narrowband, Broadband, or Meaconing).
* **Task 6.3: Event Correlation Engine**
    * Automatically link anomalies across multiple stations to confirm the geographic extent of an attack.

## WP 7: Triggered IQ Raw Data Capture and Forensics


**Goal:** To implement a high-fidelity "Black Box" recorder that captures raw RF (IQ) samples upon detection of an interference event for post-processing.

* **Task 7.1: Circular IQ Buffer Implementation**
    * Develop a background process on the producer side hat maintains a continuous N-minute circular buffer of raw IQ samples in RAM or high-speed NVMe storage.
* **Task 7.2: Trigger-Based Event Logging**
    * Implement a "Snapshot" logic: When the real-time detection engine  flags an event, the system automatically freezes the circular buffer and saves the previous $x$ minutes (pre-trigger) and the subsequent $y$ minutes (post-trigger) to a permanent log file.
* **Task 7.3: IQ Metadata Tagging**
    * Ensure each IQ dump is cryptographically signed and timestamped with the specific detection criteria (e.g., "Triggered by 15dB SNR drop") to maintain a chain of evidence for regulatory authorities.
* **Task 7.4: Post-Processing Tool Set**
    * Develop a basic utility to export these triggered IQ logs into standard formats (e.g., SigMF) for analysis in professional tools like MATLAB or GNSS-SDR.

## WP 8: Visualization, Alerting, and Reporting


**Goal:** To provide the end-user  with immediate situational awareness.

* **Task 8.1: Real-Time Dashboard**
    * Create a dashboard that visualizes the network health by 'Cluster,' allowing users to drill down from a regional OK/NOK status to individual station metrics.
* **Task 8.2: Immediate Alerting System**
    * Integrate an automated notification system (API, Email, or SMS) that triggers the moment the detection engine identifies a high-probability interference event.

## WP X: Project Coordination and Management


**Goal:** Ensure administrative coherence and regular reporting to the RMA and NGI.


* **Task X.1: Project Planning & Tracking**
    * Manage the 48-month timeline and milestone progress.
* **Task X.2: Stakeholder Communication**
    * Conduct quarterly meetings and yearly briefing.
* **Task X.3: Reporting**
    * Delivery of yearly progress reports and the final project evaluation.

