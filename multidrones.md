To handle telemetry data for **50 drones** concurrently, you'll need to modify the above code to manage multiple connections and process telemetry data in parallel. Here's how you can implement this:

### Key Considerations:
1. **Concurrent Connections**: Each drone should have its own MAVLink connection.
2. **Parallel Processing**: Use multithreading or asynchronous programming to process telemetry from all drones concurrently.
3. **Unique Telemetry Data**: Maintain separate telemetry data for each drone.
4. **Scalability**: Use queues or similar structures to manage the incoming data efficiently.

---

### Modified Implementation:

#### 1. **Use Multithreading**:
Each thread will handle telemetry data for a single drone.

#### 2. **Track Multiple Connections**:
Maintain a dictionary of connections and telemetry data for all drones.

#### 3. **Emit Data to Node.js**:
Include a unique identifier for each drone when emitting data.

---

```python
import socketio
from pymavlink import mavutil
import threading
import time
import math

# Connect to Node.js server
sio = socketio.Client()
try:
    sio.connect('http://localhost:3001', transports=['websocket'])  # Use 'websocket' explicitly
    print("Connected to Node.js server")
except Exception as e:
    print(f"Failed to connect to Node.js server: {e}")
    exit()

# Function to calculate distance between two GPS coordinates
def calculate_distance(lat1, lon1, lat2, lon2):
    R = 6371000  # Earth radius in meters
    phi1, phi2 = math.radians(lat1), math.radians(lat2)
    delta_phi = math.radians(lat2 - lat1)
    delta_lambda = math.radians(lon2 - lon1)
    a = (math.sin(delta_phi / 2) ** 2 +
         math.cos(phi1) * math.cos(phi2) * math.sin(delta_lambda / 2) ** 2)
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))
    return R * c

# Telemetry handler for each drone
def handle_drone_telemetry(drone_id, connection_url):
    connection = mavutil.mavlink_connection(connection_url)
    connection.wait_heartbeat()
    print(f"Heartbeat received from Drone {drone_id}")

    home_location = {'lat': 16.7745, 'lon': 96.1552}
    prev_lat = prev_lon = None
    total_distance = 0

    telemetry_data = {
        'drone_id': drone_id,  # Unique ID for each drone
        'lat': None,
        'lon': None,
        'alt': None,
        'dist_traveled': 0,
        'dist_to_home': None,
        'groundspeed': None,
        'battery_voltage': None,
        'time_in_air': 0
    }

    start_time = time.time()

    while True:
        try:
            msg = connection.recv_match(
                type=['GLOBAL_POSITION_INT', 'VFR_HUD', 'SYS_STATUS'],
                blocking=True, timeout=1
            )

            if not msg:
                continue

            # Process telemetry messages
            if msg.get_type() == 'GLOBAL_POSITION_INT':
                telemetry_data['lat'] = msg.lat / 1e7
                telemetry_data['lon'] = msg.lon / 1e7
                telemetry_data['alt'] = msg.alt / 1000.0
                telemetry_data['dist_to_home'] = calculate_distance(
                    telemetry_data['lat'], telemetry_data['lon'], home_location['lat'], home_location['lon']
                )

                if prev_lat and prev_lon:
                    distance = calculate_distance(prev_lat, prev_lon, telemetry_data['lat'], telemetry_data['lon'])
                    total_distance += distance
                    telemetry_data['dist_traveled'] = round(total_distance, 2)

                prev_lat, prev_lon = telemetry_data['lat'], telemetry_data['lon']

            elif msg.get_type() == 'VFR_HUD':
                telemetry_data['groundspeed'] = msg.groundspeed

            elif msg.get_type() == 'SYS_STATUS':
                telemetry_data['battery_voltage'] = msg.voltage_battery / 1000.0 if msg.voltage_battery != 65535 else None

            # Time in air
            telemetry_data['time_in_air'] = round(time.time() - start_time, 2)

            # Emit telemetry data to Node.js
            sio.emit('aircraft-telemetry', telemetry_data)
            print(f"Drone {drone_id}: Sent telemetry: {telemetry_data}")

        except Exception as e:
            print(f"Error handling Drone {drone_id}: {e}")
            break

# Start threads for each drone
def main():
    drone_connections = {
        1: 'tcp:127.0.0.1:14550',
        2: 'tcp:127.0.0.1:14551',
        3: 'tcp:127.0.0.1:14552',
        # Add up to 50 drone connections
    }

    threads = []
    for drone_id, connection_url in drone_connections.items():
        thread = threading.Thread(target=handle_drone_telemetry, args=(drone_id, connection_url))
        threads.append(thread)
        thread.start()

    for thread in threads:
        thread.join()

if __name__ == "__main__":
    main()
```

---

### Explanation:

1. **Drone Connections**:
   - Each drone has a unique MAVLink connection URL (e.g., `tcp:127.0.0.1:14550`).
   - Add all 50 drone connection URLs to the `drone_connections` dictionary.

2. **Threading**:
   - Each drone connection runs in its thread, processing telemetry data independently.

3. **Telemetry Data Structure**:
   - Each drone's telemetry includes a `drone_id` to uniquely identify it in the Node.js server.

4. **SocketIO Emission**:
   - Telemetry data for each drone is emitted with its unique `drone_id`.

5. **Scalability**:
   - Threads ensure that all drones are handled concurrently. For more scalability, consider using asynchronous programming with libraries like `asyncio`.

---

This implementation handles multiple drones efficiently and can scale to 50 connections while streaming telemetry data in real time to the Node.js server.