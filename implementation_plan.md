# Implementation Plan - SA Rivers IoT

## Goal Description
Develop a comprehensive IoT river monitoring system. This includes a Node-RED backend for processing LoRaWAN data, a SQL database for sensor metadata, Firebase for real-time data storage and authentication, and a Flutter mobile app for visualization and alerts.

## User Review Required
> [!IMPORTANT]
> **Firebase Setup**: The user needs to provide the `google-services.json` (Android) and `GoogleService-Info.plist` (iOS) files.
> **Cloud Functions**: Requires Firebase **Blaze (Pay-as-you-go)** plan.
> **SQL Database**: Connection details for the SQL database are required for Node-RED to fetch sensor locations.

## Data Flow
The system follows a linear data flow from the physical environment to the user's device:
1.  **Sensor**: A LoRaWAN sensor mounted over the river measures water level, battery voltage, and range.
2.  **Gateway**: The sensor transmits encrypted data packets via LoRa radio to a nearby LoRaWAN Gateway.
3.  **LNS (LoRa Network Server)**: The gateway forwards the packet to the LNS (e.g., The Things Network).
4.  **Application Server (Node-RED)**:
    - The LNS pushes the payload to Node-RED.
    - Node-RED parses the payload and queries the **SQL Database** for metadata (Location, Name).
    - Node-RED pushes the enriched data to **Firebase Firestore**.
5.  **Firebase (Logic & Storage)**:
    - **Firestore**: Stores the new reading.
    - **Cloud Functions**: Triggers on the new reading, checks the alert threshold configured in Firestore, and sends a notification via FCM if the level change is significant.
6.  **App**: The Flutter application receives the FCM notification and updates the dashboard via Firestore subscription.

## Proposed Changes

### Backend (Node-RED & SQL)

#### [NEW] SQL Schema (`back_end/schema.sql`)
- Table `sensors`:
    - `eui` (Primary Key)
    - `name` (varchar)
    - `lat` (float)
    - `lng` (float)
    *(Alert config moved to Firebase)*

#### [NEW] Node-RED Flows (`back_end/flows.json`)
- **Flow 1: Ingest & Enrich**
    - Input: LoRaWAN Uplink (MQTT/HTTP)
    - Process:
        - Extract `bat`, `range`, `interval`.
        - Query SQL `sensors` table using EUI to get `lat`, `lng`, `name`.
    - Output: Push enriched object to Firebase Firestore collection `readings`.

#### [NEW] Node-RED Admin Flows (`back_end/admin_flows.json`)
- **Flow 1: Database Setup**
    - **Trigger**: Manual Inject "Init DB".
    - **Action**: Execute SQL `CREATE TABLE IF NOT EXISTS sensor_locations ...`.
- **Flow 2: Sensor Management UI**
    - **UI Form**: Inputs for `DevEUI`, `Name`, `Latitude`, `Longitude`.
    - **Action**: `INSERT INTO sensor_locations ... ON DUPLICATE KEY UPDATE ...`.
    - **UI Table**: Displays `SELECT * FROM sensor_locations`.
    - **UI Map**: Displays markers for all sensors on a world map centered on South Africa.
    - **Refresh**: Auto-refresh table and map after Insert/Update.

### Firebase Structure & Logic

#### [NEW] Firestore Schema
- `sensors/{sensor_eui}`:
    - `name`, `location`, `lat`, `lng` (Synced/Static)
    - `delta_threshold` (float): Configurable change threshold for alerts.
    - `last_level` (float): Tracked for delta calculation.
- `readings/{sensor_eui}/history/{timestamp}`: Individual readings.

#### [NEW] Cloud Functions (`firebase/functions/index.js`)
- **Trigger**: `onCreate` of `readings/{sensor_eui}/history/{documentId}`
- **Logic**:
    1. Fetch `delta_threshold` and `last_level` from `sensors/{sensor_eui}`.
    2. Compare `current_level` (from new doc) with `last_level`.
    3. If `abs(current_level - last_level) >= delta_threshold`:
        - Send FCM Notification to topic `sensor_{sensor_eui}`.
    4. Update `last_level` in `sensors/{sensor_eui}`.

### Flutter App (`app/`)

#### [NEW] Dependencies
- `firebase_core`, `firebase_auth`, `cloud_firestore`, `firebase_messaging`
- `flutter_signals` (State Management)
- `go_router` (Navigation)
- `flutter_map` (OpenStreetMap)
- `fl_chart` (Graphs)

#### [NEW] Authentication (`lib/features/auth/`)
- `AuthService`: Handle Sign In, Sign Up, Sign Out, Email Verification.
- UI: `LoginScreen`, `RegisterScreen`, `VerifyEmailScreen`.

#### [NEW] Dashboard (`lib/features/dashboard/`)
- `DashboardScreen`:
    - **List View**: Scrollable list of sensors with current status.
    - **Map View**: `flutter_map` displaying markers.
    - **Logic**: Subscribe to Firestore `sensors` collection.

#### [NEW] Sensor Details (`lib/features/details/`)
- `DetailsScreen`:
    - **Graph**: `LineChart` showing water level.
    - **Settings**: Input to update `delta_threshold` in Firestore.

#### [NEW] Push Notifications
- Initialize `FirebaseMessaging`.
- Request permissions.
- Handle foreground and background messages.

### Documentation
- [x] Update `README.md` with sensor image and Auth details.
- [x] Update `task.md` with Auth tasks.

## Verification Plan

### Automated Tests
- Widget tests for Login/Register screens.

### Manual Verification
1.  **Auth**: Register, Verify Email, Login.
2.  **Data Flow**: Simulate LoRaWAN payload -> Node-RED -> Firestore.
3.  **Alerts**:
    - Set `delta_threshold` in Firestore.
    - Push a reading with a large change via Node-RED.
    - Verify Cloud Function execution in Firebase Console.
    - Verify Notification received on device.
