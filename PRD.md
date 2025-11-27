# Product Requirements Document (PRD) - SA Rivers IoT Project

## 1. Introduction
The **SA Rivers IoT Project** is a comprehensive system designed to monitor river water levels in South Africa using LoRaWAN sensors. The system provides real-time data visualization, historical trends, and critical alerts to users via a mobile application.

## 2. Objectives
- **Real-time Monitoring**: Provide up-to-date water level readings from various river locations.
- **Data Visualization**: Display historical data in intuitive graphs to track trends.
- **Alerting System**: Notify users immediately when water levels exceed user-defined thresholds (e.g., flood warnings).
- **Sensor Management**: Allow administrators to easily manage sensor metadata and locations.

## 3. Target Audience
- **Residents**: People living near rivers who need to be aware of potential flooding.
- **Estate Managers**: Managers of estates (like Brettenwood) who need to monitor water resources and safety.
- **General Public**: Anyone interested in river levels in the covered areas.

## 4. Technical Architecture
The system utilizes a hybrid architecture combining LoRaWAN for long-range sensor communication, Node-RED for backend processing, SQL for static metadata, and Firebase for real-time app syncing.

### 4.1 High-Level Data Flow
1.  **Sensor**: LoRaWAN sensor measures water level, battery, and range.
2.  **Network**: Data transmitted via LoRaWAN Gateway to a Network Server (e.g., TTN).
3.  **Ingestion (Node-RED)**:
    - Receives payload from Network Server.
    - Queries **SQL Database** for sensor details (Name, Location) using the device EUI.
    - Pushes enriched data to **Firebase Firestore**.
4.  **Storage & Logic (Firebase)**:
    - **Firestore**: Stores current readings and history.
    - **Cloud Functions**: Monitors new readings, calculates deltas, and triggers FCM notifications if thresholds are breached.
5.  **Presentation (Flutter App)**:
    - Subscribes to Firestore for real-time updates.
    - Receives Push Notifications via FCM.

## 5. Functional Requirements

### 5.1 Mobile Application (Flutter)
- **Authentication**:
    - User Registration and Login (Email/Password).
    - Email Verification.
- **Dashboard**:
    - **List View**: Display all monitored sensors with current status (Level, Battery, Last Updated).
    - **Map View**: Interactive map showing sensor locations with status markers.
- **Sensor Details**:
    - **Graph**: Interactive line chart showing water level history (e.g., last 24h, 7d).
    - **Metadata**: Display sensor name, location, and device details.
- **Settings & Alerts**:
    - User-configurable `delta_threshold` for alerts (per sensor).
    - Push Notification permissions and management.

### 5.2 Backend System (Node-RED)
- **Data Ingestion**: Securely receive MQTT/HTTP payloads from the LoRaWAN Network Server.
- **Data Enrichment**:
    - Maintain a connection to a SQL database.
    - Fetch `name`, `latitude`, `longitude` based on `DevEUI`.
- **Data Forwarding**: Format and write data to Firebase Firestore (`readings` and `sensors` collections).

### 5.3 Admin Dashboard (Node-RED Dashboard)
- **Sensor Management**:
    - **Add/Edit Sensor**: Form to input `DevEUI`, `Name`, `Latitude`, `Longitude`.
    - **List Sensors**: Table view of all configured sensors in the SQL database.
    - **Map Visualization**: Admin map view to verify sensor placement.

### 5.4 Cloud Logic (Firebase Functions)
- **Alert Trigger**:
    - Trigger on creation of new reading documents.
    - Compare current level with previous level.
    - If change > `delta_threshold`, send FCM notification to subscribed users/topics.

## 6. Non-Functional Requirements
- **Reliability**: System must handle network intermittency gracefully.
- **Latency**: End-to-end latency (Sensor to App) should be minimized (typically < 30s).
- **Scalability**: Architecture should support adding new sensors without code changes.
- **Usability**: Mobile app should be intuitive and responsive.

## 7. Data Models

### 7.1 SQL (Sensor Metadata)
Table: `sensors`
- `eui` (Primary Key, VARCHAR)
- `name` (VARCHAR)
- `lat` (FLOAT)
- `lng` (FLOAT)

### 7.2 Firestore (NoSQL)
- **Collection**: `sensors`
    - Document: `{sensor_eui}`
        - `name`, `location`, `lat`, `lng`
        - `last_level`, `delta_threshold`
- **Collection**: `readings`
    - Document: `{sensor_eui}`
        - **Sub-collection**: `history`
            - Document: `{timestamp}`
                - `level`, `battery`, `raw_range`
