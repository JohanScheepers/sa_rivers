# Implementation Plan - SA Rivers IoT

This project aims to build a system for monitoring river levels using LoRaWAN sensors. Data flows from sensors to Node-RED, is enriched with location data from a SQL database, stored in Firebase, and visualized in a Flutter mobile app.

## User Review Required
> [!IMPORTANT]
> **Node-RED & SQL**: I will provide the *logic* and *schema* for the Node-RED flow and SQL table, but I cannot execute Node-RED flows directly. You will need to implement the flow in your Node-RED instance based on the design provided.
> **Firebase**: I will attempt to set up the Firebase project using the Firebase MCP. If that fails or requires specific permissions, I will need your assistance.

## Proposed Architecture

### Data Flow
1.  **Sensor (LoRaWAN)**: Sends `{ field1: bat, field2: range, field3: interval }`.
2.  **Node-RED**:
    *   Receives payload.
    *   Extracts `eui` (Sensor ID).
    *   Queries SQL DB: `SELECT lat, lng, name FROM sensor_locations WHERE eui = ?`.
    *   Constructs final object:
        ```json
        {
          "eui": "...",
          "battery": field1,
          "level": field2, // range
          "interval": field3,
          "location": { "lat": ..., "lng": ... },
          "name": "...",
          "timestamp": <server_timestamp>
        }
        ```
    *   Pushes to Firebase Firestore collection `readings` (or similar).

### Component: Backend (Node-RED & SQL)
#### [NEW] `back_end/node_red_flow.json`
*   A JSON export or description of the Node-RED flow.
*   **Logic**:
    *   Input Node: MQTT/HTTP (depending on LoRa server).
    *   Function Node: Parse payload.
    *   SQL Node: Query location.
    *   Firebase Node: Write to Firestore.

#### [NEW] `back_end/sql_schema.sql`
*   Schema for the `sensor_locations` table.

### Component: Flutter App (`app/`)
#### [NEW] `app/lib/main.dart`
*   Entry point.
*   Setup `flutter_signals` and `go_router`.
*   Initialize Firebase.
*   **Dependencies**: `firebase_core`, `cloud_firestore`, `flutter_signals`, `go_router`, `flutter_map`, `latlong2`, `fl_chart`, `intl`.

#### [NEW] `app/lib/features/dashboard/`
*   **Screens**:
    *   `DashboardScreen`: Toggle between List and Map view.
    *   `MapScreen`: Displays sensors as markers on a map (using `flutter_map`). Tapping a marker shows a summary popup (Last Reading).
    *   `SensorDetailsScreen`: Displays current level, battery, and a historical graph.
*   **Widgets**:
    *   `SensorCard`: Summary card for list view.
    *   `HistoryGraph`: Line chart (using `fl_chart`) showing data over selected periods (24h, 7d, 30d, etc.).
    *   `TimePeriodSelector`: Dropdown/Segmented control for graph intervals.
*   **Logic**:
    *   `SensorSignal`: Listen to Firestore stream.
    *   `HistorySignal`: Fetch historical data based on selected time period.

## Verification Plan

### Automated Tests
*   **Flutter**: Widget tests for `SensorCard` to ensure it displays data correctly.

### Manual Verification
1.  **Mock Data Push**: Manually insert a document into Firestore (simulating Node-RED) and verify it appears in the Flutter app.
2.  **App UI**: Check that the dashboard updates in real-time when data changes in Firebase.
