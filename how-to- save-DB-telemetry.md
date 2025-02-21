For **storing drone video recordings**, we need a **scalable and efficient storage solution** that supports both **local and cloud storage** while ensuring easy retrieval and processing.  

### **Recommended Storage Solutions**  
#### **1. Local Storage (Onboard & Ground Station)**  
- **File System (Ext4/NTFS) + PostgreSQL (Metadata Management)**  
  - Store raw video files in a structured **local directory** (e.g., `/drone_videos/{drone_id}/{date}/video.mp4`).  
  - Use **PostgreSQL** (or embedded SQLite) to manage metadata like:  
    ```sql
    CREATE TABLE video_metadata (
        id SERIAL PRIMARY KEY,
        drone_id VARCHAR(50),
        mission_id VARCHAR(50),
        file_path TEXT,
        start_time TIMESTAMP,
        end_time TIMESTAMP,
        resolution TEXT,
        frame_rate INT,
        size BIGINT
    );
    ```
  - **Local NAS (Network Attached Storage)** can be used for high-capacity storage.  

#### **2. Cloud Storage (Primary Repository)**  
- **Best Option:** **AWS S3 or MinIO (Self-hosted S3 alternative)**
  - Supports **large-scale video storage, retrieval, and CDN access**.  
  - **Integrate with Spring Boot** using **AWS SDK** or **Spring Cloud AWS**.  
  - Store video **files in S3** and maintain **metadata in TimescaleDB/PostgreSQL**.

#### **3. Video Streaming & Processing**
- **Real-time Streaming:** Use **Kafka or WebRTC** for live transmission to GCS.  
- **Post-Processing:** Integrate **FFmpeg** for compression, frame extraction, or encoding.

### **Spring Boot Implementation for Video Upload**
- **Local Save & Upload to Cloud**  
```java
@PostMapping("/upload")
public ResponseEntity<?> uploadVideo(@RequestParam("file") MultipartFile file) throws IOException {
    String localPath = "/drone_videos/" + file.getOriginalFilename();
    File localFile = new File(localPath);
    file.transferTo(localFile);

    // Upload to S3
    String s3Url = s3Service.uploadFile(localFile);

    // Save Metadata
    videoMetadataRepository.save(new VideoMetadata(droneId, missionId, s3Url));

    return ResponseEntity.ok("File uploaded: " + s3Url);
}
```

### **Summary of Best Choices**
| **Category**  | **Best Choice** |
|--------------|---------------|
| **Local DB (Metadata)** | PostgreSQL (or SQLite for offline) |
| **Cloud DB (Metadata)** | TimescaleDB (PostgreSQL-based) |
| **Local Video Storage** | File System (SSD/NAS) |
| **Cloud Video Storage** | AWS S3 / MinIO |
| **Streaming (Live)** | WebRTC / Kafka |
| **Processing** | FFmpeg (for compression, analytics) |

Would you like a **Spring Boot example** for integrating **video storage & retrieval**? 🚀


Since you're using **MySQL for the local database** and **Timescale Cloud for the cloud database**, here's the best approach:  

---

### **How to Store Data in MySQL (Local DB)?**  
✅ **Telemetry Data (1s WebSocket updates)**  
```sql
CREATE TABLE telemetry (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    drone_id VARCHAR(50),
    time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    latitude FLOAT,
    longitude FLOAT,
    altitude FLOAT,
    speed FLOAT,
    battery FLOAT
);
```

✅ **Waypoints for Mission Planner**  
```sql
CREATE TABLE waypoints (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    mission_id VARCHAR(50),
    drone_id VARCHAR(50),
    latitude FLOAT,
    longitude FLOAT,
    altitude FLOAT,
    order_index INT,
    reached BOOLEAN DEFAULT FALSE
);
```

✅ **Video Metadata (For Storing Video Information)**  
```sql
CREATE TABLE video_metadata (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    drone_id VARCHAR(50),
    mission_id VARCHAR(50),
    file_path TEXT,
    start_time TIMESTAMP,
    end_time TIMESTAMP,
    resolution VARCHAR(20),
    frame_rate INT,
    size BIGINT
);
```

---

### **How to Sync MySQL (Local) with TimescaleDB (Cloud)?**
Since **MySQL and TimescaleDB are different**, you need a **sync mechanism**:

