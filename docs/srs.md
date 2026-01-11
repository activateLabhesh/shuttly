Software Requirements

Specification

For

Smart Shuttle Tracking & Payment System

Version 1.0 approved

Prepared by 

Labhesh Kumar(24BIT0051)

Prableen Kaur Jolly(24BIT0047)

Lakshay Balani(24BIT0436)

2nd February 2026

Software Requirements Specification for Smart Shuttle Tracking & Payment System


Table of Contents

Table of Contents ii

Revision History ii

Introduction 1

Purpose 1

Document Conventions 1

Intended Audience and Reading Suggestions 1

Product Scope 1

References 1

Overall Description 2

Product Perspective 2

Product Functions 2

User Classes and Characteristics 2

Operating Environment 2

Design and Implementation Constraints 2

User Documentation 2

Assumptions and Dependencies 3

External Interface Requirements 3

User Interfaces 3

Hardware Interfaces 3

Software Interfaces 3

Communications Interfaces 3

System Features 4

System Feature 1 4

System Feature 2 (and so on) 4

Other Nonfunctional Requirements 4

Performance Requirements 4

Safety Requirements 5

Security Requirements 5

Software Quality Attributes 5

Business Rules 5

Other Requirements 5

Appendix A: Glossary 5

Appendix B: Analysis Models 5

Appendix C: To Be Determined List 6

Revision History

Name

Date

Reason For Changes

Version

Software Requirements Specification for Smart Shuttle Tracking & Payment System

Page 1


Introduction

Purpose

The purpose of this document is to specify the software requirements for the Smart Shuttle Tracking & Payment System (SSTPS). It provides a detailed description of the system’s intended functionality, scope, and constraints. It serves as a reference for developers, project evaluators, and stakeholders to understand what the system is expected to do before implementation begins. The SRS ensures that all functional and non-functional requirements are clearly defined and agreed upon prior to system design and development.

Document Conventions

This document follows standard Software Engineering documentation practices:

- **System** refers to the Smart Shuttle Tracking & Payment System (SSTPS).
- **Student** refers to individuals using the campus shuttle service for commuting.
- **Driver** refers to shuttle operators responsible for driving shuttles and broadcasting GPS location.
- **Admin** refers to authorized personnel responsible for managing shuttle operations, routes, and fares.
- All requirements are written in clear, unambiguous language.
- Priority levels (High, Medium, Low) indicate implementation importance.

Intended Audience and Reading Suggestions

This document is intended for:

Course instructors and evaluators assessing the project.

Student developers responsible for designing and implementing the system.

University transport administrators reviewing system feasibility.

Readers interested in understanding what the system does should read Sections 1 and 2 first.
Readers interested in system functionality should focus on Section 4 in later phases.

Product Scope

The Smart Shuttle Tracking & Payment System is designed to digitalize campus shuttle operations by automating fare collection and preventing unpaid travel. The system allows students to scan a QR code when boarding and exiting a shuttle, enabling accurate fare calculation based on the distance traveled along a circular route. It also introduces automatic penalty enforcement for incomplete rides, thereby eliminating the need for manual supervision. The system provides transparency, accountability, and operational efficiency for both users and administrators.

References

Software Engineering course material.

IEEE Software Requirements Specification (SRS) guidelines.

Campus transportation operational policies (assumed)

Overall Description

Product Perspective

SSTPS is a standalone system designed to integrate with existing campus shuttle services. It does not replace the physical shuttle infrastructure but enhances it through digital tracking, payment automation, and monitoring. The system operates as a centralized platform that connects student users, shuttle movement data, and administrative controls into a single unified solution.

Product Functions

At a high level, the system enables students to register, initiate shuttle rides using QR scans, track their ride status, and automatically pay fares through a digital wallet. The system monitors shuttle movement along predefined circular routes and ensures that every ride is either properly completed or penalized. Administrators are provided tools to configure routes, manage fare structures, monitor shuttles in real time, and review ride and payment logs.

User Classes and Characteristics

The system serves three distinct user classes:

