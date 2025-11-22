# Task List - SA Rivers IoT Project

- [x] **Project Initialization**
    - [x] Create project planning artifacts (`task.md`, `implementation_plan.md`) <!-- id: 0 -->
    - [x] Verify existing project structure (`app` and `back_end`) <!-- id: 1 -->
    - [x] Create project `README.md` <!-- id: 1.5 -->
    - [x] Initialize Git repository and set remote <!-- id: 1.6 -->
    - [x] Generate App Logo/Icon <!-- id: 1.7 -->

- [ ] **Backend Setup (Node-RED & Firebase)**
    - [x] Define Firebase Data Schema <!-- id: 2 -->
    - [ ] Setup Firebase Project (using MCP or manual) <!-- id: 3 -->
    - [x] Design Node-RED Flow Logic (Documented) <!-- id: 4 -->
        - [ ] Input: LoRaWAN Uplink <!-- id: 5 -->
        - [ ] Process: Lookup Location via EUI (SQL) <!-- id: 6 -->
        - [ ] Output: Push to Firebase <!-- id: 7 -->

- [ ] **Flutter App Development**
    - [ ] Initialize Flutter App in `app` directory <!-- id: 8 -->
    - [ ] Add Dependencies (`firebase_core`, `cloud_firestore`, `flutter_signals`, `go_router`, `flutter_map`, `fl_chart`) <!-- id: 9 -->
    - [ ] Create Basic Layout & Navigation <!-- id: 10 -->
    - [ ] Implement Firebase Data Integration <!-- id: 11 -->
    - [ ] Create Dashboard UI (List & Map View) <!-- id: 12 -->
    - [ ] Create Sensor Details UI (Graph & Time Selection) <!-- id: 14 -->

- [ ] **Verification**
    - [ ] Verify Data Flow (Mock Node-RED push -> Firebase -> App) <!-- id: 13 -->