1️⃣ **Spring Boot Batch Job (ETL Sync)**  
   - Use **Spring Batch or Debezium CDC** to transfer data from **MySQL → TimescaleDB** when online.  

2️⃣ **Manual SQL Sync (Using Python or MySQL Events)**  
   - Export from MySQL:
   ```sql
   SELECT * FROM telemetry WHERE time > NOW() - INTERVAL 1 HOUR;
   ```
   - Insert into TimescaleDB:
   ```sql
   INSERT INTO cloud_db.telemetry SELECT * FROM local_mysql.telemetry;
   ```

---

### **Final Architecture**
| **Component** | **Database** |
|--------------|-------------|
| **Local DB (Offline Mode)** | MySQL |
| **Cloud DB (Online Mode)** | TimescaleDB |
| **Telemetry & Waypoints** | Stored in MySQL, synced to TimescaleDB |
| **Video Metadata** | Stored in MySQL (Videos stored on disk/S3) |

Would you like a **Spring Boot MySQL + TimescaleDB sync example?** 🚀

### **Optimizing MySQL Database Schema for Flight Management System**

Based on your **ERD**, I'll suggest optimizations to enhance **performance, scalability, and efficiency** in MySQL.

---

## **1. General Optimization Strategies**
- **Use Proper Data Types**: Minimize storage by selecting appropriate data types.
- **Indexing**: Optimize queries using indexes on frequently searched columns.
- **Foreign Keys & Relationships**: Maintain referential integrity and optimize joins.
- **Partitioning & Sharding**: For large datasets, consider partitioning.
- **Normalization vs. Denormalization**: Normalize for data integrity but denormalize selectively for read-heavy workloads.
- **Query Optimization**: Use `EXPLAIN` to analyze query performance.

---

## **2. Schema Optimization for Each Table**
Below are improvements based on your **schema**:

### **Users Table (`users_t`)**
#### **Current Schema:**
```sql
user_email VARCHAR
user_name VARCHAR
user_password VARCHAR
user_grade VARCHAR
checked_email INTEGER
created_at TIMESTAMP
created_user VARCHAR
modified_at TIMESTAMP
modified_user VARCHAR
```
#### **Optimizations:**
- **Use `VARCHAR(255)` for email** (instead of `VARCHAR` without length).
- **Use `BOOLEAN` for `checked_email`** (instead of `INTEGER`).
- **Add Primary Key (`user_id INT AUTO_INCREMENT PRIMARY KEY`)** for faster indexing.

#### **Optimized Schema:**
```sql
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    user_email VARCHAR(255) UNIQUE NOT NULL,
    user_name VARCHAR(100) NOT NULL,
    user_password VARCHAR(255) NOT NULL,
    user_grade ENUM('admin', 'pilot', 'mechanic') NOT NULL,
    checked_email BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    modified_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```
---

### **Flight Logs Table (`flightlogs_t`)**
#### **Current Schema:**
```sql
receive_at TIMESTAMP
client_ip VARCHAR
drone_id INTEGER
lat VARCHAR
lon VARCHAR
alt VARCHAR
```
#### **Optimizations:**
- **Use `DECIMAL(10,6)` for latitude/longitude** for precision instead of `VARCHAR`.
- **Use `FLOAT` for altitude and speeds** to save space.
- **Index on `receive_at` for faster time-based queries.**

#### **Optimized Schema:**
```sql
CREATE TABLE flightlogs (
    flightlog_id INT AUTO_INCREMENT PRIMARY KEY,
    receive_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    client_ip VARCHAR(45),
    drone_id INT,
    lat DECIMAL(10,6),
    lon DECIMAL(10,6),
    alt FLOAT,
    vertical_speed FLOAT,
    wind_vel FLOAT,
    airspeed FLOAT,
    groundspeed FLOAT,
    FOREIGN KEY (drone_id) REFERENCES drones(drone_id) ON DELETE CASCADE,
    INDEX (receive_at)
);
```
---

### **Pilots Table (`pilots_t`)**
#### **Optimizations:**
- **Use `VARCHAR(10)` for country codes instead of full names**.
- **Use ENUM for `pilot_status`** (Active, Inactive, Suspended).
- **Foreign Key constraints for structured relationships**.

