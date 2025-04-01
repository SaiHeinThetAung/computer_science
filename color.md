
To **show both GPS heading and target heading** in your frontend, you need to extract these values from **Mission Planner via MAVLink**. Here’s where to get them:

---

### **1️⃣ GPS Heading (Current Heading)**
- **MAVLink Message**: `VFR_HUD`
- **Field**: `heading`
- **Description**: This gives the aircraft’s **current heading** in degrees (0-360°).

---

### **2️⃣ Target Heading (Desired Heading)**
- **MAVLink Message**: `NAV_CONTROLLER_OUTPUT`
- **Field**: `nav_bearing`
- **Description**: This represents the autopilot's **target heading** (where it is trying to steer the aircraft).

---

### **3️⃣ Java Code to Get Both Headings**
Modify your **Java MAVLink backend** to read both messages:

```java
import io.dronefleet.mavlink.MavlinkConnection;
import io.dronefleet.mavlink.common.NavControllerOutput;
import io.dronefleet.mavlink.common.VfrHud;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.io.IOException;
import java.net.Socket;
import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/api")
public class HeadingController {

    @GetMapping("/headings")
    public Map<String, String> getHeadings() {
        Map<String, String> response = new HashMap<>();
        try {
            // Connect to Mission Planner
            MavlinkConnection connection = MavlinkConnection.create(
                    new Socket("127.0.0.1", 14550).getInputStream(),
                    new Socket("127.0.0.1", 14550).getOutputStream());

            Object message;
            int gpsHeading = -1;
            int targetHeading = -1;

            // Loop to capture both messages
            while ((message = connection.next()) != null) {
                if (message instanceof VfrHud) {
                    VfrHud hud = (VfrHud) message;
                    gpsHeading = hud.heading(); // Current heading from GPS
                }
                if (message instanceof NavControllerOutput) {
                    NavControllerOutput nav = (NavControllerOutput) message;
                    targetHeading = nav.navBearing(); // Target heading from autopilot
                }
                if (gpsHeading != -1 && targetHeading != -1) {
                    break; // Stop when both values are retrieved
                }
            }

            if (gpsHeading == -1) {
                response.put("error", "No GPS heading received");
            } else {
                response.put("gpsHeading", String.valueOf(gpsHeading));
            }

            if (targetHeading == -1) {
                response.put("error", "No target heading received");
            } else {
                response.put("targetHeading", String.valueOf(targetHeading));
            }

        } catch (IOException e) {
            response.put("error", "Connection error: " + e.getMessage());
        }
        return response;
    }
}
```

---

### **4️⃣ Frontend Code to Show Both Lines**
Modify your **React frontend** to fetch and display both headings:

```javascript
import { useEffect, useState } from "react";

function HeadingIndicator() {
  const [gpsHeading, setGpsHeading] = useState(0);
  const [targetHeading, setTargetHeading] = useState(0);

  useEffect(() => {
    const fetchHeadings = async () => {
      try {
        const response = await fetch("http://localhost:8080/api/headings");
        const data = await response.json();
        if (data.gpsHeading) setGpsHeading(parseInt(data.gpsHeading));
        if (data.targetHeading) setTargetHeading(parseInt(data.targetHeading));
      } catch (error) {
        console.error("Error fetching heading data:", error);
      }
    };

    // Fetch data every 2 seconds
    const interval = setInterval(fetchHeadings, 2000);
    return () => clearInterval(interval);
  }, []);

  return (
    <div>
      <h2>Heading Display</h2>
      <p>GPS Heading (Red Line): {gpsHeading}°</p>
      <p>Target Heading (Orange Line): {targetHeading}°</p>
      <div
        style={{
          width: "150px",
          height: "150px",
          backgroundColor: "black",
          borderRadius: "50%",
          position: "relative"
        }}
      >
        <div
          style={{
            position: "absolute",
            width: "2px",
            height: "100px",
            backgroundColor: "red",
            left: "50%",
            top: "25%",
            transform: `rotate(${gpsHeading}deg)`,
            transformOrigin: "bottom center"
          }}
        />
        <div
          style={{
            position: "absolute",
            width: "2px",
            height: "100px",
            backgroundColor: "orange",
            left: "50%",
            top: "25%",
            transform: `rotate(${targetHeading}deg)`,
            transformOrigin: "bottom center"
          }}
        />
      </div>
    </div>
  );
}

export default HeadingIndicator;
```