**Students** are the primary users of the system. They are expected to have basic smartphone literacy and access to the campus shuttle service. Students interact with the system through a web application to scan QR codes when boarding and exiting shuttles, view their ride history, check wallet balance, and track shuttles in real time on a map.

**Drivers** operate the campus shuttles and are responsible for broadcasting their location during active routes. Drivers use a dedicated interface to start and end their shifts, which triggers GPS location sharing. They need basic familiarity with smartphones and the ability to operate the driver dashboard while managing shuttle operations.

**Administrators** oversee the entire shuttle system. They configure routes, manage stops, set fare matrices, monitor shuttle locations in real time, and handle user management tasks like wallet top-ups. Admins are expected to have moderate technical knowledge and access the system through a web-based dashboard.

Operating Environment

The system operates in a client-server architecture designed for web browsers:

- **Student & Driver Interface**: A progressive web application (PWA) built with Next.js, accessible through any modern web browser on smartphones, tablets, or desktops. The PWA can be installed on mobile devices for an app-like experience.
- **Admin Interface**: A web-based dashboard optimized for desktop browsers, providing comprehensive management tools.
- **Backend**: A Node.js server handling API requests, fare calculations, and real-time GPS streaming.
- **Database**: Cloud-hosted PostgreSQL database (Neon) for storing user data, rides, transactions, and route configurations.
- **Real-time Communication**: Server-Sent Events (SSE) for pushing live shuttle locations to connected clients.

Internet connectivity is required for all system operations. GPS services on user devices enable location tracking for drivers.

Design and Implementation Constraints

Several constraints shape how the system must be designed and built:

- **Circular Route Assumption**: All shuttle routes are circular, meaning shuttles continuously loop through a fixed sequence of stops. Fare calculation logic depends on this structure.
- **QR Code Only**: The system uses QR codes exclusively for tap-in and tap-out. No RFID or NFC hardware is required, making the system accessible to any user with a smartphone camera.
- **GPS Dependency**: Real-time shuttle tracking relies on GPS accuracy from driver devices. Areas with poor GPS signal may experience tracking delays.
- **Network Dependency**: All operations require internet connectivity. Offline functionality is not supported in the initial version.
- **Browser Compatibility**: The web application targets modern browsers (Chrome, Safari, Firefox, Edge) with support for camera access and geolocation APIs.
- **Penalty Timeout**: Incomplete rides (no tap-out) are penalized after 90 minutes, assuming the shuttle has completed at least one full route cycle.

User Documentation

The system will include the following documentation to help users get started:

- **In-App Guides**: Contextual tooltips and help text within the application explaining key features like QR scanning, wallet management, and ride tracking.
- **Student Quick Start Guide**: A brief walkthrough covering registration, first ride, and checking ride history.
- **Driver Guide**: Instructions for starting a route, enabling GPS broadcasting, and ending a shift.
- **Admin Manual**: Comprehensive documentation covering route configuration, fare matrix setup, user management, and monitoring features.

Documentation will prioritize clarity and simplicity over technical depth.

Assumptions and Dependencies

The system is built on the following assumptions:

- Students and drivers carry smartphones with a working camera (for QR scanning) and GPS capability.
- Shuttles follow predefined circular routes without significant deviation.
- Campus has adequate cellular or WiFi coverage for internet connectivity.
- Users have valid university credentials for registration and authentication.
- Administrators will configure routes and fare matrices before the system goes live.

The system depends on:

- **Google Maps API**: For rendering shuttle locations on interactive maps.
- **Neon PostgreSQL**: Cloud database service for persistent data storage.
- **Better Auth**: Authentication library for secure user login and session management.
- **Device GPS**: For driver location broadcasting (accuracy of ~10 meters expected).
- **Device Camera**: For QR code scanning via the html5-qrcode library.

External Interface Requirements

User Interfaces

The system provides three distinct user interfaces tailored to each user class:

**Student Dashboard**

The student interface is designed for mobile-first usage with a clean, intuitive layout:

- **Home Screen**: Displays current wallet balance prominently, active ride status (if any), and quick-access buttons for scanning QR codes and viewing the shuttle map.
- **QR Scanner**: Full-screen camera view with a scanning frame overlay. Shows confirmation messages after successful tap-in or tap-out, including fare amount on exit.
- **Live Map**: Interactive Google Map showing real-time shuttle locations with route overlays. Students can see estimated arrival times at nearby stops.
- **Ride History**: Chronological list of past rides showing entry/exit stops, timestamps, fare charged, and ride status (completed or penalized).
- **Wallet**: Current balance display with transaction history showing credits (top-ups) and debits (fares, penalties).

**Driver Dashboard**

The driver interface focuses on simplicity during active driving:

- **Route Selection**: Upon login, drivers see their assigned shuttle and route. A prominent "Start Route" button initiates GPS broadcasting.
- **Active Route View**: Shows current route name, elapsed time, and a visual indicator that GPS is being broadcast. An "End Route" button stops the broadcast.
- **Shuttle Status**: Displays shuttle details (name, plate number, capacity) and current operational status.

**Admin Dashboard**

The admin interface provides comprehensive management tools in a desktop-optimized layout:

- **Overview Panel**: System statistics including active shuttles, total rides today, revenue collected, and pending penalties.
- **Route Management**: Table view of all routes with options to create, edit, activate/deactivate, and delete. Each route shows associated stops and active shuttles.
- **Stop Management**: Map-based interface for adding stops with drag-and-drop positioning. Stop details include name, coordinates, sequence order, and generated QR code.
- **Fare Matrix**: Grid-based editor for setting fares between stop pairs. Supports bulk updates and fare preview calculations.
- **User Management**: Searchable user list with filters by role. Admins can view user details, adjust wallet balance, and review ride history.
- **Live Monitoring**: Real-time map showing all active shuttles with passenger counts and route progress.
- **Logs & Reports**: Filterable logs of all rides, transactions, and penalties with export functionality.

**UI Standards**

All interfaces follow these design principles:

- Consistent color scheme using CSS variables (primary, secondary, muted tones)
- Responsive design adapting to screen sizes from 320px to 1920px+
- Toast notifications for success/error feedback (using Sonner library)
- Loading states with skeleton placeholders during data fetches
- Form validation with inline error messages
- Accessibility support including keyboard navigation and screen reader compatibility

Hardware Interfaces

The system interacts with hardware through standard web browser APIs rather than direct hardware communication:

**Smartphone Camera**

- Used for QR code scanning during tap-in and tap-out
- Accessed via the browser's MediaDevices API (getUserMedia)
- Minimum resolution requirement: 720p for reliable QR detection
- The html5-qrcode library handles camera initialization, frame capture, and QR decoding

**GPS/Location Services**

- Driver devices must have GPS capability for location broadcasting
- Accessed via the browser's Geolocation API (getCurrentPosition, watchPosition)
- Expected accuracy: 5-20 meters depending on device and environment
- Location updates are captured every 3 seconds during active routes
- Fallback to network-based location if GPS is unavailable (reduced accuracy)

**Display Requirements**

- Minimum screen resolution: 320px width (mobile) to 1920px (desktop)
- Touch screen support for mobile interactions
- Mouse and keyboard support for desktop admin interface

**No Specialized Hardware**

The system is designed to work with consumer smartphones and computers. No specialized hardware such as RFID readers, NFC terminals, or dedicated GPS trackers is required. This keeps deployment costs low and ensures broad accessibility.

Software Interfaces

The system integrates with several external software components and services:

**Google Maps JavaScript API**

- Purpose: Render interactive maps showing shuttle locations and routes
- Data Exchange: Receives map tiles and geocoding data; sends marker positions for shuttle locations
- Version: Latest stable release
- Authentication: API key stored in environment variables (NEXT_PUBLIC_GOOGLE_MAPS_API_KEY)

**Neon PostgreSQL Database**

- Purpose: Persistent storage for all application data
- Connection: Serverless WebSocket connection via @neondatabase/serverless driver
- Data Stored: Users, wallets, transactions, shuttles, routes, stops, fare matrices, rides, shuttle locations
- ORM: Drizzle ORM for type-safe database queries and migrations