#### **Optimized Schema:**
```sql
CREATE TABLE pilots (
    pilot_id VARCHAR(20) PRIMARY KEY,
    pilot_name VARCHAR(100) NOT NULL,
    pilot_certno VARCHAR(50) UNIQUE NOT NULL,
    pilot_passport VARCHAR(50) UNIQUE NOT NULL,
    pilot_email VARCHAR(255) UNIQUE NOT NULL,
    pilot_phone VARCHAR(20),
    pilot_country VARCHAR(10),
    pilot_status ENUM('Active', 'Inactive', 'Suspended') DEFAULT 'Active',
    pilot_grade ENUM('Captain', 'First Officer', 'Second Officer'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    modified_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```
---

### **Drones Table (`drones_t`)**
#### **Optimizations:**
- **Use `UNSIGNED INT` for `drone_id`** (ensures positive values).
- **Index serial numbers if frequently queried**.
- **Add `co_id` as a Foreign Key to associate drones with companies**.

#### **Optimized Schema:**
```sql
CREATE TABLE drones (
    drone_id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    make_no VARCHAR(50),
    model_name VARCHAR(50),
    fc_serialno VARCHAR(50) UNIQUE,
    gps_serialno VARCHAR(50) UNIQUE,
    rc_serialno VARCHAR(50) UNIQUE,
    co_id VARCHAR(20),
    FOREIGN KEY (co_id) REFERENCES companys(co_id) ON DELETE CASCADE
);
```
---

### **Ship Table (`ships_t`)**
#### **Optimizations:**
- **Use `ship_id` as an `UNSIGNED INT`** for faster indexing.
- **Foreign Key `co_id` references companies**.
- **Index `ship_callsign` for faster lookups**.

#### **Optimized Schema:**
```sql
CREATE TABLE ships (
    ship_id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    ship_name VARCHAR(100) NOT NULL,
    ship_callsign VARCHAR(50) UNIQUE,
    ship_country VARCHAR(10),
    co_id VARCHAR(20),
    FOREIGN KEY (co_id) REFERENCES companys(co_id) ON DELETE CASCADE,
    INDEX (ship_callsign)
);
```
---

### **Materials Table (`materials_t`)**
#### **Optimizations:**
- **Use `DATE` for tracking `material_lifetime` instead of `VARCHAR`**.
- **Index `material_serialno` for efficient searches**.

#### **Optimized Schema:**
```sql
CREATE TABLE materials (
    material_serialno VARCHAR(50) PRIMARY KEY,
    material_item VARCHAR(100),
    material_model VARCHAR(50),
    material_co VARCHAR(100),
    material_storage VARCHAR(100),
    material_enterat TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    material_lifetime DATE,
    material_usagetime INT,  -- in hours
    material_replacetime DATE,
    INDEX (material_serialno)
);
```
---

## **3. Indexing Strategy**
Indexes speed up searches but increase write overhead. Recommended indexes:
```sql
CREATE INDEX idx_receive_at ON flightlogs(receive_at);
CREATE INDEX idx_pilot_email ON pilots(pilot_email);
CREATE INDEX idx_drone_serial ON drones(fc_serialno);
CREATE INDEX idx_ship_callsign ON ships(ship_callsign);
```

---

## **4. Foreign Keys for Data Integrity**
Adding **foreign keys** ensures referential integrity:
```sql
ALTER TABLE flightlogs ADD FOREIGN KEY (drone_id) REFERENCES drones(drone_id) ON DELETE CASCADE;
ALTER TABLE pilots ADD FOREIGN KEY (pilot_country) REFERENCES country_codes(code);
ALTER TABLE ships ADD FOREIGN KEY (co_id) REFERENCES companys(co_id) ON DELETE CASCADE;
```

---

## **5. Query Optimization**
### **Before Optimization (Slow Query)**
```sql
SELECT * FROM flightlogs WHERE receive_at >= '2024-01-01';
```
### **Optimized Query (Using Index)**
```sql
SELECT * FROM flightlogs WHERE receive_at >= '2024-01-01' USE INDEX (idx_receive_at);
```

---

## **6. Performance Enhancements**
### **1. Partitioning for Large Tables**
Partitioning **flight logs by date** improves query performance.
```sql
ALTER TABLE flightlogs PARTITION BY RANGE(YEAR(receive_at)) (
    PARTITION p0 VALUES LESS THAN (2022),
    PARTITION p1 VALUES LESS THAN (2023),
    PARTITION p2 VALUES LESS THAN (2024),
    PARTITION p3 VALUES LESS THAN (MAXVALUE)
);
```

