To save this telemetry data in MySQL, follow these steps:

---

## **1️⃣ Create a Table for Telemetry Data**
Since telemetry data contains various attributes like latitude, longitude, altitude, waypoints, and battery status, we need a structured table.

```sql
CREATE TABLE telemetry_t (
    telemetry_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    drone_id VARCHAR(50) NOT NULL,
    lat DECIMAL(10, 7),
    lon DECIMAL(10, 7),
    alt FLOAT,
    dist_traveled FLOAT,
    wp_dist FLOAT,
    dist_to_home FLOAT,
    vertical_speed FLOAT,
    wind_vel FLOAT,
    airspeed FLOAT,
    groundspeed FLOAT,
    roll FLOAT,
    pitch FLOAT,
    yaw FLOAT,
    toh FLOAT,
    tot FLOAT,
    time_in_air FLOAT,
    time_in_air_min_sec FLOAT,
    gps_hdop FLOAT,
    battery_voltage FLOAT,
    battery_current FLOAT,
    ch3percent FLOAT,
    ch3out INT,
    ch9out INT,
    ch10out INT,
    ch11out INT,
    ch12out INT,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (drone_id) REFERENCES drones_t(drone_id) ON DELETE CASCADE
);
```

---

## **2️⃣ Create a Table for Waypoints (One-to-Many Relationship)**
Each telemetry record has multiple waypoints. We need a separate table for waypoints.

```sql
CREATE TABLE waypoints_t (
    waypoint_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    telemetry_id BIGINT NOT NULL,
    seq INT,
    x DECIMAL(10, 7),
    y DECIMAL(10, 7),
    z FLOAT,
    FOREIGN KEY (telemetry_id) REFERENCES telemetry_t(telemetry_id) ON DELETE CASCADE
);
```

---

## **3️⃣ Insert Data into MySQL from Python**
Since you're getting this data from a **terminal or API**, we can insert it into MySQL using **Python with MySQL Connector**.

### **Install MySQL Connector**
```bash
pip install mysql-connector-python
```

### **Python Script to Save Telemetry Data**
```python
import mysql.connector
import json

# Sample telemetry data
telemetry_data = {
    "drone_id": "drone3",
    "lat": 35.0739314,
    "lon": 129.0919685,
    "alt": 76.84,
    "dist_traveled": 64.83,
    "wp_dist": 307.828,
    "dist_to_home": 3843446.32,
    "vertical_speed": -1.14,
    "wind_vel": 0.0024324,
    "airspeed": 25.806,
    "groundspeed": 26.337,
    "roll": 59.265,
    "pitch": 3.145,
    "yaw": 106.284,
    "toh": 145928.68,
    "tot": 11.69,
    "time_in_air": 3.15,
    "time_in_air_min_sec": 0.03,
    "gps_hdop": 1.21,
    "battery_voltage": 0.0,
    "battery_current": 0.0,
    "ch3percent": 76.9,
    "ch3out": 1769,
    "ch9out": 0,
    "ch10out": 0,
    "ch11out": 0,
    "ch12out": 0,
    "waypoints": [
        {"seq": 0, "x": 35.0766974, "y": 129.0921083, "z": 43.75},
        {"seq": 1, "x": 0.0, "y": 0.0, "z": 30.0}
    ]
}

# Connect to MySQL
conn = mysql.connector.connect(
    host="localhost",
    user="root",
    password="yourpassword",
    database="flight_management"
)
cursor = conn.cursor()

# Insert telemetry data
query = """INSERT INTO telemetry_t 
    (drone_id, lat, lon, alt, dist_traveled, wp_dist, dist_to_home, vertical_speed, wind_vel, 
     airspeed, groundspeed, roll, pitch, yaw, toh, tot, time_in_air, time_in_air_min_sec, 
     gps_hdop, battery_voltage, battery_current, ch3percent, ch3out, ch9out, ch10out, ch11out, ch12out)
    VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)"""

values = (
    telemetry_data["drone_id"], telemetry_data["lat"], telemetry_data["lon"], telemetry_data["alt"],
    telemetry_data["dist_traveled"], telemetry_data["wp_dist"], telemetry_data["dist_to_home"],
    telemetry_data["vertical_speed"], telemetry_data["wind_vel"], telemetry_data["airspeed"],
    telemetry_data["groundspeed"], telemetry_data["roll"], telemetry_data["pitch"],
    telemetry_data["yaw"], telemetry_data["toh"], telemetry_data["tot"], telemetry_data["time_in_air"],
    telemetry_data["time_in_air_min_sec"], telemetry_data["gps_hdop"], telemetry_data["battery_voltage"],
    telemetry_data["battery_current"], telemetry_data["ch3percent"], telemetry_data["ch3out"],
    telemetry_data["ch9out"], telemetry_data["ch10out"], telemetry_data["ch11out"], telemetry_data["ch12out"]
)

cursor.execute(query, values)
telemetry_id = cursor.lastrowid  # Get the inserted row ID

# Insert waypoints
waypoint_query = """INSERT INTO waypoints_t (telemetry_id, seq, x, y, z) VALUES (%s, %s, %s, %s, %s)"""
for wp in telemetry_data["waypoints"]:
    cursor.execute(waypoint_query, (telemetry_id, wp["seq"], wp["x"], wp["y"], wp["z"]))

# Commit and close connection
conn.commit()
cursor.close()
conn.close()

print("Telemetry data saved successfully!")
```