**Better Auth**

- Purpose: User authentication and session management
- Features Used: Email/password authentication, session tokens, role-based access control
- Integration: Server-side auth handlers and client-side session hooks
- Session Storage: Database-backed sessions with secure HTTP-only cookies

**html5-qrcode Library**

- Purpose: QR code scanning in the browser
- Input: Camera video stream
- Output: Decoded QR code string containing stop identifier
- Error Handling: Provides callbacks for scan success, failure, and camera errors

**React Hook Form + Zod**

- Purpose: Form state management and validation
- Usage: All user input forms (login, registration, admin configuration)
- Validation: Schema-based validation with Zod, providing type inference and runtime checks

**Shared Data Formats**

All API responses follow a consistent JSON structure:
- Success: `{ "data": <payload> }`
- Error: `{ "error": "<message>", "details": [...] }`

Timestamps are stored and transmitted in ISO 8601 format (UTC).
Currency values use decimal representation with 2 decimal places.

Communications Interfaces

**HTTPS Protocol**

All client-server communication uses HTTPS (TLS 1.3) to ensure data encryption in transit. The system enforces HTTPS for:
- API requests and responses
- Authentication token transmission
- Sensitive data like wallet balances and personal information

**RESTful API Design**

The backend exposes RESTful endpoints following these conventions:
- Base URL: `/api/`
- HTTP Methods: GET (read), POST (create), PUT (update), DELETE (remove)
- Status Codes: 200 (OK), 201 (Created), 400 (Bad Request), 401 (Unauthorized), 403 (Forbidden), 404 (Not Found), 500 (Server Error)
- Content-Type: `application/json` for all requests and responses
- Authentication: Bearer tokens in Authorization header or HTTP-only session cookies

**Server-Sent Events (SSE)**

Real-time shuttle location updates use SSE for efficient one-way streaming:
- Endpoint: `/api/gps/stream`
- Content-Type: `text/event-stream`
- Event Format: `data: {"shuttleId": "...", "lat": 12.34, "lng": 56.78, "heading": 90, "speed": 25}\n\n`
- Update Frequency: Every 3 seconds per active shuttle
- Reconnection: Clients automatically reconnect on connection loss with exponential backoff

**Data Transfer Rates**

- GPS updates: ~200 bytes per update, 20 updates/minute per shuttle
- QR scan requests: ~500 bytes per request
- Dashboard data loads: 5-50 KB depending on data volume
- Map tile requests: Handled by Google Maps CDN

**Security Measures**

- CORS configured to allow only the application domain
- Rate limiting on authentication endpoints (5 attempts per minute)
- Input sanitization on all API endpoints using Zod validation
- SQL injection prevention via parameterized queries (Drizzle ORM)

System Features

This section describes the major features provided by the Smart Shuttle Tracking & Payment System. Each feature represents a core capability of the system and is described from a user and administrative perspective rather than a technical implementation perspective.

Ride Entry and Exit Tracking

Description and Priority

This feature allows a student to start and end a shuttle ride using a QR code scan. The system records the boarding and exit points of the ride and associates them with the current shuttle route cycle. This feature is fundamental to fare calculation and penalty enforcement.

Priority: High

Benefit: 9

Penalty if absent: 9

Cost: 5

Risk: 4

Stimulus/Response Sequences

The system continuously tracks the movement of each shuttle along predefined circular routes using location data. This allows the system to understand the shuttle’s progress within a cycle and determine whether a complete loop has been completed. Route tracking enables real-time monitoring and supports automatic verification of incomplete rides without requiring manual supervision from drivers or staff.

Functional Requirements

REQ-4.1-1: The system shall allow a student to initiate a ride by scanning a QR code

REQ-4.1-2: The system shall record the entry location and time of the ride.

REQ-4.1-3: The system shall associate the ride with the current shuttle route cycle.

REQ-4.1-4: The system shall allow the student to complete the ride using an exit scan.

REQ-4.1-5: The system shall prevent multiple active rides for a single user.

