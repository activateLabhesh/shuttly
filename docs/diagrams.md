# Shuttly UML Diagrams

This document contains all UML diagrams for the Shuttly smart shuttle tracking and fare enforcement system.

---

## 1. Use Case Diagram

```mermaid
flowchart TB
    subgraph Shuttly System
        subgraph Student Use Cases
            UC1[Register/Login]
            UC2[View Dashboard]
            UC3[Scan QR - Tap In]
            UC4[Scan QR - Tap Out]
            UC5[View Wallet Balance]
            UC6[View Transaction History]
            UC7[View Ride History]
            UC8[Track Shuttle on Map]
        end

        subgraph Driver Use Cases
            UC9[Start Route]
            UC10[Broadcast GPS Location]
            UC11[End Route]
            UC12[View Assigned Shuttle]
        end

        subgraph Admin Use Cases
            UC13[Manage Routes - CRUD]
            UC14[Manage Stops - CRUD]
            UC15[Configure Fare Matrix]
            UC16[Manage Shuttles - CRUD]
            UC17[Assign Driver to Shuttle]
            UC18[Manage Users]
            UC19[Top-up User Wallet]
            UC20[View Ride Logs]
            UC21[View Analytics Dashboard]
            UC22[Generate QR Codes]
        end

        subgraph System Use Cases
            UC23[Calculate Fare]
            UC24[Deduct Wallet Balance]
            UC25[Detect Incomplete Rides]
            UC26[Apply Penalty]
            UC27[Record Transaction]
        end
    end

    Student((Student))
    Driver((Driver))
    Admin((Admin))
    System((System))

    Student --> UC1
    Student --> UC2
    Student --> UC3
    Student --> UC4
    Student --> UC5
    Student --> UC6
    Student --> UC7
    Student --> UC8

    Driver --> UC1
    Driver --> UC9
    Driver --> UC10
    Driver --> UC11
    Driver --> UC12

    Admin --> UC1
    Admin --> UC13
    Admin --> UC14
    Admin --> UC15
    Admin --> UC16
    Admin --> UC17
    Admin --> UC18
    Admin --> UC19
    Admin --> UC20
    Admin --> UC21
    Admin --> UC22

    System --> UC23
    System --> UC24
    System --> UC25
    System --> UC26
    System --> UC27

    UC4 -.->|includes| UC23
    UC23 -.->|includes| UC24
    UC24 -.->|includes| UC27
    UC25 -.->|includes| UC26
    UC26 -.->|includes| UC27
```

### Use Case Descriptions

| ID | Use Case | Actor | Description |
|----|----------|-------|-------------|
| UC1 | Register/Login | All | User authentication with email/password |
| UC2 | View Dashboard | Student | See active shuttles, wallet balance, active ride |
| UC3 | Scan QR - Tap In | Student | Scan stop QR code to start a ride |
| UC4 | Scan QR - Tap Out | Student | Scan stop QR code to end a ride |
| UC5 | View Wallet Balance | Student | Check current wallet balance |
| UC6 | View Transaction History | Student | View all wallet transactions |
| UC7 | View Ride History | Student | View past rides with fares |
| UC8 | Track Shuttle on Map | Student | Real-time shuttle location tracking |
| UC9 | Start Route | Driver | Begin GPS broadcasting for assigned route |
| UC10 | Broadcast GPS Location | Driver | Send location updates every 3 seconds |
| UC11 | End Route | Driver | Stop GPS broadcasting |
| UC12 | View Assigned Shuttle | Driver | See shuttle and route assignment |
| UC13 | Manage Routes | Admin | Create, read, update, delete routes |
| UC14 | Manage Stops | Admin | CRUD stops with map coordinates |
| UC15 | Configure Fare Matrix | Admin | Set fares between stops |
| UC16 | Manage Shuttles | Admin | CRUD shuttle vehicles |
| UC17 | Assign Driver to Shuttle | Admin | Link drivers to shuttles |
| UC18 | Manage Users | Admin | View and edit user roles |
| UC19 | Top-up User Wallet | Admin | Add funds to user wallets |
| UC20 | View Ride Logs | Admin | See all rides with filters |
| UC21 | View Analytics Dashboard | Admin | System overview statistics |
| UC22 | Generate QR Codes | Admin | Create QR codes for stops |
| UC23 | Calculate Fare | System | Compute fare based on entry/exit stops |
| UC24 | Deduct Wallet Balance | System | Subtract fare from wallet |
| UC25 | Detect Incomplete Rides | System | Find rides without tap-out after timeout |
| UC26 | Apply Penalty | System | Charge penalty for incomplete rides |
| UC27 | Record Transaction | System | Log all wallet transactions |

