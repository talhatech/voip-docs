# API Structure

## Overview
Complete API endpoint organization and routing structure for the VoIP platform.

## API Architecture

```mermaid
graph TB
    subgraph Client["Client Applications"]
        MobileApp[Mobile Apps]
        WebApp[Web Portal]
        AdminApp[Admin Panel]
    end
    
    subgraph Gateway["API Gateway"]
        Nginx[Nginx<br/>Load Balancer]
        RateLimit[Rate Limiter]
        Auth[Auth Middleware]
    end
    
    subgraph APIVersions["API Versions"]
        V1[API v1<br/>/api/v1]
        V2[API v2<br/>/api/v2]
    end
    
    subgraph PublicRoutes["Public Routes"]
        AuthRoutes[Authentication<br/>/auth/*]
        MarketingRoutes[Marketing<br/>/public/*]
    end
    
    subgraph UserRoutes["User Routes (Authenticated)"]
        UserMgmt[User Management<br/>/user/*]
        Numbers[Phone Numbers<br/>/numbers/*]
        Calls[Calls<br/>/calls/*]
        Messages[SMS/MMS<br/>/sms/*, /mms/*]
        Voicemail[Voicemail<br/>/voicemails/*]
        Billing[Billing<br/>/wallet/*, /transactions/*]
        Porting[Porting<br/>/porting/*]
        Device[Device<br/>/device/*]
    end
    
    subgraph AdminRoutes["Admin Routes (Admin Only)"]
        UserAdmin[User Management<br/>/admin/users/*]
        NumberAdmin[Number Inventory<br/>/admin/numbers/*]
        PortingAdmin[Porting Requests<br/>/admin/porting/*]
        Analytics[Analytics<br/>/admin/analytics/*]
        Pricing[Pricing<br/>/admin/pricing/*]
    end
    
    subgraph WebhookRoutes["Webhook Routes (Internal)"]
        CallWebhook[Call Webhooks<br/>/webhooks/call/*]
        SMSWebhook[SMS Webhooks<br/>/webhooks/sms/*]
        MMSWebhook[MMS Webhooks<br/>/webhooks/mms/*]
    end
    
    MobileApp --> Nginx
    WebApp --> Nginx
    AdminApp --> Nginx
    
    Nginx --> RateLimit
    RateLimit --> Auth
    
    Auth --> V1
    Auth -.->|Future| V2
    
    V1 --> PublicRoutes
    V1 --> UserRoutes
    V1 --> AdminRoutes
    V1 --> WebhookRoutes

    classDef clientStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef gatewayStyle fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    classDef versionStyle fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef publicStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef userStyle fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef adminStyle fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    classDef webhookStyle fill:#e0e0e0,stroke:#424242,stroke-width:2px

    class MobileApp,WebApp,AdminApp clientStyle
    class Nginx,RateLimit,Auth gatewayStyle
    class V1,V2 versionStyle
    class AuthRoutes,MarketingRoutes publicStyle
    class UserMgmt,Numbers,Calls,Messages,Voicemail,Billing,Porting,Device userStyle
    class UserAdmin,NumberAdmin,PortingAdmin,Analytics,Pricing adminStyle
    class CallWebhook,SMSWebhook,MMSWebhook webhookStyle
```

## Complete API Endpoint Map

