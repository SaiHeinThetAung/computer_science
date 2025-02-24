# Multi-GCS Backend Data Flow

## Overview
This flowchart illustrates how multiple Ground Control Stations (GCS) connect to the backend using MAVLink over TCP and UDP, how data is processed, and how telemetry data is handled.

```mermaid
flowchart TD
    A[Multiple GCS Clients] -->|TCP/UDP Connections| B[Backend Server]
    B --> C{Connection Type}
    C -- TCP --> D[Spawn TCP Handler Thread]
    C -- UDP --> E[UDP Listener (Process Packet)]
    D --> F[Identify & Tag Connection]
    E --> F
    F --> G[Parse MAVLink Message]
    G --> H[Extract Telemetry Data]
    H --> I[Update Thread-Safe Data Store]
    I --> J[Forward Data to Telemetry Service]
    J --> K[Data Distribution: Logging / UI / Storage]