---

## 2. Sequence Diagrams

### 2.1 Student Tap-In Flow

```mermaid
sequenceDiagram
    autonumber
    actor Student
    participant Scanner as QR Scanner
    participant API as Tap-In API
    participant DB as Database
    participant Wallet as Wallet Service

    Student->>Scanner: Open QR Scanner
    Scanner->>Student: Camera activated
    Student->>Scanner: Scan stop QR code
    Scanner->>API: POST /api/rides/tap-in<br/>{stopQrCode, shuttleId}
    
    API->>DB: Validate stop QR code
    DB-->>API: Stop details (id, routeId)
    
    API->>DB: Check user has active ride
    DB-->>API: No active ride
    
    API->>Wallet: Check wallet balance
    Wallet-->>API: Balance: ₹150
    
    alt Balance >= Minimum Fare
        API->>DB: Create ride record<br/>(status: active, entryStopId, entryTime)
        DB-->>API: Ride created (id)
        API-->>Scanner: Success: Ride started
        Scanner-->>Student: ✓ Tap-in successful<br/>Entry: Stop A
    else Insufficient Balance
        API-->>Scanner: Error: Insufficient balance
        Scanner-->>Student: ✗ Please top-up wallet
    end
```

### 2.2 Student Tap-Out Flow

```mermaid
sequenceDiagram
    autonumber
    actor Student
    participant Scanner as QR Scanner
    participant API as Tap-Out API
    participant FareService as Fare Service
    participant DB as Database
    participant Wallet as Wallet Service

    Student->>Scanner: Open QR Scanner
    Scanner->>Student: Camera activated
    Student->>Scanner: Scan exit stop QR code
    Scanner->>API: POST /api/rides/tap-out<br/>{stopQrCode}
    
    API->>DB: Get user's active ride
    DB-->>API: Ride (entryStopId, routeId)
    
    API->>DB: Validate exit stop QR
    DB-->>API: Exit stop details
    
    API->>FareService: calculateFare(routeId, entryStopId, exitStopId)
    
    FareService->>DB: Get fare from fare_matrix
    alt Fare exists in matrix
        DB-->>FareService: Fare: ₹25
    else Calculate from distance
        FareService->>DB: Get stops sequence
        DB-->>FareService: Stops list
        FareService->>FareService: Calculate stops traveled<br/>(handle circular route)
        Note over FareService: fare = BASE + (stops × PER_STOP)
    end
    FareService-->>API: Fare: ₹25
    
    API->>Wallet: Deduct fare (₹25)
    Wallet->>DB: Update wallet balance
    Wallet->>DB: Create transaction record
    DB-->>Wallet: Transaction created
    Wallet-->>API: Deduction successful
    
    API->>DB: Update ride<br/>(exitStopId, exitTime, fare, status: completed)
    DB-->>API: Ride updated
    
    API-->>Scanner: Success: Ride completed, Fare: ₹25
    Scanner-->>Student: ✓ Tap-out successful<br/>Fare deducted: ₹25
```

### 2.3 Driver GPS Broadcasting Flow