---

### **5️⃣ Expected Behavior**
🚀 **Now, your frontend will show:**
- **Red line** for the GPS heading (actual direction).
- **Orange line** for the autopilot target heading (where it should go).
- The lines will update every **2 seconds**.

Let me know if you need further refinements! 🚁🔥

## **💡 Final Notes**
- ✅ **Backend format:** JSON `{ "gpsHeading": 230, "targetHeading": -150 }`
- ✅ **Frontend receives:** Parses JSON and displays heading data
- ✅ **Mission Planner (MAVLink) Source:**  
  - `VFR_HUD.heading` → **GPS Heading**
  - `NAV_CONTROLLER_OUTPUT.nav_bearing` → **Target Heading**
  
Let me know if you need further adjustments! 🚀🔥

### **🚀 Explanation of Green & Black Arrows in Mission Planner**
In **Mission Planner**, different arrow colors represent various navigation data.  

1️⃣ **Green Arrow (Waypoints / Next Navigation Target)**  
   - **Represents:** The direction to the next waypoint in the mission.  
   - **MAVLink Source:** `MISSION_CURRENT.seq` (Current Waypoint Index)  
   - **Value to Retrieve:**  
     - `MISSION_ITEM.x` (Latitude)  
     - `MISSION_ITEM.y` (Longitude)  
     - `MISSION_ITEM.z` (Altitude)  
   - **Use Case:** Helps in visualizing where the drone is navigating next.  

2️⃣ **Black Arrow (Previous / Last Heading or Flight Path History)**  
   - **Represents:** The drone’s previous heading direction.  
   - **MAVLink Source:** `GLOBAL_POSITION_INT.hdg` (Heading from GPS)  
   - **Value to Retrieve:**  
     - `GLOBAL_POSITION_INT.lat` (Latitude)  
     - `GLOBAL_POSITION_INT.lon` (Longitude)  
   - **Use Case:** Shows the past trajectory or last known direction.  

---

### **🛠️ How to Retrieve Data in Java**
#### **✅ Fetching the Next Waypoint (Green Arrow)**
```java
public class WaypointData {
    private int waypointIndex;
    private double latitude;
    private double longitude;
    private double altitude;

    public WaypointData(int waypointIndex, double latitude, double longitude, double altitude) {
        this.waypointIndex = waypointIndex;
        this.latitude = latitude;
        this.longitude = longitude;
        this.altitude = altitude;
    }

    public int getWaypointIndex() { return waypointIndex; }
    public double getLatitude() { return latitude; }
    public double getLongitude() { return longitude; }
    public double getAltitude() { return altitude; }
}
```
#### **✅ Fetching the Previous Heading (Black Arrow)**
```java
public class PreviousHeadingData {
    private double latitude;
    private double longitude;
    private int heading;

    public PreviousHeadingData(double latitude, double longitude, int heading) {
        this.latitude = latitude;
        this.longitude = longitude;
        this.heading = heading;
    }

    public double getLatitude() { return latitude; }
    public double getLongitude() { return longitude; }
    public int getHeading() { return heading; }
}
```

---

### **🔥 Send Data to Frontend via JSON (Spring Boot API)**
```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/drone")
public class DroneController {

    @GetMapping("/waypoint")
    public WaypointData getWaypointData() {
        return new WaypointData(3, 37.7749, -122.4194, 100.0);  // Example Data
    }

    @GetMapping("/previousHeading")
    public PreviousHeadingData getPreviousHeading() {
        return new PreviousHeadingData(37.7749, -122.4194, 270); // Example Data
    }
}
```

---

### **🔥 Fetching in Frontend (JavaScript Example)**
```javascript
fetch("http://your-backend-server.com/api/drone/waypoint")
  .then(response => response.json())
  .then(data => {
      console.log("Next Waypoint:", data);
  });

fetch("http://your-backend-server.com/api/drone/previousHeading")
  .then(response => response.json())
  .then(data => {
      console.log("Previous Heading:", data);
  });
```

---

### **🎯 Summary**
- **Green Arrow → Next Waypoint (`MISSION_CURRENT.seq`)**
- **Black Arrow → Previous Heading (`GLOBAL_POSITION_INT.hdg`)**
- **Retrieve & Send to Frontend via JSON API**
  
Let me know if you need more! 🚀🔥
