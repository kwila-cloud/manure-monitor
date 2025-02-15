# Manure Monitor: Manure Management System
Design Document

*Alternative names under consideration:*
- SpreadWise
- SmartSpread Pro
- PrecisionSpread
- AgriSpread
- FieldSpread

## Overview
This document outlines the design for a multi-tenant mobile/web application that will automate the tracking and calculation of chicken manure spreading across multiple farms and fields. Each customer organization will have their own isolated SQLite database instance, with separate user accounts, farms, and field data.

## Goals
- Digitize the manual process of recording manure loads
- Automate calculations for spread weight and rate per acre
- Support multiple farms, each with multiple fields
- Track historical spreading data
- Generate reports for regulatory compliance
- Provide offline capability with synchronization

## User Personas

### Primary User: Field Operator
- Needs to quickly log load weights while in the field
- May have limited connectivity in rural areas
- Requires simple, intuitive interface that works with dirty/gloved hands
- Needs to see running totals and calculations in real-time

### Secondary User: Farm Manager/Owner
- Needs to review spreading data across all fields and farms
- Requires reports for regulatory compliance
- Wants to analyze spreading rates and efficiency

### Administrator: Organization Admin
- Needs to add/manage farms, fields, and operators
- Requires comprehensive reporting capabilities
- Wants to ensure data accuracy and completeness

## Data Model

```
User
  - id
  - name
  - role (operator, manager, admin)
  - email
  - password (hashed)
  - last_sync_timestamp

Farm
  - id
  - name
  - address
  - contact_info
  - total_acreage

Field
  - id
  - farm_id (foreign key)
  - name
  - acreage
  - location_coordinates
  - notes

Job
  - id
  - field_id (foreign key)
  - operator_id (foreign key)
  - date
  - start_time
  - end_time
  - target_rate
  - unit_no
  - job_no
  - total_spread_weight
  - total_acres_spread
  - average_rate_per_acre
  - status (in-progress, completed)
  - sync_status (pending, synced)
  - last_modified_timestamp

Load
  - id
  - job_id (foreign key)
  - load_number
  - starting_scale_weight
  - ending_scale_weight
  - spread_weight (calculated)
  - end_acres
  - rate_per_acre (calculated)
  - timestamp
  - sync_status (pending, synced)
  - last_modified_timestamp
```

## Features and Functionality

### Authentication and User Management
- Secure login system
- Role-based access control
- Password reset functionality
- Local authentication for offline use

### Farm and Field Management
- Add/edit/archive farms
- Add/edit/archive fields within farms
- View field history
- Map integration for field visualization (future phase)

### Job Management
- Create new spreading job
- Assign to specific field/operator
- Set target spread rate
- Track start/end times
- Calculate and display real-time totals
- Mark jobs as complete
- Work offline with local storage

### Load Tracking
- Intuitive interface for adding new loads
- Auto-calculate spread weight (starting weight - ending weight)
- Track cumulative acres covered
- Calculate and display rate per acre
- Full offline support with local data storage
- Background synchronization when connectivity returns

### Reporting
- Generate PDF reports similar to current paper form
- Export data to CSV/Excel
- Filter reports by date range, farm, field, operator
- Compliance reporting for agricultural regulations
- Local report generation with sync to central system

### Dashboard
- Overview of recent and active jobs
- Quick stats on total tonnage spread
- Alerts for unusual spreading rates
- Weather integration for optimal spreading conditions (future phase)
- Offline access to recent data

### Synchronization
- Automatic sync when connectivity is available
- Manual sync option for operators
- Conflict resolution for simultaneous edits
- Sync status indicators
- Incremental sync to minimize data transfer

## Technical Specifications

### Database: SQLite
- Local SQLite database on each device
- Schema designed for efficient local operations
- Support for offline data storage and operations
- Conflict resolution strategy for sync operations
- Regular local backups to prevent data loss

### Synchronization Layer
- RESTful API for data synchronization
- Timestamp-based change tracking
- Conflict detection and resolution logic
- Efficient data transfer (only changed records)
- Queue system for pending changes
- Central server for data aggregation and storage

### Frontend
- Progressive Web App (PWA) for cross-platform compatibility
- Responsive design for mobile and desktop use
- Offline capability with local storage
- Simple, high-contrast UI for outdoor visibility
- Local data caching

### Backend
- Lightweight API server for synchronization
- Authentication using JWT tokens with offline capability
- Central SQLite or PostgreSQL database for aggregation
- Scheduled backups of central database
- Support for limited concurrent connections (up to 6)

### Integration Points
- Weather API for conditions monitoring (future)
- GPS integration for field mapping (future)
- Scale system API connection (if available)

## User Interface Mockups

### Mobile App - Load Entry Screen
```
+----------------------------------+
|  Manure Monitor        ☰  👤  |
+----------------------------------+
|  Farm: Smith Family Farm         |
|  Field: North Cornfield (42ac)   |
|  Job #: J20250215                |
|                        🔄 Synced |
+----------------------------------+
|  ◄ LOAD #3 ►                     |
|                                  |
|  Starting Weight:                |
|  [      15,780      ] lbs        |
|                                  |
|  Ending Weight:                  |
|  [      14,210      ] lbs        |
|                                  |
|  Spread Weight: 1,570 lbs        |
|                                  |
|  Acres Covered:                  |
|  [       3.2       ]             |
|                                  |
|  Rate: 490.6 lbs/acre            |
|                                  |
|  [    SAVE LOAD    ]             |
|                                  |
|  [ + ADD ANOTHER LOAD ]          |
+----------------------------------+
|  JOB SUMMARY:                    |
|  Total Spread: 4,680 lbs         |
|  Total Acres: 9.7                |
|  Avg Rate: 482.5 lbs/acre        |
|                                  |
|  Target Rate: 500 lbs/acre       |
|                                  |
|  [  SYNC DATA  ] [  FINISH JOB  ]|
+----------------------------------+
```