---

## **4️⃣ Verify Data in MySQL**
After running the Python script, you can check the data using SQL queries:

### **Check Telemetry Data**
```sql
SELECT * FROM telemetry_t WHERE drone_id = 'drone3';
```

### **Check Waypoints for a Specific Telemetry Record**
```sql
SELECT * FROM waypoints_t WHERE telemetry_id = 1;
```

---

## **✅ Next Steps**
1. **Automate Data Collection**  
   - Integrate this script into your **drone's telemetry system** (e.g., via MQTT, WebSocket, or HTTP API).
   - Schedule periodic inserts using a **background task**.

2. **Optimize Database for Real-Time Processing**  
   - Index `telemetry_t(drone_id, timestamp)` for fast querying.
   - Use **partitioning** if the dataset grows large.

3. **Build API for Data Retrieval**  
   - Create a **Spring Boot API** to query and display telemetry data.
   - Example: `/api/telemetry?drone_id=drone3`

---

This solution **efficiently saves and organizes telemetry data in MySQL**.  
Would you like a **Spring Boot API** example to fetch telemetry data? 🚀

### **Saving Drone Telemetry Data in TimescaleDB (Cloud Database)**
Since you're using **Timescale Cloud** as your cloud database, we need to store **time-series drone telemetry data** efficiently. TimescaleDB is built on PostgreSQL and optimized for handling time-series data.

---

## **🔹 1️⃣ Create a Telemetry Table in TimescaleDB**
We will create a **hypertable** in TimescaleDB, which is optimized for time-series data storage.

### **Connect to TimescaleDB (Cloud)**
1. **Get your Timescale Cloud credentials** (host, username, password, database name).
2. **Use `psql` or a GUI tool** (e.g., pgAdmin, DBeaver) to connect.

### **Run This SQL in TimescaleDB**
```sql
CREATE TABLE telemetry_t (
    telemetry_id SERIAL PRIMARY KEY,
    drone_id VARCHAR(50) NOT NULL,
    lat DOUBLE PRECISION,
    lon DOUBLE PRECISION,
    alt DOUBLE PRECISION,
    dist_traveled DOUBLE PRECISION,
    wp_dist DOUBLE PRECISION,
    dist_to_home DOUBLE PRECISION,
    vertical_speed DOUBLE PRECISION,
    wind_vel DOUBLE PRECISION,
    airspeed DOUBLE PRECISION,
    groundspeed DOUBLE PRECISION,
    roll DOUBLE PRECISION,
    pitch DOUBLE PRECISION,
    yaw DOUBLE PRECISION,
    toh DOUBLE PRECISION,
    tot DOUBLE PRECISION,
    time_in_air DOUBLE PRECISION,
    time_in_air_min_sec DOUBLE PRECISION,
    gps_hdop DOUBLE PRECISION,
    battery_voltage DOUBLE PRECISION,
    battery_current DOUBLE PRECISION,
    ch3percent DOUBLE PRECISION,
    ch3out INT,
    ch9out INT,
    ch10out INT,
    ch11out INT,
    ch12out INT,
    timestamp TIMESTAMPTZ DEFAULT now()
);

-- Convert it into a Hypertable (for better time-series performance)
SELECT create_hypertable('telemetry_t', 'timestamp');
```
✅ **Why use `create_hypertable`?**  
- **Optimized storage & faster queries** for time-series data.  
- **Automatic partitioning** for better scalability.

---

## **🔹 2️⃣ Create a Table for Waypoints**
Since each telemetry record has multiple **waypoints**, we use a separate table.

```sql
CREATE TABLE waypoints_t (
    waypoint_id SERIAL PRIMARY KEY,
    telemetry_id INT NOT NULL,
    seq INT,
    x DOUBLE PRECISION,
    y DOUBLE PRECISION,
    z DOUBLE PRECISION,
    FOREIGN KEY (telemetry_id) REFERENCES telemetry_t(telemetry_id) ON DELETE CASCADE
);
```

---

## **🔹 3️⃣ Python Script to Save Telemetry Data in TimescaleDB**
Since TimescaleDB is **PostgreSQL-based**, we use **`psycopg2`** to connect.

### **Install the PostgreSQL Connector**
```bash
pip install psycopg2
```

