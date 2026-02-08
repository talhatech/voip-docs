# Authentication Flow

## Overview
Complete authentication flow including user login, JWT token generation, and SIP registration.

## User Authentication Flow

```mermaid
sequenceDiagram
    participant User as User
    participant App as Mobile App
    participant Laravel as Laravel API
    participant DB as Database
    participant Kamailio as Kamailio<br/>SIP Server

    Note over User,Kamailio: Initial Login

    User->>App: 1. Enter Email/Password
    
    App->>Laravel: 2. POST /auth/login<br/>{email, password}
    
    activate Laravel
    Laravel->>DB: 3. Find User by Email
    
    DB-->>Laravel: 4. User Record
    
    Note over Laravel: 5. Verify Password<br/>(bcrypt hash)
    
    alt Password Valid
        Note over Laravel: 6. Generate JWT Token<br/>7. Generate SIP Credentials
        
        Laravel->>DB: 8. Create/Update<br/>Device Registration
        
        Laravel-->>App: 9. 200 OK<br/>{<br/>  token: "jwt_token",<br/>  user: {...},<br/>  sip: {<br/>    username: "user@domain",<br/>    password: "sip_pass",<br/>    server: "sip.domain.com"<br/>  }<br/>}
        
        deactivate Laravel
        
        Note over App: 10. Store JWT Token<br/>Store SIP Credentials
        
        App->>Kamailio: 11. SIP REGISTER<br/>Username: user@domain
        
        activate Kamailio
        Kamailio-->>App: 12. 401 Unauthorized<br/>WWW-Authenticate Challenge
        
        Note over App: 13. Calculate Response<br/>Using SIP Password
        
        App->>Kamailio: 14. SIP REGISTER<br/>With Authorization Header
        
        Note over Kamailio: 15. Validate Credentials<br/>Against Laravel DB
        
        Kamailio->>Laravel: 16. Auth Check API
        
        activate Laravel
        Laravel->>DB: 17. Verify SIP Credentials
        Laravel-->>Kamailio: 18. Auth Success
        deactivate Laravel
        
        Kamailio-->>App: 19. 200 OK<br/>Registration Successful
        deactivate Kamailio
        
        Note over App: User Registered<br/>Ready for Calls
        
    else Password Invalid
        Laravel-->>App: 401 Unauthorized<br/>{error: "Invalid credentials"}
    end
```

## PIN-Based Quick Access

```mermaid
sequenceDiagram
    participant User as User
    participant App as Mobile App
    participant Keychain as Secure Keychain
    participant Laravel as Laravel API

    Note over User,Laravel: First Time Setup

    User->>App: 1. Set 6-Digit PIN
    
    App->>Keychain: 2. Store JWT Token<br/>in Secure Storage
    
    App->>Keychain: 3. Store PIN Hash
    
    Note over App: PIN Setup Complete

    Note over User,Laravel: Subsequent Access

    User->>App: 4. Enter PIN
    
    App->>Keychain: 5. Retrieve PIN Hash
    
    Keychain-->>App: 6. PIN Hash
    
    Note over App: 7. Verify PIN Locally
    
    alt PIN Valid
        App->>Keychain: 8. Retrieve JWT Token
        
        Keychain-->>App: 9. JWT Token
        
        App->>Laravel: 10. GET /user/profile<br/>Authorization: Bearer {token}
        
        activate Laravel
        
        alt Token Valid
            Laravel-->>App: 11. 200 OK<br/>User Profile
            Note over App: User Logged In
        else Token Expired
            Laravel-->>App: 12. 401 Unauthorized
            Note over App: Redirect to<br/>Full Login
        end
        
        deactivate Laravel
        
    else PIN Invalid
        Note over App: Show Error<br/>Retry or Full Login
    end
```

## QR Code Device Pairing

```mermaid
sequenceDiagram
    participant OldDevice as Old Device
    participant NewDevice as New Device
    participant Laravel as Laravel API
    participant DB as Database

    Note over OldDevice,DB: Device Transfer via QR

    OldDevice->>Laravel: 1. POST /device/transfer/initiate
    
    activate Laravel
    Note over Laravel: 2. Create Encrypted<br/>Backup of User Data
    
    Laravel->>DB: 3. Store Temporary<br/>Transfer Token
    
    Laravel-->>OldDevice: 4. Transfer Token<br/>+ QR Code Data
    deactivate Laravel
    
    Note over OldDevice: 5. Display QR Code
    
    NewDevice->>OldDevice: 6. Scan QR Code
    
    Note over NewDevice: 7. Extract Transfer Token
    
    NewDevice->>Laravel: 8. POST /device/transfer/complete<br/>{token}
    
    activate Laravel
    Laravel->>DB: 9. Validate Token<br/>(Check Expiry)
    
    alt Token Valid
        Laravel->>DB: 10. Retrieve Encrypted<br/>Backup Data
        
        Laravel-->>NewDevice: 11. Encrypted Data<br/>+ Decryption Key
        
        Note over NewDevice: 12. Decrypt Data<br/>Import to Local DB
        
        NewDevice->>Laravel: 13. Confirm Transfer<br/>Complete
        
        Laravel->>DB: 14. Invalidate Token<br/>Delete Backup
        
        Laravel-->>NewDevice: 15. Success
        
        Note over NewDevice: Device Ready
        
    else Token Invalid/Expired
        Laravel-->>NewDevice: 16. Error: Invalid Token
    end
    
    deactivate Laravel
```

