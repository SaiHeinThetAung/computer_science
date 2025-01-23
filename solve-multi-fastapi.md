When using **FastAPI** for multi-drone telemetry data transmission via WebSocket to the UI, you might face issues such as:

- **Concurrency problems**: Handling multiple WebSocket connections simultaneously.
- **Data broadcast issues**: Sending telemetry data from multiple drones to the respective or all connected clients.
- **Performance bottlenecks**: Ensuring real-time, low-latency communication.
- **Improper handling of WebSocket lifecycle**: Dropped connections or unhandled exceptions.

---

### **Solution Steps**
To address these issues, here’s a step-by-step approach to ensure proper handling of multi-drone telemetry data in **FastAPI**:

---

### 1. **Enable Concurrency with Async WebSocket**
FastAPI uses `asyncio`, which supports handling multiple WebSocket connections concurrently. Use `async` functions to manage multiple drones and UI connections.

**Example WebSocket Setup:**
```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from typing import List

app = FastAPI()

class ConnectionManager:
    def __init__(self):
        self.active_connections: List[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)

    async def send_data(self, data: str, websocket: WebSocket):
        await websocket.send_text(data)

    async def broadcast(self, data: str):
        for connection in self.active_connections:
            await connection.send_text(data)

manager = ConnectionManager()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await manager.connect(websocket)
    try:
        while True:
            # Receiving data from a drone or client
            data = await websocket.receive_text()
            print(f"Received: {data}")
            # Broadcasting the telemetry data to all clients
            await manager.broadcast(data)
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        print("WebSocket connection closed")
```

---

### 2. **Handle Data Streams with Unique Identifiers**
If multiple drones are sending telemetry data, ensure each drone has a unique identifier (e.g., `drone_id`) to distinguish their data. Add logic to route data to specific clients if needed.

**Modified Example for Drone-Specific Data:**
```python
from fastapi import WebSocket
from typing import Dict

class DroneManager:
    def __init__(self):
        self.drone_connections: Dict[str, WebSocket] = {}
        self.ui_connections: List[WebSocket] = []

    async def connect_drone(self, drone_id: str, websocket: WebSocket):
        await websocket.accept()
        self.drone_connections[drone_id] = websocket

    async def connect_ui(self, websocket: WebSocket):
        await websocket.accept()
        self.ui_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        if websocket in self.ui_connections:
            self.ui_connections.remove(websocket)
        else:
            for drone_id, conn in self.drone_connections.items():
                if conn == websocket:
                    del self.drone_connections[drone_id]
                    break

    async def send_to_ui(self, data: str):
        for connection in self.ui_connections:
            await connection.send_text(data)

manager = DroneManager()

@app.websocket("/ws/drone/{drone_id}")
async def drone_endpoint(drone_id: str, websocket: WebSocket):
    await manager.connect_drone(drone_id, websocket)
    try:
        while True:
            data = await websocket.receive_text()
            print(f"Data from {drone_id}: {data}")
            # Forward data to UI clients
            await manager.send_to_ui(f"{drone_id}: {data}")
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        print(f"Drone {drone_id} disconnected")

@app.websocket("/ws/ui")
async def ui_endpoint(websocket: WebSocket):
    await manager.connect_ui(websocket)
    try:
        while True:
            # UI may also send control commands
            command = await websocket.receive_text()
            print(f"Command from UI: {command}")
    except WebSocketDisconnect:
        manager.disconnect(websocket)
        print("UI disconnected")
```

---

### 3. **Improve Performance with Background Tasks**
Telemetry data can be processed in the background to prevent blocking WebSocket communication.

**Example Using FastAPI BackgroundTasks:**
```python
from fastapi import BackgroundTasks

@app.websocket("/ws/drone/{drone_id}")
async def drone_endpoint(drone_id: str, websocket: WebSocket, background_tasks: BackgroundTasks):
    await manager.connect_drone(drone_id, websocket)
    try:
        while True:
            data = await websocket.receive_text()
            background_tasks.add_task(process_telemetry, drone_id, data)
            await manager.send_to_ui(f"{drone_id}: {data}")
    except WebSocketDisconnect:
        manager.disconnect(websocket)

async def process_telemetry(drone_id: str, data: str):
    # Process telemetry data (e.g., save to DB, analyze, etc.)
    print(f"Processing telemetry for {drone_id}: {data}")
```

---

### 4. **Save Telemetry Data to Database**
Use a time-series database (e.g., **TimescaleDB**, **InfluxDB**, or **MongoDB**) to store telemetry data efficiently.

**Saving to MongoDB Example:**
```python
from pymongo import MongoClient

client = MongoClient("mongodb://localhost:27017/")
db = client["drone_data"]
collection = db["telemetry"]

async def save_to_db(drone_id: str, data: str):
    document = {"drone_id": drone_id, "data": data, "timestamp": datetime.now()}
    collection.insert_one(document)
    print(f"Saved telemetry for {drone_id}")
```

Integrate this into the background task:
```python
background_tasks.add_task(save_to_db, drone_id, data)
```

---

### 5. **Broadcast to UI**
If the UI needs real-time updates for multiple drones:
- Use a WebSocket connection dedicated to UI clients.
- Use the `broadcast` method to send data from all drones to the UI.

---

### 6. **Testing & Debugging**
- **Tools**: Use **Postman**, **WebSocket clients**, or custom scripts to test drone and UI connections.
- **Logs**: Enable detailed logging to track WebSocket lifecycle events and data flows.

---

### Additional Considerations:
- **Scaling**: Use tools like **Redis Pub/Sub** or **Kafka** for scaling real-time telemetry data.
- **Security**:
  - Use **JWT tokens** or other authentication methods for WebSocket connections.
  - Secure WebSocket connections with **SSL/TLS** (`wss://`).

This approach ensures that FastAPI can handle multi-drone telemetry data effectively while updating the UI in real time. Let me know if you'd like more details on any part!

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

