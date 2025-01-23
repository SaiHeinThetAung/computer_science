The provided code focuses on handling telemetry data, drone management, and WebSocket communication in a FastAPI application. If you want to extend the code for **multi-drone** support, you need to ensure that the application can simultaneously manage telemetry data for multiple drones and handle individual WebSocket connections for each drone.

Below is an analysis of what changes and enhancements are required for multi-drone support:

---

### **1. Multi-Drone Telemetry Support**
The code already maintains a dictionary `drones_data` to store telemetry data for each drone, indexed by `drone_id`. This structure inherently supports multiple drones, but the `fetch_telemetry` function needs to handle multiple drones concurrently.

#### Solution:
- Use **async tasks** to fetch telemetry data for multiple drones simultaneously.

#### Code Adjustments:
In the `fetch_telemetry` function, ensure that a new task is created for each drone.

```python
from asyncio import create_task

async def start_telemetry(drone_id: str, connection_string: str):
    """
    Start telemetry data fetch for a specific drone.
    """
    create_task(fetch_telemetry(drone_id, connection_string))

# Example usage for multiple drones
drones = {
    "drone1": "udp:127.0.0.1:14550",
    "drone2": "udp:127.0.0.1:14551",
}

async def start_all_drones():
    for drone_id, connection_string in drones.items():
        await start_telemetry(drone_id, connection_string)
```

---

### **2. WebSocket Connections for Multiple Drones**
The `websocket_endpoint` function is designed to handle a single WebSocket connection for a specific drone. To support multiple drones:
- Each drone's WebSocket session should be maintained independently.
- Ensure proper synchronization when updating telemetry data.

#### Solution:
Use a dictionary to store WebSocket connections indexed by `drone_id`.

#### Code Adjustments:
```python
websockets = {}

@router.websocket("/{drone_id}")
async def websocket_endpoint(websocket: WebSocket, drone_id: str):
    await websocket.accept()
    websockets[drone_id] = websocket  # Store the WebSocket connection for the drone

    try:
        while True:
            if drone_id in drones_data:
                telemetry = drones_data[drone_id]
                await websocket.send_json(telemetry)  # Send telemetry data
            await asyncio.sleep(1)  # Adjust the interval as needed
    except WebSocketDisconnect:
        print(f"WebSocket disconnected for drone {drone_id}")
        del websockets[drone_id]  # Remove the WebSocket connection
```

---

### **3. Database Schema Enhancements**
The current database schema seems to support basic drone attributes. For multi-drone support, ensure each drone has a unique identifier and attributes to distinguish telemetry streams.

#### Suggestions:
- Add a unique `drone_id` column to the `DroneModel`.
- Track connection status or assign unique connection strings for each drone.

Example Schema:
```python
class DroneModel(Base):
    __tablename__ = "drones"

    id = Column(Integer, primary_key=True, index=True)
    drone_id = Column(String, unique=True, index=True)
    drone_type = Column(String)
    category = Column(String)
    # Additional fields...
```

---

### **4. Multi-Drone Telemetry Updates**
When telemetry data is received, ensure it is appropriately updated for the corresponding drone in `drones_data`.

#### Adjustments in `fetch_telemetry`:
Use `drone_id` as the key when updating telemetry data:
```python
if msg.get_type() == 'GLOBAL_POSITION_INT':
    drones_data[drone_id]['lat'] = msg.lat / 1e7
    drones_data[drone_id]['lon'] = msg.lon / 1e7
    # Update other fields...
```

---

### **5. Handling Multiple Waypoints**
The `fetch_waypoints` function supports fetching waypoints for a single drone. For multiple drones:
- Modify the function to accept `drone_id` and store waypoints separately for each drone.

#### Code Adjustments:
```python
def fetch_waypoints(connection, drone_id):
    waypoints = []
    logger.info(f"Requesting waypoint list for {drone_id}...")
    # Fetch and process waypoints for the specific drone
    drones_data[drone_id]['waypoints'] = waypoints
```

---

### **6. Real-Time WebSocket Updates**
Ensure WebSocket connections are updated in real-time when new telemetry data is received.

#### Adjustments in `fetch_telemetry`:
After updating `drones_data`, send data to the corresponding WebSocket if it exists:
```python
if drone_id in websockets:
    websocket = websockets[drone_id]
    await websocket.send_json(drones_data[drone_id])
```

---

### **7. Managing Disconnected Drones**
Handle scenarios where a drone disconnects or telemetry is unavailable:
- Remove the drone's data from `drones_data`.
- Close the WebSocket connection.

#### Adjustments:
```python
if not msg:
    print(f"Drone {drone_id} disconnected.")
    drones_data.pop(drone_id, None)
    if drone_id in websockets:
        await websockets[drone_id].close()
        del websockets[drone_id]
    break
```

---

### **8. Running the System**
Create a function to start telemetry for all drones:
```python
async def main():
    await start_all_drones()

if __name__ == "__main__":
    import uvicorn
    asyncio.run(main())
    uvicorn.run("app:app", host="0.0.0.0", port=8000, reload=True)

