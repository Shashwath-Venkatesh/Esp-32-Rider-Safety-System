## Rider Safety System

## Overview
This project is an affordable, comprehensive, and energy-efficient rider safety system built using an ESP32 microcontroller. Unlike traditional systems that only focus on impact detection, this system utilizes distinct algorithmic pathways to detect both high-impact crashes and medical emergencies (such as fainting or incapacitation due to heatstroke). 

Because it relies on ubiquitous GSM technology rather than internet connectivity, it functions reliably in remote off-road environments.

## Key Features
*   **Dual Detection Pathways:** Separate logical flows for high-g crash impacts and low-activity medical emergencies.
*   **Multi-Sensor Integration:** Cross-references data to minimize false positives during regular steady-state riding.
*   **Instant SMS Alerts:** Sends emergency contacts a direct Google Maps link with precise GPS coordinates.
*   **Follow-Up Updates:** Dispatches a secondary confirmation message if the rider remains stationary after an alert.
*   **Rider Override:** Features an audible horn feedback loop and a handlebar cancel button to prevent false alarms.

## Hardware Components
*   **Controller:** ESP32 Microcontroller
*   **IMU:** MPU6050 (Accelerometer + Gyroscope)
*   **GSM Module:** SIM800L
*   **GPS Module:** NEO-6M
*   **Contact Sensors:** Capacitive Handlebar Touch Strip & Leg Rest Pressure Sensor
