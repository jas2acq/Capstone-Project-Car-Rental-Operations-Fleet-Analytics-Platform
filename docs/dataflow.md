## Data Ingestion Flow: Car Rental Operations & Fleet Analytics Platform

This document provides a detailed overview of the Data Ingestion phase for the Car Rental Operations & Fleet Analytics Platform, leveraging both streaming and batch architectures to address key operational challenges and support fleet analytics. The context is derived from the project's problem statement, data sources, and governance requirements.

***

## Table of Contents
* [Context and Problem Statement](#context-and-problem-statement)
* [Data Sources and Characteristics](#data-sources-and-characteristics)
    * [Stream Sources](#stream-sources)
    * [Batch Sources](#batch-sources)
* [Data Ingestion Architecture (AWS Cloud)](#data-ingestion-architecture-aws-cloud)
    * [Streaming Data Ingestion](#streaming-data-ingestion)
    * [Batch Data Ingestion](#batch-data-ingestion)

***

## Context and Problem Statement

The platform is designed to overcome critical operational challenges faced by a multinational car rental company, including **fleet underutilization** and **overbooking** in different cities, **customer dissatisfaction** (delayed pickups, billing disputes), and **revenue leakage**.

The solution requires a data-driven platform that provides **real-time visibility** into bookings and vehicle tracking, alongside support for **batch analytics** for demand forecasting and pricing optimization. The ingestion pipeline is the foundation for achieving these goals by consolidating all operational data.

***

## Data Sources and Characteristics

The platform consumes data from five distinct sources, categorized by their processing type and volume.

### Stream Sources
These sources drive the real-time visibility required by Operations Managers.

| Source               | Frequency & Volume          | Format   | Key Data                                                                                  |
|----------------------|----------------------------|----------|-------------------------------------------------------------------------------------------|
| **Telemetry** (GPS + IoT)          | Stream Per second           | JSON     | Location (latitude, longitude), speed, fuel_level, engine_status, tire_pressure           |
| **Booking Transactions**            | Stream Real-time            | JSON     | booking_id, vehicle_id, pickup_location, dropoff_time, amount, PII (customer name, email, license number) |

### Batch Sources
These sources support periodic analytics for Finance, Marketing, and Data Science teams.

| Source                    | Frequency & Volume    | Format       | Key Data                                                     |
|---------------------------|----------------------|--------------|--------------------------------------------------------------|
| **Fleet Inventory** (Vehicles) | Batch Hourly         | CSV/Parquet  | vehicle_id, VIN, status, mileage, next_service_due           |
| **Billing & Payments**        | Batch Daily          | CSV          | payment_id, booking_id, amount, status, PII (card number, billing address) |
| **Customer Master Data**      | Batch Weekly         | CSV/XML      | customer_id, loyalty_points, risk_flag, PII (full name, DOB, address) |

***

## Data Ingestion Architecture (AWS Cloud)

The ingestion pipeline is designed to handle high-volume streaming data (up to 200k telematics events/sec and 50k bookings/min) and diverse batch loads, landing all raw data into **Amazon S3** (Simple Storage Service).

### Streaming Data Ingestion

The ingestion process for real-time data uses **AWS Fargate** (for polling/extraction) and **Amazon MSK (Managed Streaming Kafka)** (for streaming) before landing in S3.

1.  **Polling/Extraction (①, ②):** **Fargate** polls the raw Telemetry and Booking Transactions data sources and feeds the data into their respective **MSK Topics**.
2.  **Streaming & Producer (③):** The streams from both Telematics and Booking Transactions are combined and fed by a **Producer**.
3.  **Raw Data Landing (④):** The combined, real-time streaming data is then moved directly from MSK into an **S3 bucket** designated for streaming data.

### Batch Data Ingestion

Batch data is loaded directly into the Raw Zone S3 bucket on a scheduled basis.

| Batch Source            | S3 Target Location                   | Ingestion Frequency |
|------------------------|------------------------------------|---------------------|
| **Fleet Inventory** (⑤)      | s3://raw/batch/fleet_inventory       | Hourly              |
| **Billing & Payments** (⑥)    | s3://raw/batch/billing_payments      | Daily               |
| **Customer Master Data** (⑦)  | s3://raw/batch/customer_master_data  | Weekly              |

**Note on Governance:** The raw zone must handle **PII data** (customer names, license numbers, card details, DOB) from Booking, Billing, and Customer Master sources, necessitating strict governance and security measures from the initial ingestion point.
