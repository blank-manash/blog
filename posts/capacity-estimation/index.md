<!--
.. title: Capacity Estimation
.. slug: capacity-estimation
.. date: 2024-10-31 17:53:19 UTC+05:30
.. has_math: true
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text
-->
### Capacity Estimation in System Design: Quick Reference Guide

This guide provides a concise overview of storage and compute estimation, common sizes, and solutions by scale for system design interviews and real-world applications.

<!-- TEASER_END -->
---

## 1. Storage Estimation

### Key Estimations for Storage:

| **Item**                   | **Avg Size**       | **Examples**                                                   |
|--------------------------|--------------------|----------------------------------------------------------------|
| **Text Message**             | 50 bytes          | Chat messages                                                  |
| **Tweet**                    | 200 bytes         | Includes hashtags, metadata                                    |
| **User Profile**             | 1-5 KB            | Profile info, settings, and preferences                        |
| **Image (compressed)**       | 100-500 KB        | JPG/PNG, low to medium resolution                              |
| **Request Payload**      | 50 KB             | POST/PUT Request                                               |
| **Video (1 min)**            | 5-10 MB           | Compressed, e.g., 720p                                         |
| **Log Entry**                | 100 bytes         | Timestamp, log level, message                                  |
| **Sensor Data Point**        | 100 bytes         | IoT data with metadata                                         |
| **Machine Learning Model**   | 100 MB - 1 GB     | Simple to complex models                                       |

### Storage Solutions by Scale:

| **Data Volume**            | **Suggested Solution**                     | **Examples**               |
|----------------------------|--------------------------------------------|----------------------------|
| **< 10 GB**                    | Single DB Instance                         | MySQL, PostgreSQL          |
| **10 GB - 1 TB**               | Partitioned DB or file storage             | PostgreSQL, AWS S3         |
| **1 TB - 100 TB**              | Distributed storage or sharded DB          | MongoDB, Cassandra, HDFS   |
| **> 100 TB**                   | Data lake or distributed file system       | AWS S3, Hadoop HDFS        |

---

## 2. Compute Estimation

### Key Estimations for Compute:

- **QPS Calculation**: \\( \text{QPS} = \frac{\text{DAU} \times \text{Actions/User}}{86,400} \\)
- **Peak Load**: Generally 3-5x the average QPS for consumer applications.
- **Example**: 1M DAU with 20 actions/day → \\( \approx 20 \\) QPS avg, ~100 QPS peak.

### Compute Solutions by Scale:

| **QPS**                    | **Suggested Solution**                    | **Examples**                                 |
|----------------------------|-------------------------------------------|----------------------------------------------|
| **< 100 QPS**                  | Single server or small cluster            | EC2 instance, Kubernetes pod                 |
| **100 - 1000 QPS**             | Autoscaling cluster                       | AWS ECS, GCP Compute Engine                  |
| **1000 - 10,000 QPS**          | Load-balanced cluster with caching        | Redis, Kubernetes, Load balancer             |
| **> 10,000 QPS**               | Distributed, multi-region setup           | Global load balancers, sharded DB            |

---

## Common Use Case Scenarios

| **Scenario**                | **Estimated Usage**                             | **Solution**                                     |
|-----------------------------|-------------------------------------------------|--------------------------------------------------|
| **1M DAU Chat App**             | 20 QPS peak, 1 GB/day                         | Autoscaling, Redis cache                         |
| **50M Tweets/Day**              | 600 QPS avg, 2.5 TB/year                       | Sharded DB, multi-region storage                 |
| **10K QPS E-commerce**          | 500 MB/s data throughput                       | Load balancing, distributed caching              |
| **1 PB Data Lake**              | 100 TB/month growth                            | Data lake, e.g., S3, Hadoop HDFS                 |
| **100M Video Views/Day**        | 1 PB storage monthly, 2 Gbps peak              | CDN, global load balancer                        |
| **5M User Transactions/Day**    | 50 QPS avg, 5 TB storage/year                  | Sharded RDBMS, caching, high consistency         |

---

### Quick Calculation Tips

- **Time Conversions**:
  - 1 day ≈ \\(10^5\\) seconds, 1 month ≈ \\(10^6\\) seconds
- **Bytes Conversions**:
  - 1 KB ≈ \\(10^3\\) bytes, 1 MB ≈ \\(10^6\\) bytes, 1 GB ≈ \\(10^9\\) bytes, 1 TB ≈ \\(10^{12}\\) bytes

### Bytes to Powers of 10 Conversion Table

| **Unit**          | **In Powers of 10**      | **Approximation**                   |
|-------------------|-------------------------|---------------------------|
| **Kilobyte (KB)** | \\( 10^3 \\)               | 1 thousand bytes              |
| **Megabyte (MB)** | \\( 10^6 \\)               | 1 million bytes               |
| **Gigabyte (GB)** | \\( 10^9 \\)               | 1 billion bytes               |
| **Terabyte (TB)** | \\( 10^{12} \\) | 1 trillion bytes              |
| **Petabyte (PB)** | \\( 10^{15} \\) | 1 quadrillion bytes              |
| **Exabyte (EB)**  | \\( 10^{18} \\) | 1 quintillion bytes               |