### **Python Script to Insert Data**
```python
import psycopg2
import json
from datetime import datetime

# Timescale Cloud connection details
DB_HOST = "your-timescale-cloud-host"
DB_NAME = "your_database"
DB_USER = "your_user"
DB_PASS = "your_password"
DB_PORT = "5432"

# Sample telemetry data
telemetry_data = {
    "drone_id": "drone3",
    "lat": 35.0739314,
    "lon": 129.0919685,
    "alt": 76.84,
    "dist_traveled": 64.83,
    "wp_dist": 307.828,
    "dist_to_home": 3843446.32,
    "vertical_speed": -1.14,
    "wind_vel": 0.0024324,
    "airspeed": 25.806,
    "groundspeed": 26.337,
    "roll": 59.265,
    "pitch": 3.145,
    "yaw": 106.284,
    "toh": 145928.68,
    "tot": 11.69,
    "time_in_air": 3.15,
    "time_in_air_min_sec": 0.03,
    "gps_hdop": 1.21,
    "battery_voltage": 0.0,
    "battery_current": 0.0,
    "ch3percent": 76.9,
    "ch3out": 1769,
    "ch9out": 0,
    "ch10out": 0,
    "ch11out": 0,
    "ch12out": 0,
    "waypoints": [
        {"seq": 0, "x": 35.0766974, "y": 129.0921083, "z": 43.75},
        {"seq": 1, "x": 0.0, "y": 0.0, "z": 30.0}
    ]
}

# Connect to TimescaleDB
conn = psycopg2.connect(
    host=DB_HOST, database=DB_NAME, user=DB_USER, password=DB_PASS, port=DB_PORT
)
cursor = conn.cursor()

# Insert telemetry data
query = """INSERT INTO telemetry_t 
    (drone_id, lat, lon, alt, dist_traveled, wp_dist, dist_to_home, vertical_speed, wind_vel, 
     airspeed, groundspeed, roll, pitch, yaw, toh, tot, time_in_air, time_in_air_min_sec, 
     gps_hdop, battery_voltage, battery_current, ch3percent, ch3out, ch9out, ch10out, ch11out, ch12out, timestamp)
    VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, NOW())
    RETURNING telemetry_id;"""

values = (
    telemetry_data["drone_id"], telemetry_data["lat"], telemetry_data["lon"], telemetry_data["alt"],
    telemetry_data["dist_traveled"], telemetry_data["wp_dist"], telemetry_data["dist_to_home"],
    telemetry_data["vertical_speed"], telemetry_data["wind_vel"], telemetry_data["airspeed"],
    telemetry_data["groundspeed"], telemetry_data["roll"], telemetry_data["pitch"],
    telemetry_data["yaw"], telemetry_data["toh"], telemetry_data["tot"], telemetry_data["time_in_air"],
    telemetry_data["time_in_air_min_sec"], telemetry_data["gps_hdop"], telemetry_data["battery_voltage"],
    telemetry_data["battery_current"], telemetry_data["ch3percent"], telemetry_data["ch3out"],
    telemetry_data["ch9out"], telemetry_data["ch10out"], telemetry_data["ch11out"], telemetry_data["ch12out"]
)

cursor.execute(query, values)
telemetry_id = cursor.fetchone()[0]  # Get inserted telemetry ID

# Insert waypoints
waypoint_query = """INSERT INTO waypoints_t (telemetry_id, seq, x, y, z) VALUES (%s, %s, %s, %s, %s)"""
for wp in telemetry_data["waypoints"]:
    cursor.execute(waypoint_query, (telemetry_id, wp["seq"], wp["x"], wp["y"], wp["z"]))

# Commit transaction
conn.commit()

# Close connection
cursor.close()
conn.close()

print("Telemetry data saved successfully in TimescaleDB!")
```

---

## **🔹 4️⃣ Query Data from TimescaleDB**
### **Get Latest Telemetry Data for a Drone**
```sql
SELECT * FROM telemetry_t WHERE drone_id = 'drone3' ORDER BY timestamp DESC LIMIT 10;
```

### **Get Waypoints for a Specific Flight**
```sql
SELECT * FROM waypoints_t WHERE telemetry_id = 1;
```

### **Get Average Speed of Drone Over Time**
```sql
SELECT time_bucket('1 minute', timestamp) AS time_interval, AVG(groundspeed)
FROM telemetry_t
WHERE drone_id = 'drone3'
GROUP BY time_interval
ORDER BY time_interval DESC;
```

---

## **✅ Next Steps**
1. **Integrate with Spring Boot**  
   - Expose telemetry data via a **REST API** (`/api/telemetry`).
   - Fetch live data from TimescaleDB.

2. **Real-Time Analytics**  
   - Use **Grafana** to visualize drone data.

3. **Optimize Database Performance**  
   - Use **continuous aggregates** to speed up queries.

---

🚀 **Would you like a Spring Boot REST API example to fetch telemetry data from TimescaleDB?**