```mermaid
sequenceDiagram
    autonumber
    actor Driver
    participant App as Driver App
    participant GeoAPI as Geolocation API
    participant GPS_API as GPS Update API
    participant DB as Database
    participant SSE as SSE Stream
    participant StudentApp as Student Apps

    Driver->>App: Start Route
    App->>DB: Get assigned shuttle & route
    DB-->>App: Shuttle & Route details
    
    App->>App: Set route as active
    
    loop Every 3 seconds
        App->>GeoAPI: getCurrentPosition()
        GeoAPI-->>App: {lat, lng, heading, speed}
        
        App->>GPS_API: POST /api/gps/update<br/>{shuttleId, lat, lng, heading, speed}
        
        GPS_API->>DB: Insert shuttle_location record
        DB-->>GPS_API: Location saved
        
        GPS_API->>SSE: Broadcast new position
        SSE-->>StudentApp: SSE Event: position update
        
        GPS_API-->>App: Position updated
    end
    
    Driver->>App: End Route
    App->>GPS_API: POST /api/gps/update<br/>{shuttleId, status: inactive}
    GPS_API->>DB: Update shuttle status
    GPS_API->>SSE: Broadcast shuttle inactive
    SSE-->>StudentApp: SSE Event: shuttle offline
```

### 2.4 Incomplete Ride Penalty Flow

```mermaid
sequenceDiagram
    autonumber
    participant Cron as Cron Job / Scheduler
    participant PenaltyService as Penalty Service
    participant DB as Database
    participant Wallet as Wallet Service

    Note over Cron: Runs every 15 minutes
    
    Cron->>PenaltyService: detectIncompleteRides()
    
    PenaltyService->>DB: Find rides WHERE<br/>status = 'active' AND<br/>entryTime < NOW() - 90 minutes
    DB-->>PenaltyService: List of incomplete rides
    
    loop For each incomplete ride
        PenaltyService->>DB: Begin Transaction
        
        PenaltyService->>DB: Update ride<br/>(status: penalty, penaltyApplied: true)
        
        PenaltyService->>DB: Get user wallet
        DB-->>PenaltyService: Wallet (id, balance)
        
        PenaltyService->>Wallet: Deduct penalty (₹50)
        Wallet->>DB: Update wallet balance
        Wallet->>DB: Create transaction<br/>(type: penalty, amount: 50)
        
        PenaltyService->>DB: Commit Transaction
        
        Note over PenaltyService: Send notification to user
    end
    
    PenaltyService-->>Cron: Processed X incomplete rides
```

### 2.5 Admin Wallet Top-up Flow

```mermaid
sequenceDiagram
    autonumber
    actor Admin
    participant UI as Admin Dashboard
    participant API as Wallet API
    participant DB as Database

    Admin->>UI: Navigate to User Management
    UI->>API: GET /api/users
    API->>DB: Fetch users with wallet info
    DB-->>API: Users list
    API-->>UI: Display users
    
    Admin->>UI: Select user, enter top-up amount
    Admin->>UI: Click "Top-up"
    
    UI->>API: POST /api/wallet/topup<br/>{userId, amount: 100}
    
    API->>API: Validate admin role
    API->>DB: Get user wallet
    DB-->>API: Wallet (id, balance: 50)
    
    API->>DB: Begin Transaction
    API->>DB: Update wallet balance<br/>(balance = 50 + 100 = 150)
    API->>DB: Create transaction record<br/>(type: credit, amount: 100)
    API->>DB: Commit Transaction
    
    DB-->>API: Success
    API-->>UI: Top-up successful, new balance: ₹150
    UI-->>Admin: ✓ Wallet topped up
```

---

## 3. Class Diagram