REQ-4.1-6: If an invalid or duplicate scan occurs, the system shall notify the user and reject the action.

Shuttle Route and Movement Tracking

Description and Priority

This feature tracks shuttle movement along predefined circular routes using location data. It enables real-time monitoring and supports detection of completed route cycles for penalty enforcement.

Priority: High

Benefit: 8

Penalty if absent: 8

Cost: 6

Risk: 5

Stimulus/Response Sequences

Shuttle begins operation on a configured route.

System continuously receives location updates.

System maps the shuttle’s position to the predefined route.

System detects completion of a full route cycle.

Functional Requirements

REQ-4.2-1: The system shall track shuttle movement in real time.

REQ-4.2-2: The system shall associate shuttle movement with predefined routes.

REQ-4.2-3: The system shall detect when a shuttle completes a full route cycle.

REQ-4.2-4: The system shall handle temporary loss of location data gracefully.

Fare Calculation and Wallet Deduction

Description and Priority

This feature calculates the fare for a completed ride based on entry and exit points and deducts the amount from the student’s digital wallet automatically.

Priority: High

Benefit: 9

Penalty if absent: 9

Cost: 4

Risk: 3

Stimulus/Response Sequences

Student completes a ride by scanning on exit.

System retrieves the fare matrix

System calculates the applicable fare.

System deducts the amount from the student’s wallet.

System confirms payment to the user.

Functional Requirements (TBD)

REQ-4.3-1: The system shall calculate fare based on entry and exit points.

REQ-4.3-2: The system shall deduct the calculated fare from the user’s wallet.

REQ-4.3-3: The system shall notify the user of successful payment.

REQ-4.3-4: If wallet balance is insufficient, the system shall handle the condition as per defined rules

 

Incomplete Ride Detection and Penalty Enforcement

Description and Priority

This feature identifies rides where a student fails to scan while exiting and applies a predefined penalty once the shuttle completes a full route cycle.

Priority: High

Benefit: 8

Penalty if absent: 9

Cost: 3

Risk: 4

Stimulus/Response Sequences

Student initiates a ride but does not scan on exit.

Shuttle completes a full route cycle.

System detects an incomplete ride.

System applies penalty to the student’s wallet.

System logs the penalty event.

Functional Requirements

REQ-4.4-1: The system shall detect rides without an exit scan.

REQ-4.4-2: The system shall apply a predefined penalty after a full route cycle.

REQ-4.4-3: The system shall record penalty transactions in ride logs.

REQ-4.4-4: The system shall notify the user when a penalty is applied.

User Account and Ride History Management

Description and Priority

This feature allows students to register, manage their profiles, and view ride history, payments, and penalties.

Priority: Medium

Benefit: 7

Penalty if absent: 5

Cost: 3

Risk: 2

Stimulus/Response Sequences

User logs into the application.

User navigates to ride history.

System displays previous rides and transactions.

Functional Requirements

REQ-4.5-1: The system shall allow users to register and authenticate.

REQ-4.5-2: The system shall display ride history and payment records.

REQ-4.5-3: The system shall display wallet balance and penalties.

Administrative Route and Fare Management 

 Description and Priority

This feature enables administrators to configure routes, define stops, and manage fare matrices.

Priority: High

Benefit: 8

Penalty if absent: 7

Cost: 4

Risk: 3

Stimulus/Response Sequences

Admin logs into the admin dashboard.

Admin creates or updates a route.

Admin updates fare matrix.

System saves and applies changes.

Functional Requirements

REQ-4.6-1: The system shall allow admins to configure shuttle routes.

REQ-4.6-2: The system shall allow admins to manage fare matrices.

REQ-4.6-3: The system shall validate route and fare configurations.

Real-Time Shuttle and Ride Monitoring

Description and Priority

This feature provides administrators with real-time visibility into shuttle locations, active passengers, and ride logs.

Priority: Medium

Benefit: 7

Penalty if absent: 6

Cost: 5

Risk: 4

Stimulus/Response Sequences

Admin opens monitoring dashboard.

System displays live shuttle locations.

Admin views ride logs and passenger counts.

