# SMS/MMS Workflow

## Overview
This diagram shows how SMS and MMS messages are processed through the platform.

## SMS Sending Workflow

```mermaid
graph TB
    Start([User Sends SMS]) --> Validate{Validate Request}
    
    Validate -->|Invalid| Error1[Return Error]
    Validate -->|Valid| CheckNumber{Verify Sender<br/>Number Ownership}
    
    CheckNumber -->|Not Owned| Error2[Unauthorized Error]
    CheckNumber -->|Owned| CheckBalance{Check Wallet<br/>Balance}
    
    CheckBalance -->|Insufficient| Error3[Insufficient Funds]
    CheckBalance -->|Sufficient| CalcCost[Calculate Cost<br/>Based on Segments]
    
    CalcCost --> CreateRecord[Create SMS Record<br/>Status: queued]
    
    CreateRecord --> Scheduled{Scheduled<br/>for Later?}
    
    Scheduled -->|Yes| ScheduleJob[Add to Scheduled<br/>Jobs Table]
    Scheduled -->|No| QueueImmediate[Queue for<br/>Immediate Sending]
    
    ScheduleJob --> WaitTime[Wait Until<br/>Scheduled Time]
    WaitTime --> QueueImmediate
    
    QueueImmediate --> Redis[(Redis Queue)]
    
    Redis --> Worker[Queue Worker<br/>Processes Job]
    
    Worker --> JasminGW[Jasmin SMS<br/>Gateway]
    
    JasminGW --> SelectCarrier{Select Best<br/>Carrier Route}
    
    SelectCarrier --> Carrier1[Carrier 1<br/>SMPP]
    SelectCarrier --> Carrier2[Carrier 2<br/>SMPP]
    SelectCarrier --> Carrier3[Carrier 3<br/>HTTP API]
    
    Carrier1 --> SendSMS[Send to SMPP<br/>Network]
    Carrier2 --> SendSMS
    Carrier3 --> SendSMS
    
    SendSMS --> UpdateStatus[Update Status:<br/>sent]
    
    UpdateStatus --> DeductWallet[Deduct from<br/>Wallet]
    
    DeductWallet --> CreateTransaction[Create Transaction<br/>Record]
    
    CreateTransaction --> WaitDelivery[Wait for Delivery<br/>Receipt]
    
    WaitDelivery --> DeliveryWebhook{Delivery<br/>Receipt}
    
    DeliveryWebhook -->|Delivered| UpdateDelivered[Status: delivered]
    DeliveryWebhook -->|Failed| UpdateFailed[Status: failed]
    
    UpdateDelivered --> NotifyUser[Push Notification<br/>to User]
    UpdateFailed --> RefundUser[Refund to Wallet]
    
    RefundUser --> NotifyUser
    NotifyUser --> End([Complete])

    classDef errorStyle fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    classDef processStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef decisionStyle fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    classDef queueStyle fill:#e1bee7,stroke:#6a1b9a,stroke-width:2px

    class Error1,Error2,Error3,UpdateFailed,RefundUser errorStyle
    class CreateRecord,Worker,JasminGW,SendSMS,UpdateStatus,DeductWallet,CreateTransaction,UpdateDelivered processStyle
    class Validate,CheckNumber,CheckBalance,Scheduled,SelectCarrier,DeliveryWebhook decisionStyle
    class Redis,QueueImmediate,ScheduleJob queueStyle
```

## SMS Receiving Workflow

```mermaid
sequenceDiagram
    participant Carrier as SMS Carrier
    participant Jasmin as Jasmin Gateway
    participant Redis as Redis Queue
    participant Worker as Queue Worker
    participant Laravel as Laravel API
    participant DB as Database
    participant App as Mobile App

    Carrier->>Jasmin: 1. Incoming SMS<br/>via SMPP
    
    activate Jasmin
    Note over Jasmin: 2. Parse SMPP PDU<br/>Extract Message Data
    
    Jasmin->>Redis: 3. Queue Inbound<br/>Message
    deactivate Jasmin
    
    activate Redis
    Redis->>Worker: 4. Dispatch Job
    deactivate Redis
    
    activate Worker
    Worker->>Laravel: 5. POST /webhooks/sms/inbound<br/>Message Data
    deactivate Worker
    
    activate Laravel
    Note over Laravel: 6. Validate Webhook<br/>Signature
    
    Laravel->>DB: 7. Find Phone Number<br/>Owner
    
    DB-->>Laravel: 8. User Details
    
    Laravel->>DB: 9. Create SMS Record<br/>Direction: inbound<br/>Status: received
    
    Laravel->>App: 10. Push Notification<br/>New Message
    
    App-->>Laravel: 11. Acknowledge
    
    Laravel-->>Worker: 12. 200 OK
    deactivate Laravel
    
    Note over App: User Reads Message
```