```mermaid
classDiagram
    class User {
        +String id
        +String email
        +String name
        +String passwordHash
        +String role
        +DateTime createdAt
        +DateTime updatedAt
        +login()
        +logout()
        +register()
    }

    class Wallet {
        +UUID id
        +String userId
        +Decimal balance
        +DateTime updatedAt
        +getBalance()
        +credit(amount)
        +debit(amount)
    }

    class Transaction {
        +UUID id
        +UUID walletId
        +String type
        +Decimal amount
        +String description
        +UUID rideId
        +DateTime createdAt
    }

    class Shuttle {
        +UUID id
        +String name
        +String plateNumber
        +Integer capacity
        +String status
        +UUID currentRouteId
        +String currentDriverId
        +DateTime createdAt
        +activate()
        +deactivate()
        +assignDriver(driverId)
        +assignRoute(routeId)
    }

    class Route {
        +UUID id
        +String name
        +String description
        +Boolean isActive
        +DateTime createdAt
        +activate()
        +deactivate()
        +getStops()
    }

    class Stop {
        +UUID id
        +UUID routeId
        +String name
        +Double latitude
        +Double longitude
        +Integer sequenceOrder
        +String qrCode
        +DateTime createdAt
        +generateQrCode()
        +validateQrCode(code)
    }

    class FareMatrix {
        +UUID id
        +UUID routeId
        +UUID fromStopId
        +UUID toStopId
        +Decimal fare
        +getFare(from, to)
    }

    class Ride {
        +UUID id
        +String userId
        +UUID shuttleId
        +UUID routeId
        +UUID entryStopId
        +UUID exitStopId
        +DateTime entryTime
        +DateTime exitTime
        +Decimal fare
        +String status
        +Boolean penaltyApplied
        +DateTime createdAt
        +tapIn(stopId)
        +tapOut(stopId)
        +calculateFare()
        +applyPenalty()
    }

    class ShuttleLocation {
        +UUID id
        +UUID shuttleId
        +Double latitude
        +Double longitude
        +Double heading
        +Double speed
        +DateTime recordedAt
        +updatePosition(lat, lng)
        +getLatestPosition()
    }

    class Session {
        +String id
        +String userId
        +String token
        +DateTime expiresAt
        +validate()
        +refresh()
        +revoke()
    }

    %% Relationships
    User "1" --> "1" Wallet : has
    User "1" --> "*" Ride : takes
    User "1" --> "*" Session : has
    User "1" --> "*" Shuttle : drives

    Wallet "1" --> "*" Transaction : has

    Shuttle "1" --> "1" Route : assigned to
    Shuttle "1" --> "*" ShuttleLocation : has
    Shuttle "1" --> "*" Ride : serves

    Route "1" --> "*" Stop : contains
    Route "1" --> "*" FareMatrix : has
    Route "1" --> "*" Ride : used in

    Stop "1" --> "*" FareMatrix : from
    Stop "1" --> "*" FareMatrix : to
    Stop "1" --> "*" Ride : entry point
    Stop "1" --> "*" Ride : exit point

    Transaction "*" --> "0..1" Ride : related to

    %% Enumerations
    class UserRole {
        <<enumeration>>
        STUDENT
        DRIVER
        ADMIN
    }

    class ShuttleStatus {
        <<enumeration>>
        ACTIVE
        INACTIVE
        MAINTENANCE
    }

    class RideStatus {
        <<enumeration>>
        ACTIVE
        COMPLETED
        PENALTY
    }

    class TransactionType {
        <<enumeration>>
        CREDIT
        DEBIT
        PENALTY
    }
```

### Class Descriptions

| Class | Description |
|-------|-------------|
| **User** | System user with role-based access (student, driver, admin) |
| **Wallet** | User's digital wallet for fare payments |
| **Transaction** | Record of wallet credits, debits, and penalties |
| **Shuttle** | Physical shuttle vehicle with capacity and status |
| **Route** | Circular shuttle route containing ordered stops |
| **Stop** | Physical stop location with QR code for tap-in/out |
| **FareMatrix** | Fare configuration between two stops on a route |
| **Ride** | Record of a student's journey from entry to exit |
| **ShuttleLocation** | GPS position record for real-time tracking |
| **Session** | User authentication session managed by Better Auth |

---

## 4. State Diagrams

### 4.1 Ride State Diagram

```mermaid
stateDiagram-v2
    [*] --> Idle: User opens scanner

    Idle --> Active: Tap-In (scan entry QR)
    
    Active --> Completed: Tap-Out (scan exit QR)
    Active --> Penalty: Timeout (90 min without tap-out)
    
    Completed --> [*]: Fare deducted
    Penalty --> [*]: Penalty applied
    
    note right of Active
        - Entry stop recorded
        - Entry time recorded
        - Waiting for tap-out
    end note
    
    note right of Completed
        - Exit stop recorded
        - Exit time recorded
        - Fare calculated & deducted
    end note
    
    note right of Penalty
        - No exit recorded
        - Penalty amount deducted
        - Marked as incomplete
    end note
```

### 4.2 Shuttle State Diagram

