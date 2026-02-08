# High-Level System Architecture

## Overview
This diagram shows the complete system architecture with all major layers and components.

## Architecture Diagram

```mermaid
graph TB
    subgraph CLIENT["Client Layer"]
        Android["Android App<br/>(Native/Flutter)"]
        iOS["iOS App<br/>(Native/Flutter)"]
        WebPortal["Web User Portal<br/>(Vue.js/React)"]
        AdminPanel["Admin Panel<br/>(Vue.js/React)"]
        Marketing["Marketing Site<br/>(Vue.js/React)"]
    end

    subgraph LB["Load Balancing"]
        Nginx["Nginx Load Balancer<br/>SSL Termination"]
    end

    subgraph APP["Application Layer"]
        Laravel["Laravel Backend API"]
        
        subgraph Modules["Core Modules"]
            Auth["Auth Module"]
            User["User Module"]
            Number["Number Module"]
            Billing["Billing Module"]
            Webhook["Webhook Handler"]
            CallRules["Call Rules"]
            SMSMgr["SMS Manager"]
            MMSMgr["MMS Manager"]
            Voicemail["Voicemail"]
            Analytics["Analytics"]
        end
    end

    subgraph COMM["Communication Layer"]
        VoIP["VoIP Server<br/>(Kamailio/FreeSWITCH)"]
        SMSGateway["SMS/MMS Gateway<br/>(Jasmin/Custom)"]
        PushServer["Push Notification<br/>Server (Self-hosted)"]
    end

    subgraph CARRIER["Carrier Layer"]
        SIPTrunk["SIP Trunk Providers<br/>(Bandwidth, Telnyx)"]
        SMPP["SMPP Connection<br/>(Carriers)"]
        DIDProvider["Number Provisioning<br/>(Bandwidth, Telnyx, Plivo)"]
    end

    subgraph PSTN["PSTN Network"]
        CellPhones["Cell Phones"]
        Landlines["Landlines"]
        OtherVoIP["Other VoIP Services"]
    end

    subgraph DATA["Data Layer"]
        MySQL["MySQL Database"]
        Redis["Redis Cache/Queue"]
        S3["S3/MinIO Storage"]
    end

    %% Client to Load Balancer
    Android --> Nginx
    iOS --> Nginx
    WebPortal --> Nginx
    AdminPanel --> Nginx
    Marketing --> Nginx

    %% Load Balancer to Application
    Nginx --> Laravel

    %% Laravel to Modules
    Laravel --> Modules

    %% Application to Communication
    Laravel --> VoIP
    Laravel --> SMSGateway
    Laravel --> PushServer

    %% Application to Data
    Laravel --> MySQL
    Laravel --> Redis
    Laravel --> S3

    %% Communication to Carrier
    VoIP --> SIPTrunk
    SMSGateway --> SMPP
    Laravel --> DIDProvider

    %% Carrier to PSTN
    SIPTrunk --> PSTN
    SMPP --> PSTN

    %% PSTN Components
    PSTN --> CellPhones
    PSTN --> Landlines
    PSTN --> OtherVoIP

    classDef clientStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef appStyle fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef commStyle fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef carrierStyle fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px
    classDef pstnStyle fill:#fce4ec,stroke:#880e4f,stroke-width:2px
    classDef dataStyle fill:#fff9c4,stroke:#f57f17,stroke-width:2px

    class Android,iOS,WebPortal,AdminPanel,Marketing clientStyle
    class Laravel,Modules,Auth,User,Number,Billing,Webhook,CallRules,SMSMgr,MMSMgr,Voicemail,Analytics appStyle
    class VoIP,SMSGateway,PushServer commStyle
    class SIPTrunk,SMPP,DIDProvider carrierStyle
    class PSTN,CellPhones,Landlines,OtherVoIP pstnStyle
    class MySQL,Redis,S3 dataStyle
```

## Layer Descriptions

### Client Layer
- **Mobile Apps**: Native or Flutter apps for Android and iOS with SIP client integration
- **Web Portal**: User-facing web application for account management
- **Admin Panel**: Administrative interface for platform management
- **Marketing Site**: Public-facing website for customer acquisition

### Application Layer
- **Laravel Backend**: RESTful API server handling all business logic
- **Core Modules**: Modular architecture for different features

### Communication Layer
- **VoIP Server**: Handles SIP signaling and media processing
- **SMS/MMS Gateway**: Manages text messaging through SMPP protocol
- **Push Notifications**: Self-hosted push notification service

### Carrier Layer
- **SIP Trunks**: Connect to PSTN for voice calls
- **SMPP Connections**: Direct carrier connections for SMS/MMS
- **DID Providers**: Number provisioning and management

### Data Layer
- **MySQL**: Primary relational database
- **Redis**: Caching and queue management
- **S3/MinIO**: Object storage for media files

## Key Features

1. **Fully In-House**: No dependency on Twilio or Firebase
2. **Scalable**: Horizontal scaling at each layer
3. **Secure**: TLS/SRTP encryption throughout
4. **White-Label Ready**: Complete customization support