## MMS Sending Workflow

```mermaid
graph LR
    Start([User Sends MMS]) --> Upload[Upload Media<br/>to S3/MinIO]
    
    Upload --> Validate{Validate<br/>Media Type<br/>& Size}
    
    Validate -->|Invalid| Error[Return Error]
    Validate -->|Valid| CreateMMS[Create MMS Record]
    
    CreateMMS --> CreateMedia[Create MMS Media<br/>Records]
    
    CreateMedia --> CalcCost[Calculate Cost<br/>Higher than SMS]
    
    CalcCost --> CheckBalance{Check<br/>Balance}
    
    CheckBalance -->|Insufficient| Error
    CheckBalance -->|Sufficient| Queue[Queue for Sending]
    
    Queue --> Worker[Queue Worker]
    
    Worker --> Jasmin[Jasmin Gateway]
    
    Jasmin --> SendMMS[Send MMS via<br/>SMPP/HTTP]
    
    SendMMS --> Deduct[Deduct from<br/>Wallet]
    
    Deduct --> Complete([Complete])

    classDef errorStyle fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    classDef processStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px

    class Error errorStyle
    class Upload,CreateMMS,CreateMedia,Worker,Jasmin,SendMMS,Deduct processStyle
```

## Message Queue Architecture

```mermaid
graph TB
    subgraph Laravel["Laravel Application"]
        SmsService[SMS Service]
        MmsService[MMS Service]
        SchedulerService[Scheduler Service]
    end

    subgraph Queue["Message Queue (Redis)"]
        ImmediateQueue[Immediate Queue]
        ScheduledQueue[Scheduled Queue]
        InboundQueue[Inbound Queue]
    end

    subgraph Workers["Queue Workers"]
        Worker1[Worker 1]
        Worker2[Worker 2]
        Worker3[Worker 3]
    end

    subgraph Gateway["Jasmin SMS Gateway"]
        SMPPServer[SMPP Server]
        HTTPConnector[HTTP Connector]
        Router[Message Router]
    end

    subgraph Carriers["Carriers"]
        Carrier1[Carrier 1<br/>SMPP]
        Carrier2[Carrier 2<br/>SMPP]
        Carrier3[Carrier 3<br/>HTTP API]
    end

    SmsService --> ImmediateQueue
    MmsService --> ImmediateQueue
    SchedulerService --> ScheduledQueue
    
    ImmediateQueue --> Worker1
    ImmediateQueue --> Worker2
    ScheduledQueue --> Worker3
    
    Worker1 --> SMPPServer
    Worker2 --> SMPPServer
    Worker3 --> SMPPServer
    
    SMPPServer --> Router
    HTTPConnector --> Router
    
    Router --> Carrier1
    Router --> Carrier2
    Router --> Carrier3
    
    Carrier1 -.->|Delivery Receipt| SMPPServer
    Carrier2 -.->|Delivery Receipt| SMPPServer
    Carrier3 -.->|Delivery Receipt| HTTPConnector
    
    SMPPServer -.->|Webhook| InboundQueue
    HTTPConnector -.->|Webhook| InboundQueue
    
    InboundQueue -.-> Worker1
    InboundQueue -.-> Worker2

    classDef serviceStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef queueStyle fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef workerStyle fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef gatewayStyle fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px
    classDef carrierStyle fill:#fce4ec,stroke:#880e4f,stroke-width:2px

    class SmsService,MmsService,SchedulerService serviceStyle
    class ImmediateQueue,ScheduledQueue,InboundQueue queueStyle
    class Worker1,Worker2,Worker3 workerStyle
    class SMPPServer,HTTPConnector,Router gatewayStyle
    class Carrier1,Carrier2,Carrier3 carrierStyle
```

## Key Features

### SMS Features
1. **Scheduled Sending**: Queue messages for future delivery
2. **Segment Calculation**: Automatic handling of long messages
3. **Delivery Tracking**: Real-time delivery status updates
4. **Cost Optimization**: Intelligent carrier routing

### MMS Features
1. **Media Upload**: Support for images, videos, audio
2. **Size Validation**: Automatic file size checking
3. **Format Support**: Multiple media formats
4. **Higher Pricing**: Separate pricing model from SMS

### Queue Management
1. **Redis-based**: Fast, reliable message queuing
2. **Multiple Workers**: Parallel message processing
3. **Retry Logic**: Automatic retry on failures
4. **Priority Queues**: Immediate vs scheduled handling