```mermaid
stateDiagram-v2
    [*] --> Inactive: Shuttle created

    Inactive --> Active: Driver starts route
    Active --> Inactive: Driver ends route
    Inactive --> Maintenance: Admin marks for maintenance
    Maintenance --> Inactive: Maintenance completed
    Active --> Maintenance: Emergency maintenance
    
    state Active {
        [*] --> Broadcasting
        Broadcasting --> Broadcasting: GPS update (every 3s)
        Broadcasting --> Stopped: Shuttle stops
        Stopped --> Broadcasting: Shuttle moves
    }
    
    note right of Inactive
        - Not broadcasting GPS
        - Not visible on student map
        - Available for assignment
    end note
    
    note right of Active
        - Broadcasting GPS
        - Visible on student map
        - Accepting passengers
    end note
    
    note right of Maintenance
        - Not available for routes
        - Not visible to students
        - Admin only access
    end note
```

### 4.3 Wallet Transaction State Diagram

```mermaid
stateDiagram-v2
    [*] --> Idle: Wallet created (balance: 0)

    Idle --> Processing: Transaction initiated
    
    state Processing {
        [*] --> Validating
        Validating --> Executing: Validation passed
        Validating --> Failed: Validation failed
        Executing --> Recording: Balance updated
        Recording --> Success: Transaction logged
    }
    
    Processing --> Credited: Top-up successful
    Processing --> Debited: Fare/penalty deducted
    Processing --> Failed: Insufficient funds / Error
    
    Credited --> Idle: Ready for next transaction
    Debited --> Idle: Ready for next transaction
    Failed --> Idle: No changes made
    
    note right of Credited
        Transaction types:
        - Admin top-up
        - Refund
    end note
    
    note right of Debited
        Transaction types:
        - Ride fare
        - Penalty charge
    end note
```

### 4.4 User Authentication State Diagram

```mermaid
stateDiagram-v2
    [*] --> Guest: User visits app

    Guest --> Registering: Click Register
    Registering --> Guest: Registration failed
    Registering --> LoggedIn: Registration successful
    
    Guest --> Authenticating: Click Login
    Authenticating --> Guest: Login failed
    Authenticating --> LoggedIn: Login successful
    
    LoggedIn --> Guest: Logout / Session expired
    
    state LoggedIn {
        [*] --> StudentDashboard: role = student
        [*] --> DriverDashboard: role = driver
        [*] --> AdminDashboard: role = admin
        
        StudentDashboard --> Scanning: Open scanner
        Scanning --> StudentDashboard: Scan complete/cancel
        
        DriverDashboard --> Broadcasting: Start route
        Broadcasting --> DriverDashboard: End route
    }
    
    note right of Guest
        - Can view landing page
        - No access to dashboard
    end note
    
    note right of LoggedIn
        - Session token valid
        - Role-based routing
        - Access to features
    end note
```

### 4.5 Route Management State Diagram

```mermaid
stateDiagram-v2
    [*] --> Draft: Route created

    Draft --> Active: Admin activates
    Draft --> Draft: Add/edit stops
    Draft --> Draft: Configure fares
    
    Active --> Inactive: Admin deactivates
    Active --> Active: Update stops/fares
    
    Inactive --> Active: Admin reactivates
    Inactive --> Deleted: Admin deletes
    
    Deleted --> [*]: Cascade delete stops, fares
    
    note right of Draft
        - Stops can be added
        - Fares can be configured
        - Not visible to students
        - Cannot assign shuttles
    end note
    
    note right of Active
        - Visible to students
        - Shuttles can be assigned
        - Students can ride
        - QR codes generated
    end note
    
    note right of Inactive
        - Hidden from students
        - Existing data preserved
        - Can be reactivated
    end note
```

---

## Summary

| Diagram Type | Count | Purpose |
|--------------|-------|---------|
| Use Case | 1 | Shows all actors and their interactions with the system |
| Sequence | 5 | Details the flow of key operations (tap-in, tap-out, GPS, penalty, top-up) |
| Class | 1 | Defines all entities, attributes, methods, and relationships |
| State | 5 | Shows lifecycle states for Ride, Shuttle, Wallet, User Auth, and Route |

These diagrams provide a complete blueprint for implementing the Shuttly system.