### Web Dashboard - Job Overview (with sync status)
```
+-----------------------------------------------------------------------+
|  Manure Monitor                                    Admin ▼  Logout  |
+-----------------------------------------------------------------------+
|  Dashboard  |  Farms  |  Jobs  |  Reports  |  Settings  |             |
+-----------------------------------------------------------------------+
|                                                   Last sync: 10:45 AM  |
|  Active Jobs                                        [ + New Job ]     |
|  +-----------------------------------------------------------------+  |
|  | Date       | Farm         | Field        | Operator   | Status  |  |
|  |-----------------------------------------------------------------|  |
|  | 2025-02-15 | Smith Family | North Corn   | J. Miller  | Active 🔄|  |
|  | 2025-02-15 | Greenfield   | West Field   | C. Thomas  | Active 🔄|  |
|  | 2025-02-14 | Riverdale    | South Pasture| J. Miller  | Paused ⚠️ |  |
|  +-----------------------------------------------------------------+  |
|                                                                       |
|  Recent Completed Jobs                             [ View All... ]    |
|  +-----------------------------------------------------------------+  |
|  | Job #    | Farm         | Field        | Total Spread | Rate    |  |
|  |-----------------------------------------------------------------|  |
|  | J20250213| Smith Family | South Corn   | 12.4 tons    | 485 lbs |  |
|  | J20250212| Greenfield   | East Field   | 8.7 tons     | 510 lbs |  |
|  | J20250211| Johnson Farm | North 40     | 18.2 tons    | 495 lbs |  |
|  +-----------------------------------------------------------------+  |
|                                                                       |
|  Sync Status                                     [ Force Sync ]       |
|  +-----------------------------------------------------------------+  |
|  | Operator    | Last Sync          | Pending Changes | Status     |  |
|  |-----------------------------------------------------------------|  |
|  | J. Miller   | Today, 10:45 AM    | 0               | ✓ Current  |  |
|  | C. Thomas    | Today, 8:30 AM     | 2               | ⚠️ Pending  |  |
|  | T. Johnson  | Yesterday, 4:15 PM | 0               | ✓ Current  |  |
|  +-----------------------------------------------------------------+  |
+-----------------------------------------------------------------------+
```

## Implementation Phases

### Phase 1: Core Functionality with SQLite (3 months)
- Set up SQLite database schema
- Implement local-first architecture
- User, farm, and field management
- Basic job and load tracking
- Simple reporting
- Mobile app with offline capability
- Basic synchronization mechanism
- Web dashboard for administration

### Phase 2: Enhanced Sync and Features (2 months)
- Robust synchronization with conflict resolution
- Advanced reporting and analytics
- Data visualization (charts, graphs)
- Improved offline functionality
- Sync status monitoring and management
- API integrations (if applicable)

### Phase 3: Optimization (2 months)
- Performance improvements for SQLite operations
- Sync efficiency enhancements
- User feedback incorporation
- Additional features based on initial usage
- Scale integration (if available)
- Evaluation of database scaling needs

## Technologies to Consider
- Frontend: React Native (mobile) or React (web PWA)
- Backend: Node.js with Express
- Database: SQLite for local storage
- Sync Layer: Custom REST API with timestamp-based syncing
- Central Storage: SQLite or PostgreSQL based on growth
- Cloud hosting: Lightweight VPS or AWS/Azure
- CI/CD: Jenkins or GitHub Actions

## Security Considerations
- Local data encryption for SQLite databases
- Secure sync mechanism with authentication
- Regular security audits
- Compliance with agricultural data privacy regulations
- Secure API access with rate limiting
- Device-level security recommendations

## SQLite-Specific Considerations
- Regular vacuum operations to maintain database performance
- Appropriate indexing for sync-related queries
- Transaction management for data integrity
- Backup strategy for local databases
- Database versioning and migration strategy
- Handling of database corruption scenarios

## Synchronization Strategy
1. Each device maintains a local SQLite database
2. Changes are tracked with timestamps and sync status flags
3. When online, devices sync with central server in background
4. Conflicts are resolved using last-modified timestamp and predefined rules
5. Central server aggregates all data for reporting and dashboard
6. Periodic full sync to ensure data consistency

## Success Metrics
- Reduction in manual calculation errors
- Time saved per job (target: 15 minutes)
- User adoption rate (target: 90% of operators within 2 months)
- Sync success rate (target: 99.5%)
- Reduction in paper usage
- Improved reporting accuracy for compliance

## Maintenance Plan
- Regular backups of all local and central databases
- Scheduled maintenance for central synchronization server
- Monitoring of sync failures and resolution
- Scheduled updates and feature additions
- User feedback collection and incorporation
- Technical support process for field operators

## Conclusion
This app will significantly improve the efficiency of manure spreading operations by automating calculations, reducing errors, and providing valuable insights through data analytics. The SQLite-based architecture ensures operators can work effectively even in areas with limited connectivity, while the synchronization mechanism ensures data integrity and availability for reporting and analysis.
