# Number Porting Workflow

## Overview
Complete workflow for porting phone numbers into and out of the platform.

## Port-In Workflow

```mermaid
stateDiagram-v2
    [*] --> Pending: User Initiates Request
    
    Pending --> DocumentsUploaded: User Uploads Documents
    
    DocumentsUploaded --> AdminReview: Auto-transition
    
    AdminReview --> Approved: Admin Approves
    AdminReview --> Rejected: Admin Rejects
    
    Rejected --> [*]: Notify User
    
    Approved --> SubmittedToCarrier: Backend Submits
    
    SubmittedToCarrier --> Completed: Carrier Confirms (3-5 days)
    SubmittedToCarrier --> Failed: Carrier Rejects
    
    Failed --> AdminReview: Review & Retry
    
    Completed --> [*]: Number Active
    
    AdminReview --> Cancelled: User Cancels
    Approved --> Cancelled: User Cancels
    SubmittedToCarrier --> Cancelled: User Cancels
    
    Cancelled --> [*]: Refund Issued

    note right of Pending
        User provides:
        - Phone number
        - Current carrier
        - Account details
    end note

    note right of DocumentsUploaded
        Required documents:
        - ID proof
        - Ownership proof
        - Account number
        - PIN/Password
    end note

    note right of AdminReview
        Admin verifies:
        - Document validity
        - Identity match
        - Ownership proof
        - Completeness
    end note

    note right of SubmittedToCarrier
        Automated process:
        - API call to carrier
        - LNP request submission
        - Status tracking
    end note

    note right of Completed
        Final steps:
        - Number activated
        - SIP routing updated
        - User notified
        - Billing starts
    end note
```

## Port-In Sequence Diagram

```mermaid
sequenceDiagram
    participant User
    participant App as Mobile App
    participant Laravel as Laravel API
    participant Admin as Admin Panel
    participant Carrier as Telecom Carrier
    participant SIP as SIP Server

    Note over User,SIP: Step 1: User Initiates Port-In

    User->>App: 1. Request Port-In<br/>Enter Number Details
    
    App->>Laravel: 2. POST /porting/request<br/>{number, carrier, account}
    
    activate Laravel
    Laravel->>Laravel: 3. Create Porting Request<br/>Status: pending
    
    Laravel-->>App: 4. Request Created<br/>Upload Documents
    deactivate Laravel
    
    User->>App: 5. Upload Documents<br/>(ID, Ownership Proof)
    
    App->>Laravel: 6. POST Documents
    
    activate Laravel
    Laravel->>Laravel: 7. Store Documents<br/>Status: documents_uploaded
    
    Laravel->>Admin: 8. Notification:<br/>New Porting Request
    
    Laravel-->>App: 9. Documents Received
    deactivate Laravel

    Note over User,SIP: Step 2: Admin Review

    Admin->>Laravel: 10. GET /admin/porting/{id}
    
    activate Laravel
    Laravel-->>Admin: 11. Request Details<br/>+ Documents
    deactivate Laravel
    
    Note over Admin: 12. Review Documents<br/>Verify Identity<br/>Check Completeness
    
    alt Documents Valid
        Admin->>Laravel: 13. POST /admin/porting/{id}/approve
        
        activate Laravel
        Laravel->>Laravel: 14. Update Status:<br/>approved
        
        Laravel->>User: 15. Email: Request Approved<br/>Processing Started
        
        Note over Laravel: Step 3: Submit to Carrier
        
        Laravel->>Carrier: 16. LNP Request API<br/>{number, account, auth}
        deactivate Laravel
        
        activate Carrier
        Carrier->>Carrier: 17. Validate Request<br/>Check Current Carrier<br/>Initiate Port
        
        Carrier-->>Laravel: 18. Port Request Accepted<br/>Reference Number
        deactivate Carrier
        
        activate Laravel
        Laravel->>Laravel: 19. Update Status:<br/>submitted_to_carrier
        
        Laravel->>User: 20. Notification:<br/>Submitted to Carrier<br/>ETA: 3-5 business days
        deactivate Laravel
        
        Note over Carrier: 21. Carrier Processing<br/>(3-5 business days)
        
        Carrier->>Laravel: 22. Webhook: Port Complete
        
        activate Laravel
        Laravel->>Laravel: 23. Update Status: completed<br/>Create Phone Number Record
        
        Laravel->>SIP: 24. Configure Routing<br/>for New Number
        
        SIP-->>Laravel: 25. Routing Active
        
        Laravel->>User: 26. Notification:<br/>Port Complete!<br/>Number Active
        
        Laravel-->>App: 27. Push: Number Ready
        deactivate Laravel
        
    else Documents Invalid
        Admin->>Laravel: 13b. POST /admin/porting/{id}/reject<br/>{reason}
        
        activate Laravel
        Laravel->>Laravel: 14b. Update Status: rejected
        
        Laravel->>User: 15b. Email: Request Rejected<br/>Reason: {reason}
        deactivate Laravel
    end
```

## Port-Out Workflow