```mermaid
graph LR
    API[API v1<br/>/api/v1] --> Auth[Authentication]
    API --> User[User]
    API --> Numbers[Phone Numbers]
    API --> CallRules[Call Rules]
    API --> Calls[Calls]
    API --> SMS[SMS]
    API --> MMS[MMS]
    API --> Voicemail[Voicemail]
    API --> Billing[Billing]
    API --> Porting[Porting]
    API --> Device[Device]
    API --> Admin[Admin]
    API --> Webhooks[Webhooks]
    
    Auth --> AuthLogin[POST /auth/login]
    Auth --> AuthRegister[POST /auth/register]
    Auth --> AuthLogout[POST /auth/logout]
    Auth --> AuthRefresh[POST /auth/refresh]
    Auth --> AuthPin[POST /auth/pin/verify]
    Auth --> AuthQR[POST /auth/qr/generate<br/>POST /auth/qr/verify]
    
    User --> UserProfile[GET /user/profile<br/>PUT /user/profile]
    User --> UserPassword[PUT /user/password]
    User --> UserPin[PUT /user/pin]
    
    Numbers --> NumbersList[GET /numbers]
    Numbers --> NumbersSearch[GET /numbers/search]
    Numbers --> NumbersPurchase[POST /numbers/purchase]
    Numbers --> NumbersRelease[DELETE /numbers/{id}]
    Numbers --> NumbersPrimary[PUT /numbers/{id}/primary]
    Numbers --> NumbersStatus[PUT /numbers/{id}/status]
    
    CallRules --> RulesList[GET /numbers/{id}/rules]
    CallRules --> RulesCreate[POST /numbers/{id}/rules]
    CallRules --> RulesUpdate[PUT /numbers/{id}/rules/{ruleId}]
    CallRules --> RulesDelete[DELETE /numbers/{id}/rules/{ruleId}]
    
    Calls --> CallsList[GET /calls]
    Calls --> CallsInitiate[POST /calls/initiate]
    Calls --> CallsDetails[GET /calls/{id}]
    
    SMS --> SMSList[GET /sms]
    SMS --> SMSSend[POST /sms/send]
    SMS --> SMSSchedule[POST /sms/schedule]
    SMS --> SMSScheduledList[GET /sms/scheduled]
    SMS --> SMSCancel[DELETE /sms/scheduled/{id}]
    
    MMS --> MMSList[GET /mms]
    MMS --> MMSSend[POST /mms/send]
    MMS --> MMSDetails[GET /mms/{id}]
    
    Voicemail --> VoicemailList[GET /voicemails]
    Voicemail --> VoicemailGet[GET /voicemails/{id}]
    Voicemail --> VoicemailRead[PUT /voicemails/{id}/read]
    Voicemail --> VoicemailDelete[DELETE /voicemails/{id}]
    
    Billing --> WalletBalance[GET /wallet]
    Billing --> WalletRecharge[POST /wallet/recharge]
    Billing --> Transactions[GET /transactions]
    Billing --> Invoices[GET /invoices<br/>GET /invoices/{id}/download]
    
    Porting --> PortingRequest[POST /porting/request]
    Porting --> PortingStatus[GET /porting/status/{id}]
    Porting --> PortingCancel[DELETE /porting/cancel/{id}]
    
    Device --> DeviceRegister[POST /device/register]
    Device --> DeviceUnregister[DELETE /device/{id}]
    Device --> DeviceTransfer[POST /device/transfer/initiate<br/>POST /device/transfer/complete]
    
    Admin --> AdminUsers[GET /admin/users<br/>GET /admin/users/{id}]
    Admin --> AdminNumbers[GET /admin/numbers<br/>POST /admin/numbers/provision]
    Admin --> AdminPorting[GET /admin/porting<br/>POST /admin/porting/{id}/approve]
    Admin --> AdminAnalytics[GET /admin/analytics]
    Admin --> AdminPricing[GET /admin/pricing<br/>PUT /admin/pricing]
    
    Webhooks --> WebhookCall[POST /webhooks/call/status]
    Webhooks --> WebhookSMS[POST /webhooks/sms/inbound<br/>POST /webhooks/sms/status]
    Webhooks --> WebhookMMS[POST /webhooks/mms/inbound]
    Webhooks --> WebhookVoicemail[POST /webhooks/voicemail]

    classDef moduleStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef endpointStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px

    class Auth,User,Numbers,CallRules,Calls,SMS,MMS,Voicemail,Billing,Porting,Device,Admin,Webhooks moduleStyle
```

## Request/Response Flow

```mermaid
sequenceDiagram
    participant Client
    participant Nginx as Nginx LB
    participant Middleware as Middleware Stack
    participant Controller as Controller
    participant Service as Service Layer
    participant Model as Model/DB
    participant Queue as Queue/Jobs

    Client->>Nginx: 1. HTTP Request<br/>POST /api/v1/sms/send
    
    activate Nginx
    Nginx->>Middleware: 2. Route Request
    deactivate Nginx
    
    activate Middleware
    Note over Middleware: 3. Rate Limiting<br/>Check: 100 req/min
    
    Note over Middleware: 4. Authentication<br/>Validate JWT Token
    
    Note over Middleware: 5. Authorization<br/>Check Permissions
    
    Middleware->>Controller: 6. Forward Request
    deactivate Middleware
    
    activate Controller
    Note over Controller: 7. Validate Input<br/>FormRequest Validation
    
    Controller->>Service: 8. Call Service Method<br/>SmsService::send()
    deactivate Controller
    
    activate Service
    Note over Service: 9. Business Logic<br/>- Check balance<br/>- Calculate cost<br/>- Verify number ownership
    
    Service->>Model: 10. Create SMS Record<br/>Sms::create()
    
    activate Model
    Model->>Model: 11. Save to Database
    Model-->>Service: 12. SMS Model
    deactivate Model
    
    Service->>Queue: 13. Dispatch Job<br/>SendSms::dispatch()
    
    activate Queue
    Queue-->>Service: 14. Job Queued
    deactivate Queue
    
    Service-->>Controller: 15. Return Result
    deactivate Service
    
    activate Controller
    Controller-->>Nginx: 16. JSON Response<br/>201 Created
    deactivate Controller
    
    activate Nginx
    Nginx-->>Client: 17. HTTP Response<br/>{id, status, cost}
    deactivate Nginx
    
    Note over Queue: 18. Background Processing<br/>Worker picks up job
    
    Queue->>Service: 19. Execute Job<br/>Send via Jasmin Gateway
```