## Token Refresh Flow

```mermaid
graph TB
    Start([API Request]) --> CheckToken{JWT Token<br/>Valid?}
    
    CheckToken -->|Valid| ProcessRequest[Process Request]
    CheckToken -->|Expired| CheckRefresh{Has Refresh<br/>Token?}
    
    CheckRefresh -->|No| Logout[Force Logout]
    CheckRefresh -->|Yes| ValidateRefresh{Refresh Token<br/>Valid?}
    
    ValidateRefresh -->|Invalid| Logout
    ValidateRefresh -->|Valid| IssueNew[Issue New JWT<br/>+ Refresh Token]
    
    IssueNew --> RetryRequest[Retry Original<br/>Request]
    
    RetryRequest --> ProcessRequest
    
    ProcessRequest --> Response[Return Response]
    
    Logout --> LoginScreen[Redirect to<br/>Login Screen]
    
    Response --> End([Complete])
    LoginScreen --> End

    classDef successStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef errorStyle fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    classDef processStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px

    class ProcessRequest,IssueNew,RetryRequest,Response successStyle
    class Logout,LoginScreen errorStyle
    class CheckToken,CheckRefresh,ValidateRefresh processStyle
```

## Multi-Device Session Management

```mermaid
graph TB
    subgraph User["User Account"]
        UserID[User ID: 12345]
    end

    subgraph Devices["Registered Devices"]
        Device1["Device 1<br/>iPhone 14<br/>Active<br/>Last Seen: 2m ago"]
        Device2["Device 2<br/>Android Pixel<br/>Active<br/>Last Seen: 1h ago"]
        Device3["Device 3<br/>iPad<br/>Inactive<br/>Last Seen: 2d ago"]
    end

    subgraph SIPRegistrations["SIP Registrations"]
        SIP1["user@domain:5060<br/>Contact: Device1<br/>Expires: 3600s"]
        SIP2["user@domain:5060<br/>Contact: Device2<br/>Expires: 3600s"]
    end

    subgraph IncomingCall["Incoming Call Handling"]
        CallReceived[Incoming Call<br/>to User's Number]
        RingAll[Ring All Active<br/>Devices]
        FirstAnswer[First Device<br/>to Answer Wins]
        CancelOthers[Cancel Ring on<br/>Other Devices]
    end

    UserID --> Device1
    UserID --> Device2
    UserID --> Device3

    Device1 --> SIP1
    Device2 --> SIP2
    Device3 -.->|Not Registered| SIPRegistrations

    CallReceived --> RingAll
    RingAll --> SIP1
    RingAll --> SIP2
    
    SIP1 --> FirstAnswer
    SIP2 --> FirstAnswer
    
    FirstAnswer --> CancelOthers

    classDef userStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef activeStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef inactiveStyle fill:#ffecb3,stroke:#f57f17,stroke-width:2px
    classDef callStyle fill:#f3e5f5,stroke:#4a148c,stroke-width:2px

    class UserID userStyle
    class Device1,Device2,SIP1,SIP2 activeStyle
    class Device3 inactiveStyle
    class CallReceived,RingAll,FirstAnswer,CancelOthers callStyle
```

## Security Features

### JWT Token
- **Expiry**: 1 hour (configurable)
- **Refresh Token**: 30 days
- **Algorithm**: RS256 (asymmetric)
- **Claims**: user_id, device_id, issued_at, expires_at

### SIP Credentials
- **Generated per device**: Unique credentials for each device
- **Stored securely**: Hashed in database
- **Auto-rotation**: Optional periodic rotation
- **Revocation**: Immediate on logout/device removal

### PIN Security
- **Local verification**: PIN never sent to server
- **Secure storage**: Keychain (iOS) / Keystore (Android)
- **Biometric option**: Face ID / Touch ID / Fingerprint
- **Attempt limiting**: Lock after failed attempts

### Device Pairing
- **Time-limited tokens**: 24-hour expiry
- **One-time use**: Token invalidated after use
- **Encrypted transfer**: AES-256 encryption
- **Audit logging**: All transfers logged