```mermaid
graph TB
    Start([User Requests<br/>Port-Out]) --> Validate{Verify Number<br/>Ownership}
    
    Validate -->|Not Owned| Error1[Error: Not Authorized]
    Validate -->|Owned| CheckBalance{Outstanding<br/>Balance?}
    
    CheckBalance -->|Yes| Error2[Error: Clear Balance First]
    CheckBalance -->|No| GenerateInfo[Generate Port-Out Info]
    
    GenerateInfo --> DisplayInfo[Display to User:<br/>- Account Number<br/>- PIN<br/>- Current Carrier Info]
    
    DisplayInfo --> UserAction{User Action}
    
    UserAction -->|Cancel| End1([Cancelled])
    UserAction -->|Confirm| CreateRequest[Create Port-Out Request<br/>Status: pending]
    
    CreateRequest --> NotifyAdmin[Notify Admin]
    
    NotifyAdmin --> AdminReview{Admin Review}
    
    AdminReview -->|Reject| Rejected[Status: rejected<br/>Notify User]
    AdminReview -->|Approve| Approved[Status: approved<br/>Provide Auth Info]
    
    Rejected --> End2([Port-Out Denied])
    
    Approved --> WaitNewCarrier[Wait for New Carrier<br/>to Request Port]
    
    WaitNewCarrier --> CarrierRequest{New Carrier<br/>Requests Port}
    
    CarrierRequest -->|Timeout 30d| Expired[Status: expired]
    CarrierRequest -->|Request Received| ValidateRequest[Validate Request<br/>Against Our Records]
    
    ValidateRequest --> ApprovePort[Approve Port<br/>to New Carrier]
    
    ApprovePort --> DisableNumber[Disable Number<br/>on Our Platform]
    
    DisableNumber --> UpdateStatus[Status: completed]
    
    UpdateStatus --> NotifyUser[Notify User:<br/>Port Complete]
    
    Expired --> End3([Request Expired])
    NotifyUser --> End4([Port-Out Complete])

    classDef errorStyle fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    classDef processStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef decisionStyle fill:#fff9c4,stroke:#f57f17,stroke-width:2px

    class Error1,Error2,Rejected,Expired errorStyle
    class GenerateInfo,DisplayInfo,CreateRequest,Approved,ApprovePort,DisableNumber,UpdateStatus,NotifyUser processStyle
    class Validate,CheckBalance,UserAction,AdminReview,CarrierRequest decisionStyle
```

## Porting Timeline

```mermaid
gantt
    title Number Porting Timeline (Port-In)
    dateFormat YYYY-MM-DD
    section User Actions
    Submit Request           :a1, 2024-01-01, 1d
    Upload Documents         :a2, after a1, 1d
    
    section Admin Review
    Document Review          :a3, after a2, 1d
    Approval Decision        :a4, after a3, 1d
    
    section Carrier Processing
    Submit to Carrier        :a5, after a4, 1d
    Carrier Validation       :a6, after a5, 1d
    Port Processing          :a7, after a6, 3d
    Port Activation          :a8, after a7, 1d
    
    section Platform Setup
    Configure Routing        :a9, after a8, 1d
    Number Activation        :a10, after a9, 1d
```

## Porting Request States

| State | Description | User Actions | Admin Actions | Next States |
|-------|-------------|--------------|---------------|-------------|
| **pending** | Initial request created | Upload documents | - | documents_uploaded |
| **documents_uploaded** | Documents provided | Wait | Review documents | admin_review |
| **admin_review** | Under admin review | Wait | Approve/Reject | approved, rejected |
| **approved** | Admin approved | Wait | - | submitted_to_carrier |
| **rejected** | Admin rejected | Resubmit or cancel | - | - |
| **submitted_to_carrier** | Sent to carrier | Wait | Monitor | completed, failed |
| **failed** | Carrier rejected | Provide more info | Review & retry | admin_review |
| **completed** | Port successful | Use number | - | - |
| **cancelled** | User cancelled | - | - | - |

## Required Documents

### For Port-In
1. **Government-issued ID**
   - Passport
   - Driver's License
   - National ID Card

2. **Proof of Ownership**
   - Recent bill from current carrier
   - Account statement
   - Contract/Agreement

3. **Account Information**
   - Account number
   - PIN/Password (if applicable)
   - Billing address

### For Port-Out
1. **Account Number**: Provided by our platform
2. **PIN**: Generated for port-out authorization
3. **Carrier Information**: Our carrier details
4. **Billing Status**: Must be current (no outstanding balance)

## API Endpoints

```
POST   /api/v1/porting/request          # Initiate port-in
GET    /api/v1/porting/status/{id}      # Check status
POST   /api/v1/porting/{id}/documents   # Upload documents
DELETE /api/v1/porting/{id}             # Cancel request

# Admin endpoints
GET    /api/v1/admin/porting            # List all requests
GET    /api/v1/admin/porting/{id}       # View details
POST   /api/v1/admin/porting/{id}/approve   # Approve
POST   /api/v1/admin/porting/{id}/reject    # Reject
POST   /api/v1/admin/porting/{id}/submit    # Submit to carrier
```

## Notifications

### User Notifications
- Request received
- Documents uploaded successfully
- Admin review in progress
- Request approved/rejected
- Submitted to carrier
- Port completed
- Number active

### Admin Notifications
- New porting request
- Documents uploaded
- Carrier response received
- Port completion

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| Invalid number format | Wrong format | Validate and resubmit |
| Number not portable | Landline/special number | Contact support |
| Missing documents | Incomplete upload | Upload missing docs |
| Carrier rejection | Account mismatch | Verify account details |
| Timeout | Carrier delay | Contact carrier support |