Functional Requirements

REQ-4.7-1: The system shall display real-time shuttle locations.

REQ-4.7-2: The system shall show active rides per shuttle.

REQ-4.7-3: The system shall provide access to ride and transaction logs.

  

Other Nonfunctional Requirements

Performance Requirements

The system shall support real-time shuttle tracking with minimal delay to ensure accurate ride validation and monitoring. Ride entry, exit, fare calculation, and penalty application shall be processed within an acceptable response time to maintain a smooth user experience. The system shall be capable of handling multiple concurrent users and shuttles without noticeable degradation in performance.

Safety Requirements

The system shall ensure that no user data or transaction information is lost due to unexpected system failures. In case of partial failures such as temporary network loss or GPS unavailability, the system shall preserve ride state information and recover gracefully once normal operation resumes. Critical operations such as fare deduction and penalty enforcement shall not be executed in an inconsistent or duplicated manner.

Security Requirements

The system shall ensure secure authentication and authorization for both students and administrators. User wallet data, ride history, and personal information shall be protected against unauthorized access. Administrative functions such as route configuration and fare management shall be accessible only to authorized personnel. All financial transactions shall be securely processed and logged to prevent tampering or misuse.

Software Quality Attributes

The system shall be reliable and available during shuttle operating hours. It shall be designed for maintainability so that routes, fares, and rules can be updated without affecting core functionality. The system shall be scalable to support an increase in the number of users, shuttles, or routes without requiring major changes. Usability shall be prioritized to ensure that students can use the system with minimal learning effort.

Business Rules

The system shall enforce a fixed penalty for incomplete rides as defined by campus transportation policies. Fare calculation shall strictly follow the configured fare matrix without manual overrides during active rides. Penalty enforcement shall be automatic and consistent across all users to ensure fairness and transparency.

Other Requirements

**Database Requirements**

- All data must be stored in a PostgreSQL database (Neon serverless)
- Database schema must support UUID primary keys for all tables
- Transaction records must be immutable once created
- Ride and transaction data must support efficient querying by date range and user

**Internationalization**

- Currency display shall use Indian Rupee (₹) format
- Date and time shall be stored in UTC and displayed in IST (Indian Standard Time)
- The initial release supports English only; multi-language support may be added later

**Legal and Compliance**

- User data handling must comply with applicable data protection regulations
- Transaction logs must be retained for audit purposes as per institutional policies
- Users must accept terms of service during registration

**Deployment Requirements**

- The system shall be deployable on Vercel or similar serverless platforms
- Environment variables must be used for all sensitive configuration (API keys, database URLs)
- The system shall support zero-downtime deployments

Appendix A: Glossary

| Term | Definition |
|------|------------|
| **SSTPS** | Smart Shuttle Tracking & Payment System - the official name of this software system |
| **Tap-In** | The action of scanning a QR code when boarding a shuttle to start a ride |
| **Tap-Out** | The action of scanning a QR code when exiting a shuttle to end a ride and trigger fare calculation |
| **QR Code** | Quick Response code - a two-dimensional barcode that encodes stop identification data, scanned by smartphone cameras |
| **Circular Route** | A shuttle route that forms a continuous loop, where the last stop connects back to the first stop |
| **Fare Matrix** | A configuration table defining the fare amount between any two stops on a route |
| **Wallet** | A digital account balance associated with each student, used for automatic fare deduction |
| **Penalty** | A fixed charge (₹50) applied when a student fails to tap-out within the allowed time (90 minutes) |
| **SSE** | Server-Sent Events - a web technology enabling real-time one-way data streaming from server to client |
| **PWA** | Progressive Web Application - a web app that can be installed on devices and provides app-like experience |
| **GPS** | Global Positioning System - satellite-based navigation providing location coordinates |
| **API** | Application Programming Interface - a set of protocols for building and integrating software |
| **REST** | Representational State Transfer - an architectural style for designing networked applications |
| **HTTPS** | Hypertext Transfer Protocol Secure - encrypted communication protocol for secure data transfer |
| **ORM** | Object-Relational Mapping - a technique for converting data between incompatible systems using object-oriented programming |
| **UUID** | Universally Unique Identifier - a 128-bit identifier used for database primary keys |
| **Session** | A temporary authentication state that tracks a logged-in user across requests |
| **Geolocation API** | A browser API that provides access to device location through GPS or network triangulation |
| **Drizzle** | A TypeScript ORM used for type-safe database queries |
| **Neon** | A serverless PostgreSQL database service used for data persistence |
| **Better Auth** | An authentication library for handling user login, registration, and session management |
| **Zod** | A TypeScript-first schema validation library used for input validation |
| **CORS** | Cross-Origin Resource Sharing - a security mechanism controlling which domains can access the API |
| **Route Cycle** | One complete loop of a shuttle through all stops on its assigned circular route |

