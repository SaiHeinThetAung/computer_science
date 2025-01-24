The terms **Latitude (Lat)**, **Longitude (Long)**, and **Altitude (Alt)** are used in both **Global Position Int** and **GPS_RAW_INT** messages, but they differ in their representation, precision, and context. Here's a breakdown of the differences:

---

### 1. **Global Position Int (MAVLink Message: GLOBAL_POSITION_INT)**
   - This message provides a **filtered, global position estimate** in integer format.
   - It typically comes from a higher-level system (e.g., an autopilot or flight controller) that fuses data from multiple sources (e.g., GPS, IMU, barometer) to provide a more accurate and stable position estimate.
   - **Latitude (Lat)** and **Longitude (Long)** are given in **degrees scaled by 10^7** (1e7). This means:
     - Latitude: `-90 * 1e7` to `+90 * 1e7`
     - Longitude: `-180 * 1e7` to `+180 * 1e7`
   - **Altitude (Alt)** is typically given in **millimeters** (mm) relative to a reference point (e.g., home altitude or sea level).
   - This message is often used for navigation and control purposes, as it provides a smoothed and reliable position estimate.

---

### 2. **GPS Raw Int (MAVLink Message: GPS_RAW_INT)**
   - This message provides **raw GPS data** directly from the GPS receiver, without any additional filtering or fusion.
   - It represents the **unprocessed GPS fix** and is useful for debugging or analyzing the quality of the GPS signal.
   - **Latitude (Lat)** and **Longitude (Long)** are also given in **degrees scaled by 10^7** (1e7), similar to `GLOBAL_POSITION_INT`.
   - **Altitude (Alt)** is typically given in **millimeters** (mm) relative to the **WGS84 ellipsoid** (a mathematical model of the Earth's shape used by GPS systems).
   - This message may include additional raw GPS data, such as:
     - Number of satellites visible
     - HDOP (Horizontal Dilution of Precision)
     - Fix type (e.g., no fix, 2D fix, 3D fix)
     - Velocity components (if available)

---

### Key Differences:
| Feature                  | Global Position Int (GLOBAL_POSITION_INT)       | GPS Raw Int (GPS_RAW_INT)                     |
|--------------------------|------------------------------------------------|-----------------------------------------------|
| **Source**               | Filtered, fused estimate (e.g., from autopilot) | Raw GPS data directly from the GPS receiver   |
| **Lat/Long Precision**   | Degrees scaled by 1e7                          | Degrees scaled by 1e7                         |
| **Altitude Reference**   | Typically relative to home or sea level        | Relative to WGS84 ellipsoid                  |
| **Use Case**             | Navigation, control, and high-level decision-making | Debugging, GPS signal analysis               |
| **Additional Data**      | May include fused velocity, heading, etc.      | Includes raw GPS metrics (e.g., HDOP, satellites) |

---

### Example:
- **GLOBAL_POSITION_INT**:
  - Lat: `37.4220000°` → `374220000` (scaled by 1e7)
  - Long: `-122.0840000°` → `-1220840000` (scaled by 1e7)
  - Alt: `100.000 meters` → `100000 mm` (relative to home or sea level)

- **GPS_RAW_INT**:
  - Lat: `37.4220000°` → `374220000` (scaled by 1e7)
  - Long: `-122.0840000°` → `-1220840000` (scaled by 1e7)
  - Alt: `100.000 meters` → `100000 mm` (relative to WGS84 ellipsoid)

---

### Summary:
- **Global Position Int** is a refined, fused position estimate suitable for navigation.
- **GPS Raw Int** is the raw, unfiltered GPS data, useful for diagnostics and analysis.
- Both use the same scaling for latitude and longitude, but the altitude reference and context differ.