## Authentication Middleware Flow

```mermaid
graph TB
    Request[Incoming Request] --> CheckHeader{Authorization<br/>Header Present?}
    
    CheckHeader -->|No| Public{Public<br/>Route?}
    CheckHeader -->|Yes| ExtractToken[Extract JWT Token]
    
    Public -->|Yes| AllowAccess[Allow Access]
    Public -->|No| Return401[401 Unauthorized]
    
    ExtractToken --> ValidateToken{Token<br/>Valid?}
    
    ValidateToken -->|No| Return401
    ValidateToken -->|Yes| CheckExpiry{Token<br/>Expired?}
    
    CheckExpiry -->|Yes| Return401
    CheckExpiry -->|No| LoadUser[Load User from DB]
    
    LoadUser --> CheckStatus{User<br/>Active?}
    
    CheckStatus -->|No| Return403[403 Forbidden]
    CheckStatus -->|Yes| CheckPermissions{Has Required<br/>Permissions?}
    
    CheckPermissions -->|No| Return403
    CheckPermissions -->|Yes| AllowAccess
    
    AllowAccess --> NextMiddleware[Next Middleware]
    Return401 --> ErrorResponse[Error Response]
    Return403 --> ErrorResponse

    classDef successStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef errorStyle fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    classDef processStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px

    class AllowAccess,NextMiddleware successStyle
    class Return401,Return403,ErrorResponse errorStyle
    class ExtractToken,ValidateToken,LoadUser processStyle
```

## Rate Limiting Strategy

```mermaid
graph TB
    subgraph RateLimits["Rate Limit Tiers"]
        Public[Public Routes<br/>10 req/min]
        Authenticated[Authenticated<br/>100 req/min]
        Premium[Premium Users<br/>500 req/min]
        Admin[Admin<br/>1000 req/min]
    end
    
    subgraph Implementation["Implementation"]
        Redis[(Redis<br/>Token Bucket)]
        Middleware[Rate Limit<br/>Middleware]
    end
    
    subgraph Response["Response Handling"]
        Allow[Allow Request<br/>200 OK]
        Throttle[Throttle<br/>429 Too Many Requests]
        Headers[Rate Limit Headers<br/>X-RateLimit-*]
    end
    
    Public --> Middleware
    Authenticated --> Middleware
    Premium --> Middleware
    Admin --> Middleware
    
    Middleware --> Redis
    
    Redis --> Allow
    Redis --> Throttle
    
    Allow --> Headers
    Throttle --> Headers

    classDef limitStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef implStyle fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef responseStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px

    class Public,Authenticated,Premium,Admin limitStyle
    class Redis,Middleware implStyle
    class Allow,Throttle,Headers responseStyle
```

## API Response Format

### Success Response
```json
{
  "success": true,
  "data": {
    "id": 123,
    "number": "+1234567890",
    "status": "active"
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "request_id": "req_abc123"
  }
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "INSUFFICIENT_BALANCE",
    "message": "Insufficient wallet balance",
    "details": {
      "required": 5.00,
      "available": 2.50
    }
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "request_id": "req_abc123"
  }
}
```

### Paginated Response
```json
{
  "success": true,
  "data": [...],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total": 150,
    "total_pages": 8,
    "next_page": 2,
    "prev_page": null
  }
}
```

## HTTP Status Codes

| Code | Usage |
|------|-------|
| **200** | Successful GET, PUT, PATCH, DELETE |
| **201** | Successful POST (resource created) |
| **204** | Successful DELETE (no content) |
| **400** | Bad Request (invalid input) |
| **401** | Unauthorized (missing/invalid auth) |
| **403** | Forbidden (insufficient permissions) |
| **404** | Not Found |
| **409** | Conflict (duplicate resource) |
| **422** | Unprocessable Entity (validation error) |
| **429** | Too Many Requests (rate limited) |
| **500** | Internal Server Error |
| **503** | Service Unavailable |

## API Versioning Strategy

### URL Versioning
- Current: `/api/v1/...`
- Future: `/api/v2/...`
- Maintain backward compatibility for 1 year
- Deprecation warnings in response headers

### Version Lifecycle
1. **Active**: Current production version
2. **Deprecated**: Old version, still supported
3. **Sunset**: Version removed

## Documentation

- **OpenAPI/Swagger**: Auto-generated from code
- **Postman Collection**: Available for download
- **Interactive Docs**: Swagger UI at `/api/documentation`
- **Code Examples**: Available in multiple languages