Appendix B: Analysis Models

Detailed analysis models for the SSTPS are documented in a separate file: `docs/diagrams.md`. This file contains the following UML diagrams created using Mermaid syntax:

**Use Case Diagram**

Illustrates all system actors (Student, Driver, Admin, System) and their interactions with 27 identified use cases. Shows relationships between use cases including dependencies for fare calculation and penalty enforcement flows.

**Sequence Diagrams**

Five sequence diagrams detail the step-by-step message flow for critical system operations:

1. **Tap-In Flow**: Student scans QR → system validates stop → checks wallet → creates ride record
2. **Tap-Out Flow**: Student scans exit QR → system calculates fare → deducts wallet → completes ride
3. **GPS Broadcasting Flow**: Driver starts route → system broadcasts position every 3 seconds via SSE → students receive updates
4. **Penalty Flow**: Scheduled job detects incomplete rides → applies penalty → records transaction
5. **Wallet Top-up Flow**: Admin selects user → enters amount → system credits wallet

**Class Diagram**

Defines the system's data model with 10 entity classes:
- User, Wallet, Transaction (user domain)
- Shuttle, Route, Stop, FareMatrix, ShuttleLocation (shuttle domain)
- Ride, Session (operational domain)

Includes 4 enumerations: UserRole, ShuttleStatus, RideStatus, TransactionType

**State Diagrams**

Five state diagrams model the lifecycle of key entities:

1. **Ride States**: Idle → Active → Completed/Penalty
2. **Shuttle States**: Inactive → Active (Broadcasting/Stopped) → Maintenance
3. **Wallet Transaction States**: Idle → Processing → Credited/Debited/Failed
4. **User Authentication States**: Guest → Authenticating → LoggedIn (role-based)
5. **Route Management States**: Draft → Active → Inactive → Deleted

All diagrams are rendered using Mermaid and can be viewed in any compatible Markdown viewer including GitHub.

Appendix C: To Be Determined List

The following items require further discussion or decision before or during implementation:

| ID | Item | Description | Status |
|----|------|-------------|--------|
| TBD-1 | Payment Gateway Integration | Method for students to add money to their wallets (UPI, card payments, campus payment system integration) | Pending |
| TBD-2 | Notification System | Whether to implement push notifications, SMS alerts, or email notifications for ride confirmations and penalties | Pending |
| TBD-3 | Offline Mode | Handling scenarios where students lose connectivity mid-ride; whether to queue tap-out requests | Pending |
| TBD-4 | Refund Policy | Process for disputing incorrect fares or penalties; admin workflow for issuing refunds | Pending |
| TBD-5 | Minimum Wallet Balance | Whether to enforce a minimum balance requirement before allowing tap-in (and what that amount should be) | Pending |
| TBD-6 | Multi-Route Transfers | Handling students who transfer between shuttles on different routes within a single journey | Pending |
| TBD-7 | Peak Hour Pricing | Whether fare matrices should support time-based pricing variations | Pending |
| TBD-8 | Data Retention Policy | How long to retain ride history, GPS logs, and transaction records | Pending |
| TBD-9 | Accessibility Features | Specific accessibility requirements beyond standard WCAG compliance | Pending |
| TBD-10 | Load Testing Targets | Exact concurrent user limits and performance benchmarks | Pending |