### **2. Read-Write Optimization**
- Use **Read Replica** in MySQL for read-heavy workloads.
- Enable **Query Caching** for frequently accessed data.

### **3. Use `VARCHAR` Instead of `TEXT` When Possible**
- `VARCHAR(255)` is **faster** than `TEXT` for indexing.

---

## **Final Thoughts**
- **MySQL is a great choice** for structured, relational data.
- **AWS AuroraDB** could be used if cloud scalability is needed.
- **Optimizations include indexing, foreign keys, partitioning, and query optimization**.

Would you like me to generate sample queries for **data retrieval and reporting**? 🚀

### **MySQL Pricing Overview** 💰  

The cost of using **MySQL** depends on whether you choose a **self-hosted, managed, or cloud-based** solution. Here’s a breakdown of different pricing models:

---

## **1. Free & Self-Hosted (On-Premises) – $0**
✅ **Best if you want full control and minimal cost**  

| **Version** | **Pricing** | **Pros** | **Cons** |
|------------|------------|----------|----------|
| **MySQL Community Edition** | **Free** (Open-source) | No licensing cost, full control, supports most workloads | Requires self-management, no built-in high availability |
| **MySQL on your own server** | **$0 (only server costs)** | No additional fees beyond hardware and maintenance | Requires DBA expertise for performance tuning |

🚀 **Best for:** Developers, startups, and small businesses who can handle **database administration (DBA)**.

---

## **2. MySQL on Cloud (Managed)**
✅ **Best if you want scalability, backups, and less maintenance**  

| **Provider** | **Pricing (Estimated)** | **Pros** | **Cons** |
|-------------|----------------|----------|----------|
| **Amazon RDS for MySQL** | **$0.02 – $0.08 per hour (~$15 – $60/month) for small instances** | Fully managed, automated backups, security | More expensive than self-hosted |
| **AWS Aurora MySQL** | **$0.10 – $0.30 per hour (~$75 – $220/month)** | High performance, auto-scaling, failover | Costs more than RDS |
| **Google Cloud SQL for MySQL** | **$0.02 – $0.10 per hour (~$15 – $75/month)** | Fully managed, easy to integrate with GCP | Limited fine-tuning options |
| **Azure Database for MySQL** | **$0.02 – $0.10 per hour (~$15 – $75/month)** | Scalable, integrated with Azure services | Microsoft ecosystem dependency |

🚀 **Best for:** Medium to large businesses needing **scalability, backups, and automatic updates**.

---

## **3. MySQL Enterprise Edition (Paid)**
✅ **Best for enterprises needing **advanced security & support**  

| **Plan** | **Pricing** | **Pros** | **Cons** |
|---------|------------|----------|----------|
| **Standard** | **$2,000 per year** | 24/7 support, advanced monitoring | High upfront cost |
| **Enterprise** | **$5,000 per year** | Security features (encryption, firewall, audit logs) | Expensive |
| **Cluster Carrier Grade** | **$10,000 per year** | High availability, best for telcos and banks | Very costly |

🚀 **Best for:** **Banks, telecom companies, and mission-critical applications** needing **enterprise-grade support**.

---

## **4. My Recommendation Based on Cost & Needs**
### **If cost is the priority → Self-hosted MySQL (Free)**
- Install MySQL **Community Edition** on your **own server (VPS, local machine, or Docker)**.
- You’ll only pay for **hardware and maintenance**.

### **If you want scalability but lower cost → Amazon RDS for MySQL (~$15+/month)**
- Fully managed, **automatic backups, and updates**.
- More cost-effective than **AWS Aurora** but still cloud-based.

### **If performance & auto-scaling is critical → AWS Aurora (~$75+/month)**
- **Better than regular MySQL** for high workloads.
- **Expensive** but removes **manual database maintenance**.

---

### **Final Verdict** 🏆  
- If you're running a small to mid-sized **flight management** system, **self-hosted MySQL** is the **cheapest** and most flexible.
- If you **don’t want to manage databases manually**, go for **Amazon RDS (~$15/month)**.
- If **high traffic and performance** are expected, **AWS Aurora MySQL (~$75+/month)** is the best.

Would you like help setting up MySQL **on-premises or on AWS?** 🚀
