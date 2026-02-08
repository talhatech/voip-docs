# VoIP Platform - System Architecture & Technical Understanding Document

## 1. Executive Overview

This document outlines the technical architecture for building a **secure, scalable VoIP communication platform** using Laravel as the backend framework. The platform will enable users to make/receive phone calls to/from regular cell phones and landlines, send/receive SMS and MMS, manage multiple virtual phone numbers, and handle voicemail - all built in-house without relying on third-party services like Twilio.

---

## 2. Core Requirements Summary

### 2.1 Functional Requirements

| Feature | Description |
|---------|-------------|
| **Voice Calls** | Make and receive calls to/from cell phones and landlines worldwide |
| **SMS/MMS** | Send and receive text messages and multimedia messages |
| **Voicemail** | Receive and store voicemail messages |
| **Multi-Number Support** | Users can activate multiple phone numbers (like eSIM), enable/disable them, and use 1-5 numbers simultaneously |
| **Call Forwarding** | Forward incoming calls to other numbers |
| **Number Porting** | Port existing numbers in/out with admin verification workflow (3-5 business days) |
| **Caller ID** | Dynamic caller ID selection per call/message |
| **Scheduled SMS** | Schedule SMS messages for future delivery |
| **White-Labeling** | Complete branding customization (UI theme, app identity) |

### 2.2 Non-Functional Requirements

| Requirement | Specification |
|-------------|---------------|
| **Security** | TLS + Encrypted VoIP (SRTP), no E2EE required, VAPT before launch |
| **Data Storage** | All data on device, encrypted backup for device transfers (vault-style) |
| **Compliance** | GDPR compliant (HIPAA not required) |
| **Infrastructure** | Fully in-house (no Twilio/Firebase) |
| **Regions** | Must work reliably in restricted VoIP regions (GCC) with optimized routing |

---

## 3. High-Level System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                    CLIENT LAYER                                      │
├─────────────────────┬─────────────────────┬─────────────────────┬───────────────────┬─────────────────┤
│   Android App       │     iOS App         │   Web User Portal   │  Admin Panel      │ Marketing Site  │
│   (Native/Flutter)  │   (Native/Flutter)  │   (Vue.js/React)    │  (Vue.js/React)   │ (Vue.js/React)  │
└─────────┬───────────┴─────────┬───────────┴─────────┬───────────┴─────────┬─────────┴─────────┬─────┘
          │                     │                     │                     │
          └─────────────────────┴──────────┬──────────┴─────────────────────┘
                                           │
                                    ┌──────▼──────┐
                                    │   Nginx     │
                                    │ Load Balancer│
                                    └──────┬──────┘
                                           │
┌──────────────────────────────────────────┼──────────────────────────────────────────┐
│                              APPLICATION LAYER                                       │
│  ┌───────────────────────────────────────┼───────────────────────────────────────┐  │
│  │                         Laravel Backend (API)                                  │  │
│  │  ┌─────────────┬─────────────┬─────────────┬─────────────┬─────────────────┐  │  │
│  │  │ Auth Module │ User Module │Number Module│Billing Module│ Webhook Handler │  │  │
│  │  └─────────────┴─────────────┴─────────────┴─────────────┴─────────────────┘  │  │
│  │  ┌─────────────┬─────────────┬─────────────┬─────────────┬─────────────────┐  │  │
│  │  │ Call Rules  │ SMS Manager │ MMS Manager │ Voicemail   │ Analytics       │  │  │
│  │  └─────────────┴─────────────┴─────────────┴─────────────┴─────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────┬──────────────────────────────────────────┘
                                           │
┌──────────────────────────────────────────┼──────────────────────────────────────────┐
│                            COMMUNICATION LAYER                                       │
│  ┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────────────┐  │
│  │   VoIP Server       │  │   SMS/MMS Gateway   │  │   Push Notification Server  │  │
│  │   (Kamailio/        │  │   (Jasmin SMS GW/   │  │   (Self-hosted)             │  │
│  │    FreeSWITCH)      │  │    Custom Gateway)  │  │                             │  │
│  └──────────┬──────────┘  └──────────┬──────────┘  └─────────────────────────────┘  │
└─────────────┼────────────────────────┼──────────────────────────────────────────────┘
              │                        │
┌─────────────┼────────────────────────┼──────────────────────────────────────────────┐
│             │    CARRIER LAYER       │                                               │
│  ┌──────────▼──────────┐  ┌──────────▼──────────┐  ┌─────────────────────────────┐  │
│  │   SIP Trunk         │  │   SMPP Connection   │  │   Number Provisioning       │  │
│  │   Providers         │  │   (Carriers)        │  │   API (DID Providers)       │  │
│  │   (Bandwidth,       │  │                     │  │   (Bandwidth, Telnyx,       │  │
│  │    Telnyx, etc)     │  │                     │  │    Plivo DID)               │  │
│  └─────────────────────┘  └─────────────────────┘  └─────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────────┘
                                           │
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                    PSTN NETWORK                                      │
│                    (Cell Phones, Landlines, Other VoIP Services)                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Component Breakdown

### 4.1 Laravel Backend Architecture

```
app/
├── Http/
│   ├── Controllers/
│   │   ├── Api/
│   │   │   ├── V1/
│   │   │   │   ├── AuthController.php
│   │   │   │   ├── UserController.php
│   │   │   │   ├── PhoneNumberController.php
│   │   │   │   ├── CallController.php
│   │   │   │   ├── SmsController.php
│   │   │   │   ├── MmsController.php
│   │   │   │   ├── VoicemailController.php
│   │   │   │   ├── ScheduledSmsController.php
│   │   │   │   ├── CallRuleController.php
│   │   │   │   ├── BillingController.php
│   │   │   │   └── WebhookController.php
│   │   │   └── Admin/
│   │   │       ├── UserManagementController.php
│   │   │       ├── NumberInventoryController.php
│   │   │       ├── PricingController.php
│   │   │       └── AnalyticsController.php
│   │   └── Middleware/
│   │       ├── ApiAuthentication.php
│   │       ├── RateLimiter.php
│   │       └── EncryptionMiddleware.php
│   └── Requests/
├── Models/
│   ├── User.php
│   ├── PhoneNumber.php
│   ├── Call.php
│   ├── Sms.php
│   ├── Mms.php
│   ├── Voicemail.php
│   ├── CallRule.php
│   ├── Transaction.php
│   └── Wallet.php
├── Services/
│   ├── VoIP/
│   │   ├── SipService.php
│   │   ├── CallRoutingService.php
│   │   └── VoicemailService.php
│   ├── Messaging/
│   │   ├── SmsService.php
│   │   ├── MmsService.php
│   │   └── SmppGatewayService.php
│   ├── NumberProvisioning/
│   │   ├── NumberSearchService.php
│   │   ├── NumberPurchaseService.php
│   │   └── PortingService.php
│   ├── Billing/
│   │   ├── WalletService.php
│   │   ├── PaymentGatewayService.php
│   │   └── InvoiceService.php
│   └── Security/
│       ├── EncryptionService.php
│       └── TokenService.php
├── Jobs/
│   ├── ProcessIncomingCall.php
│   ├── ProcessOutgoingCall.php
│   ├── SendSms.php
│   ├── SendMms.php
│   ├── ProcessVoicemail.php
│   └── SyncDeviceData.php
└── Events/
    ├── CallStarted.php
    ├── CallEnded.php
    ├── SmsReceived.php
    └── VoicemailReceived.php
```

### 4.2 Database Schema Design

```sql
-- Users Table
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    uuid UUID UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    password VARCHAR(255) NOT NULL,
    pin VARCHAR(6),
    status ENUM('active', 'suspended', 'deleted') DEFAULT 'active',
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- Phone Numbers (Virtual Numbers owned by users)
CREATE TABLE phone_numbers (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT REFERENCES users(id),
    number VARCHAR(20) UNIQUE NOT NULL,
    country_code VARCHAR(5) NOT NULL,
    type ENUM('local', 'toll_free', 'mobile') DEFAULT 'local',
    status ENUM('active', 'inactive', 'porting', 'released') DEFAULT 'active',
    is_primary BOOLEAN DEFAULT FALSE,
    capabilities JSON, -- {voice: true, sms: true, mms: true}
    monthly_cost DECIMAL(10,2),
    purchased_at TIMESTAMP,
    expires_at TIMESTAMP,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- Call Records
CREATE TABLE calls (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    uuid UUID UNIQUE NOT NULL,
    user_id BIGINT REFERENCES users(id),
    phone_number_id BIGINT REFERENCES phone_numbers(id),
    direction ENUM('inbound', 'outbound') NOT NULL,
    from_number VARCHAR(20) NOT NULL,
    to_number VARCHAR(20) NOT NULL,
    status ENUM('initiated', 'ringing', 'answered', 'completed', 'failed', 'busy', 'no_answer') NOT NULL,
    duration_seconds INT DEFAULT 0,
    cost DECIMAL(10,4),
    recording_url VARCHAR(500),
    call_sid VARCHAR(100), -- SIP Call ID
    started_at TIMESTAMP,
    answered_at TIMESTAMP,
    ended_at TIMESTAMP,
    metadata JSON,
    created_at TIMESTAMP
);

-- SMS Messages
CREATE TABLE sms_messages (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    uuid UUID UNIQUE NOT NULL,
    user_id BIGINT REFERENCES users(id),
    phone_number_id BIGINT REFERENCES phone_numbers(id),
    direction ENUM('inbound', 'outbound') NOT NULL,
    from_number VARCHAR(20) NOT NULL,
    to_number VARCHAR(20) NOT NULL,
    body TEXT NOT NULL,
    segments INT DEFAULT 1,
    status ENUM('queued', 'sent', 'delivered', 'failed', 'received') NOT NULL,
    cost DECIMAL(10,4),
    sent_at TIMESTAMP,
    delivered_at TIMESTAMP,
    scheduled_at TIMESTAMP,
    scheduled_status ENUM('pending', 'sent', 'cancelled') DEFAULT NULL,
    metadata JSON,
    created_at TIMESTAMP
);

-- MMS Messages
CREATE TABLE mms_messages (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    uuid UUID UNIQUE NOT NULL,
    user_id BIGINT REFERENCES users(id),
    phone_number_id BIGINT REFERENCES phone_numbers(id),
    direction ENUM('inbound', 'outbound') NOT NULL,
    from_number VARCHAR(20) NOT NULL,
    to_number VARCHAR(20) NOT NULL,
    body TEXT,
    status ENUM('queued', 'sent', 'delivered', 'failed', 'received') NOT NULL,
    cost DECIMAL(10,4),
    sent_at TIMESTAMP,
    delivered_at TIMESTAMP,
    metadata JSON,
    created_at TIMESTAMP
);

-- MMS Media Attachments
CREATE TABLE mms_media (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    mms_message_id BIGINT REFERENCES mms_messages(id),
    media_url VARCHAR(500) NOT NULL,
    media_type VARCHAR(100) NOT NULL,
    file_size INT,
    created_at TIMESTAMP
);

-- Voicemails
CREATE TABLE voicemails (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    uuid UUID UNIQUE NOT NULL,
    user_id BIGINT REFERENCES users(id),
    phone_number_id BIGINT REFERENCES phone_numbers(id),
    from_number VARCHAR(20) NOT NULL,
    duration_seconds INT DEFAULT 0,
    audio_url VARCHAR(500) NOT NULL,
    transcription TEXT,
    is_read BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP
);

-- Call Rules (Forwarding, DND, etc.)
CREATE TABLE call_rules (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    phone_number_id BIGINT REFERENCES phone_numbers(id),
    rule_type ENUM('forward_always', 'forward_busy', 'forward_no_answer', 'forward_unreachable', 'dnd', 'schedule') NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    forward_to VARCHAR(20),
    timeout_seconds INT DEFAULT 30,
    schedule JSON, -- {days: [1,2,3,4,5], start_time: "09:00", end_time: "17:00"}
    priority INT DEFAULT 0,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- Wallet & Billing
CREATE TABLE wallets (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT UNIQUE REFERENCES users(id),
    balance DECIMAL(12,4) DEFAULT 0,
    currency VARCHAR(3) DEFAULT 'USD',
    auto_recharge BOOLEAN DEFAULT FALSE,
    auto_recharge_amount DECIMAL(10,2),
    auto_recharge_threshold DECIMAL(10,2),
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- Transactions
CREATE TABLE transactions (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    uuid UUID UNIQUE NOT NULL,
    user_id BIGINT REFERENCES users(id),
    wallet_id BIGINT REFERENCES wallets(id),
    type ENUM('credit', 'debit', 'refund') NOT NULL,
    category ENUM('recharge', 'call', 'sms', 'mms', 'number_purchase', 'number_renewal', 'porting') NOT NULL,
    amount DECIMAL(12,4) NOT NULL,
    balance_after DECIMAL(12,4) NOT NULL,
    reference_type VARCHAR(50), -- 'call', 'sms', 'phone_number'
    reference_id BIGINT,
    description VARCHAR(255),
    created_at TIMESTAMP
);

-- Number Porting Requests
CREATE TABLE porting_requests (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT REFERENCES users(id),
    number VARCHAR(20) NOT NULL,
    direction ENUM('port_in', 'port_out') NOT NULL,
    status ENUM('pending', 'documents_uploaded', 'admin_review', 'approved', 'rejected', 'submitted_to_carrier', 'completed', 'cancelled') DEFAULT 'pending',
    carrier_name VARCHAR(100),
    account_number VARCHAR(50),
    pin VARCHAR(20),
    documents JSON, -- Uploaded identity/ownership documents
    admin_notes TEXT,
    admin_approved_by BIGINT REFERENCES users(id),
    admin_approved_at TIMESTAMP,
    submitted_at TIMESTAMP,
    completed_at TIMESTAMP,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- Device Registrations (for SIP/Push)
CREATE TABLE device_registrations (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT REFERENCES users(id),
    device_id VARCHAR(255) NOT NULL,
    device_type ENUM('android', 'ios', 'web') NOT NULL,
    push_token VARCHAR(500),
    sip_username VARCHAR(100),
    sip_password VARCHAR(255),
    is_active BOOLEAN DEFAULT TRUE,
    last_seen_at TIMESTAMP,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    UNIQUE(user_id, device_id)
);
```

---

## 5. VoIP Infrastructure (In-House)

### 5.1 Recommended Stack

Since the requirement is **fully in-house** (no Twilio/Firebase), we need to set up our own VoIP infrastructure:

| Component | Technology | Purpose |
|-----------|------------|---------|
| **SIP Proxy/Registrar** | Kamailio | Handle SIP signaling, registration, routing |
| **Media Server** | FreeSWITCH | Handle actual voice media, codecs, recording |
| **RTP Proxy** | RTPEngine | Handle NAT traversal, SRTP encryption |
| **SMS/MMS Gateway** | Jasmin SMS Gateway | SMPP protocol handling |
| **STUN/TURN Server** | Coturn | NAT traversal for WebRTC/VoIP |

### 5.2 VoIP Call Flow

```
┌─────────────┐                                           ┌─────────────┐
│  Mobile App │                                           │   PSTN      │
│  (SIP Client)                                           │  (Landline/ │
│             │                                           │   Cell)     │
└──────┬──────┘                                           └──────┬──────┘
       │                                                         │
       │ 1. SIP INVITE                                           │
       │    (TLS/WSS)                                            │
       ▼                                                         │
┌──────────────┐                                                 │
│   Kamailio   │ 2. Authenticate & Route                         │
│  SIP Proxy   │─────────────────────────┐                       │
└──────┬───────┘                         │                       │
       │                                 │                       │
       │ 3. Forward to Media Server      │                       │
       ▼                                 │                       │
┌──────────────┐                         │                       │
│  FreeSWITCH  │ 4. Connect Media        │                       │
│ Media Server │◄────────────────────────┘                       │
└──────┬───────┘                                                 │
       │                                                         │
       │ 5. SIP INVITE to SIP Trunk                              │
       ▼                                                         │
┌──────────────┐                                                 │
│  SIP Trunk   │ 6. Connect to PSTN ─────────────────────────────┤
│  (Bandwidth/ │                                                 │
│   Telnyx)    │◄────────────────────────────────────────────────┘
└──────────────┘
```

### 5.3 SIP Trunk Providers (Carrier Layer)

Since we're building in-house infrastructure, we still need **SIP trunk providers** to connect to the PSTN network:

| Provider | Features | Use Case |
|----------|----------|----------|
| **Bandwidth** | Enterprise-grade, good US coverage | Primary carrier |
| **Telnyx** | Global coverage, competitive pricing | International calls |
| **Plivo** | DID provisioning, SMS/MMS | Number inventory |
| **SignalWire** | FreeSWITCH-based, developer-friendly | Backup |

---

## 6. SMS/MMS Infrastructure

### 6.1 Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Laravel Backend                           │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │   SmsService    │  │   MmsService    │  │ SchedulerService │  │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘  │
└───────────┼─────────────────────┼─────────────────────┼──────────┘
            │                     │                     │
            └─────────────────────┼─────────────────────┘
                                  │
                                  ▼
                    ┌─────────────────────────┐
                    │   Message Queue         │
                    │   (Redis/RabbitMQ)      │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │   Jasmin SMS Gateway    │
                    │   (SMPP Server)         │
                    └────────────┬────────────┘
                                 │
            ┌────────────────────┼────────────────────┐
            │                    │                    │
            ▼                    ▼                    ▼
     ┌────────────┐      ┌────────────┐      ┌────────────┐
     │  Carrier 1 │      │  Carrier 2 │      │  Carrier 3 │
     │   (SMPP)   │      │   (SMPP)   │      │  (HTTP API)│
     └────────────┘      └────────────┘      └────────────┘
```

### 6.2 SMS/MMS Workflow

```php
// Laravel SMS Service Example
namespace App\Services\Messaging;

class SmsService
{
    public function send(string $from, string $to, string $message, ?Carbon $scheduledAt = null): Sms
    {
        // 1. Validate sender number belongs to user
        // 2. Check wallet balance
        // 3. Calculate cost based on destination
        // 4. Create SMS record
        // 5. Queue for sending (or schedule)
        // 6. Deduct from wallet on success
        
        $sms = Sms::create([
            'user_id' => auth()->id(),
            'phone_number_id' => $this->getPhoneNumberId($from),
            'direction' => 'outbound',
            'from_number' => $from,
            'to_number' => $to,
            'body' => $message,
            'segments' => $this->calculateSegments($message),
            'status' => $scheduledAt ? 'scheduled' : 'queued',
            'scheduled_at' => $scheduledAt,
        ]);
        
        if (!$scheduledAt) {
            SendSms::dispatch($sms);
        }
        
        return $sms;
    }
}
```

---

## 7. Mobile App Architecture

### 7.1 Recommended Approach

| Option | Pros | Cons |
|--------|------|------|
| **Flutter** | Single codebase, fast development | SIP libraries less mature |
| **Native (Kotlin/Swift)** | Best performance, mature SIP libs | Double development effort |
| **React Native** | Large community, reusable code | Performance concerns for VoIP |

**Recommendation**: **Native apps** for best VoIP performance, or **Flutter** with native SIP bridges.

### 7.2 Mobile App Components

```
┌─────────────────────────────────────────────────────────────┐
│                       Mobile App                             │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  UI Layer   │  │  Business   │  │    Data Layer       │  │
│  │             │  │   Logic     │  │                     │  │
│  │ - Dialer    │  │             │  │ - Local SQLite DB   │  │
│  │ - Call UI   │  │ - Call Mgmt │  │ - Encrypted Storage │  │
│  │ - SMS/MMS   │  │ - SMS/MMS   │  │ - Secure Keychain   │  │
│  │ - Contacts  │  │ - Rules     │  │ - API Client        │  │
│  │ - Settings  │  │ - Auth      │  │                     │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐│
│  │                   SIP Stack (Native)                    ││
│  │  - SRTP encryption    - SRTP support                    ││
│  │  - TLS for signaling  - Codec selection (Opus, G.711)   ││
│  │  - SRTP encryption    - NAT traversal (ICE/STUN/TURN)   ││
│  │  Libraries: OPAL, OPAL, OPAL (iOS), Opal (Android)     ││
│  │  or: OPAL, oLSIP, oLSIP, oLSIP                         ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

### 7.3 Recommended SIP Libraries

| Platform | Library | License |
|----------|---------|---------|
| **Android** | OPAL SIP, oLSIP, oLSIP | Various |
| **iOS** | OPAL, oLSIP, oLSIP | Various |
| **Cross-platform** | oLSIP (with Flutter bindings) | MIT |

**Recommended**: **oLSIP** - Open source, actively maintained, supports oLSIP/oLSIP.

---

## 8. Security Architecture

### 8.1 Encryption Layers

```
┌─────────────────────────────────────────────────────────────┐
│                    Security Architecture                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │               Transport Layer Security               │   │
│  │  • TLS 1.3 for all API communications               │   │
│  │  • TLS/WSS for SIP signaling                        │   │
│  │  • Certificate pinning in mobile apps               │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                 Voice Encryption                     │   │
│  │  • SRTP (Secure Real-time Transport Protocol)       │   │
│  │  • AES-128/256 for media encryption                 │   │
│  │  • DTLS-SRTP key exchange                           │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Application Layer Security              │   │
│  │  • JWT tokens with short expiry                     │   │
│  │  • PIN-based quick access (device-bound)            │   │
│  │  • QR-based device pairing                          │   │
│  │  • Rate limiting on all endpoints                   │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                Data at Rest Encryption               │   │
│  │  • AES-256 for stored data on device                │   │
│  │  • Encrypted SQLite database                        │   │
│  │  • Secure enclave for keys (iOS/Android)            │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 8.2 Authentication Flow

```
┌──────────┐                  ┌──────────┐                  ┌──────────┐
│  Client  │                  │  Laravel │                  │   SIP    │
│   App    │                  │  Backend │                  │  Server  │
└────┬─────┘                  └────┬─────┘                  └────┬─────┘
     │                             │                             │
     │ 1. Login (email/password)   │                             │
     │────────────────────────────►│                             │
     │                             │                             │
     │ 2. JWT Token + SIP Creds    │                             │
     │◄────────────────────────────│                             │
     │                             │                             │
     │ 3. SIP REGISTER (TLS)       │                             │
     │─────────────────────────────┼────────────────────────────►│
     │                             │                             │
     │ 4. SIP 401 (Challenge)      │                             │
     │◄────────────────────────────┼─────────────────────────────│
     │                             │                             │
     │ 5. SIP REGISTER (Auth)      │                             │
     │─────────────────────────────┼────────────────────────────►│
     │                             │                             │
     │                             │ 6. Validate with Laravel    │
     │                             │◄────────────────────────────│
     │                             │                             │
     │                             │ 7. OK                       │
     │                             │────────────────────────────►│
     │                             │                             │
     │ 8. SIP 200 OK               │                             │
     │◄────────────────────────────┼─────────────────────────────│
     │                             │                             │
```

---

## 9. Device Data Sync & Backup

Since all data should be stored on device with transfer capability:

### 9.1 Sync Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    Device Data Architecture                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  LOCAL STORAGE (Primary)                                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Encrypted SQLite Database                           │    │
│  │  • Call history                                      │    │
│  │  • SMS/MMS messages                                  │    │
│  │  • Voicemails (audio files)                         │    │
│  │  • Contacts (if app-managed)                        │    │
│  │  • Settings & preferences                           │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  SYNC SERVICE (Device Transfer)                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Encrypted Backup to Server (Temporary)              │    │
│  │  • User initiates transfer                          │    │
│  │  • Data encrypted with user's key                   │    │
│  │  • Uploaded to secure storage                       │    │
│  │  • New device downloads & decrypts                  │    │
│  │  • Server copy deleted after transfer               │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### 9.2 Transfer Flow

```php
// Device Transfer Service
class DeviceTransferService
{
    public function initiateTransfer(User $user): TransferToken
    {
        // 1. Create encrypted backup of user data
        $backup = $this->createEncryptedBackup($user);
        
        // 2. Upload to temporary secure storage
        $storageUrl = $this->uploadToSecureStorage($backup);
        
        // 3. Generate one-time transfer token (valid 24 hours)
        $token = TransferToken::create([
            'user_id' => $user->id,
            'token' => Str::random(64),
            'storage_url' => $storageUrl,
            'expires_at' => now()->addHours(24),
        ]);
        
        // 4. Generate QR code with token
        return $token;
    }
    
    public function completeTransfer(string $token): bool
    {
        // 1. Validate token
        // 2. Download encrypted backup
        // 3. Decrypt with user's key
        // 4. Import to new device
        // 5. Delete server copy
        // 6. Invalidate token
    }
}
```

---

## 10. Call Rules Engine

### 10.1 Rule Types

| Rule Type | Description |
|-----------|-------------|
| `forward_always` | Always forward calls to another number |
| `forward_busy` | Forward when line is busy |
| `forward_no_answer` | Forward after X seconds of no answer |
| `forward_unreachable` | Forward when device is offline |
| `dnd` | Do Not Disturb - send to voicemail |
| `schedule` | Time-based availability rules |

### 10.2 Rule Processing Flow

```php
class CallRoutingService
{
    public function routeIncomingCall(string $toNumber, string $fromNumber): CallRoute
    {
        $phoneNumber = PhoneNumber::where('number', $toNumber)->firstOrFail();
        $rules = $phoneNumber->callRules()->active()->orderBy('priority')->get();
        
        foreach ($rules as $rule) {
            if ($this->shouldApplyRule($rule, $phoneNumber)) {
                return $this->applyRule($rule, $fromNumber);
            }
        }
        
        // Default: Ring user's devices
        return new CallRoute('ring_devices', $phoneNumber->user);
    }
    
    private function shouldApplyRule(CallRule $rule, PhoneNumber $number): bool
    {
        return match ($rule->rule_type) {
            'forward_always' => true,
            'dnd' => true,
            'schedule' => $this->isWithinSchedule($rule->schedule),
            'forward_busy' => $this->isUserBusy($number->user),
            'forward_unreachable' => !$this->isUserOnline($number->user),
            'forward_no_answer' => false, // Handled during call
            default => false,
        };
    }
}
```

---

## 11. Number Porting Workflow

### 11.1 Porting Process Overview

Number porting is a critical feature allowing users to transfer their existing phone numbers to/from the platform. The process involves user initiation, admin verification, carrier submission, and completion notification.

### 11.2 Port-In Workflow (4-Step Process)

```
┌──────────────────────────────────────────────────────────────────┐
│                    Number Port-In Workflow                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STEP 1: User Initiates Request (Mobile App / Web Portal)       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  • User enters existing phone number                        │ │
│  │  • User provides current carrier details                    │ │
│  │  • User uploads identity/ownership documents                │ │
│  │  • User provides account number and PIN (if required)       │ │
│  │  • Status: pending → documents_uploaded                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              ↓                                   │
│  STEP 2: Admin Verification (Admin Panel)                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  • Admin reviews uploaded documents                         │ │
│  │  • Admin verifies identity and ownership                    │ │
│  │  • Admin checks for completeness and validity               │ │
│  │  • Admin approves or rejects the request                    │ │
│  │  • Status: admin_review → approved / rejected               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              ↓                                   │
│  STEP 3: Submission to Carrier (Backend Automation)             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  • Backend sends porting request to telecom carrier         │ │
│  │  • Carrier processes the port (3-5 business days)           │ │
│  │  • Carrier confirms completion via webhook/API              │ │
│  │  • Status: submitted_to_carrier → completed                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                              ↓                                   │
│  STEP 4: Completion Notification (App / Email / SMS)            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  • User receives notification of successful port            │ │
│  │  • Number is now active on the platform                     │ │
│  │  • User can make/receive calls and messages                 │ │
│  │  • Status: completed                                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### 11.3 Porting Service Implementation

```php
class PortingService
{
    public function initiatePortIn(User $user, array $data): PortingRequest
    {
        // 1. Validate number and carrier details
        // 2. Store uploaded documents securely
        // 3. Create porting request with status 'documents_uploaded'
        
        return PortingRequest::create([
            'user_id' => $user->id,
            'number' => $data['number'],
            'direction' => 'port_in',
            'status' => 'documents_uploaded',
            'carrier_name' => $data['carrier_name'],
            'account_number' => $data['account_number'],
            'pin' => $data['pin'],
            'documents' => $data['documents'], // JSON array of document URLs
        ]);
    }
    
    public function adminApprove(PortingRequest $request, User $admin): bool
    {
        // 1. Verify admin has permission
        // 2. Update status to 'approved'
        // 3. Record admin who approved
        // 4. Trigger submission to carrier
        
        $request->update([
            'status' => 'approved',
            'admin_approved_by' => $admin->id,
            'admin_approved_at' => now(),
        ]);
        
        // Dispatch job to submit to carrier
        SubmitPortingToCarrier::dispatch($request);
        
        return true;
    }
    
    public function submitToCarrier(PortingRequest $request): void
    {
        // 1. Connect to carrier's porting API
        // 2. Submit porting request with all details
        // 3. Update status to 'submitted_to_carrier'
        // 4. Store carrier confirmation/reference number
        
        $request->update([
            'status' => 'submitted_to_carrier',
            'submitted_at' => now(),
        ]);
    }
    
    public function handleCarrierCompletion(PortingRequest $request): void
    {
        // 1. Receive webhook from carrier
        // 2. Activate number on platform
        // 3. Update status to 'completed'
        // 4. Notify user
        
        $request->update([
            'status' => 'completed',
            'completed_at' => now(),
        ]);
        
        // Create phone number record for user
        PhoneNumber::create([
            'user_id' => $request->user_id,
            'number' => $request->number,
            'status' => 'active',
            'is_ported' => true,
        ]);
        
        // Send notification
        $request->user->notify(new NumberPortedSuccessfully($request));
    }
}
```

### 11.4 Admin Panel Features for Porting

- View all porting requests with filtering (pending, approved, rejected, completed)
- Document viewer for uploaded identity/ownership documents
- Approval/rejection workflow with notes
- Carrier submission tracking
- Timeline view of porting progress
- Bulk actions for processing multiple requests

---

## 12. Billing & Payment System

### 11.1 Pricing Model

```
┌─────────────────────────────────────────────────────────────┐
│                      Billing System                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  PHONE NUMBERS                                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  • Monthly subscription per number                   │   │
│  │  • Different rates by country/type                   │   │
│  │  • Auto-renewal from wallet                          │   │
│  │  • Grace period before release                       │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  VOICE CALLS                                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  • Per-minute billing (rounded up)                   │   │
│  │  • Different rates by destination                    │   │
│  │  • Inbound may be free or charged                    │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  SMS/MMS                                                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  • Per-message billing                               │   │
│  │  • Long SMS = multiple segments                      │   │
│  │  • MMS higher rate than SMS                          │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 11.2 Payment Gateways

**Supported Payment Methods:**
- Stripe (Credit/Debit Cards)
- PayPal
- Apple Pay
- Cash App
- Amazon Pay
- Card Payments (Direct)

```php
// Payment Gateway Interface
interface PaymentGatewayInterface
{
    public function charge(float $amount, string $currency, array $paymentMethod): PaymentResult;
    public function refund(string $transactionId, float $amount): RefundResult;
    public function createCustomer(User $user): string;
    public function addPaymentMethod(string $customerId, array $paymentData): string;
}

// Supported Gateways
class StripeGateway implements PaymentGatewayInterface { }
class PayPalGateway implements PaymentGatewayInterface { }
class ApplePayGateway implements PaymentGatewayInterface { }
class CashAppGateway implements PaymentGatewayInterface { }
class AmazonPayGateway implements PaymentGatewayInterface { }

// Auto-Release Logic
class NumberRenewalService
{
    public function handlePaymentFailure(PhoneNumber $number): void
    {
        // 1. Attempt auto-recharge if enabled
        // 2. Send payment failure notification
        // 3. Grace period (e.g., 7 days)
        // 4. Auto-release number if payment not received
        // 5. Notify user of number release
    }
}
```

---

## 12. API Structure

### 12.1 API Endpoints

```
API Routes (api/v1/)

Authentication
├── POST   /auth/register
├── POST   /auth/login
├── POST   /auth/logout
├── POST   /auth/refresh
├── POST   /auth/pin/verify
├── POST   /auth/qr/generate
└── POST   /auth/qr/verify

User
├── GET    /user/profile
├── PUT    /user/profile
├── PUT    /user/password
└── PUT    /user/pin

Phone Numbers
├── GET    /numbers                     # List user's numbers
├── GET    /numbers/search              # Search available numbers
├── POST   /numbers/purchase            # Purchase a number
├── DELETE /numbers/{id}                # Release a number
├── PUT    /numbers/{id}/primary        # Set as primary
├── PUT    /numbers/{id}/status         # Enable/disable
└── GET    /numbers/{id}/capabilities

Call Rules
├── GET    /numbers/{id}/rules
├── POST   /numbers/{id}/rules
├── PUT    /numbers/{id}/rules/{ruleId}
└── DELETE /numbers/{id}/rules/{ruleId}

Calls
├── GET    /calls                       # Call history
├── POST   /calls/initiate              # Initiate outbound call
└── GET    /calls/{id}                  # Call details

SMS
├── GET    /sms                         # SMS history
├── POST   /sms/send                    # Send SMS
├── POST   /sms/schedule                # Schedule SMS
├── GET    /sms/scheduled               # List scheduled SMS
├── DELETE /sms/scheduled/{id}          # Cancel scheduled SMS
└── GET    /sms/{id}                    # SMS details

MMS
├── GET    /mms                         # MMS history
├── POST   /mms/send                    # Send MMS
└── GET    /mms/{id}                    # MMS details

Voicemail
├── GET    /voicemails                  # List voicemails
├── GET    /voicemails/{id}             # Get voicemail
├── PUT    /voicemails/{id}/read        # Mark as read
└── DELETE /voicemails/{id}             # Delete voicemail

Billing
├── GET    /wallet                      # Wallet balance
├── POST   /wallet/recharge             # Add funds
├── GET    /transactions                # Transaction history
├── GET    /invoices                    # Invoice history
└── GET    /invoices/{id}/download      # Download invoice

Porting
├── POST   /porting/request             # Request port-in
├── GET    /porting/status/{id}         # Check porting status
└── DELETE /porting/cancel/{id}         # Cancel porting

Device
├── POST   /device/register             # Register device for push
├── DELETE /device/{id}                 # Unregister device
├── POST   /device/transfer/initiate    # Start device transfer
└── POST   /device/transfer/complete    # Complete transfer

Webhooks (Internal - from VoIP/SMS servers)
├── POST   /webhooks/call/status        # Call status updates
├── POST   /webhooks/sms/inbound        # Incoming SMS
├── POST   /webhooks/sms/status         # SMS delivery status
├── POST   /webhooks/mms/inbound        # Incoming MMS
└── POST   /webhooks/voicemail          # New voicemail
```

---

## 13. Infrastructure Requirements

### 13.1 Server Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Production Infrastructure                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                     Load Balancer (Nginx)                     │  │
│  │                     SSL Termination                           │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                              │                                      │
│         ┌────────────────────┼────────────────────┐                │
│         │                    │                    │                │
│         ▼                    ▼                    ▼                │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐          │
│  │ Laravel App │     │ Laravel App │     │ Laravel App │          │
│  │  Server 1   │     │  Server 2   │     │  Server N   │          │
│  └─────────────┘     └─────────────┘     └─────────────┘          │
│                              │                                      │
│         ┌────────────────────┼────────────────────┐                │
│         │                    │                    │                │
│         ▼                    ▼                    ▼                │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐          │
│  │   MySQL     │     │    Redis    │     │  S3/MinIO   │          │
│  │  (Primary)  │     │   Cluster   │     │  (Storage)  │          │
│  └─────────────┘     └─────────────┘     └─────────────┘          │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                      VoIP Infrastructure                      │  │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐              │  │
│  │  │  Kamailio  │  │ FreeSWITCH │  │  RTPEngine │              │  │
│  │  │  (HA Pair) │  │  (Cluster) │  │  (Cluster) │              │  │
│  │  └────────────┘  └────────────┘  └────────────┘              │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                     SMS/MMS Infrastructure                    │  │
│  │  ┌────────────────┐  ┌────────────────┐                      │  │
│  │  │  Jasmin SMS    │  │  Message Queue │                      │  │
│  │  │   Gateway      │  │   (RabbitMQ)   │                      │  │
│  │  └────────────────┘  └────────────────┘                      │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 13.2 Estimated Server Requirements

| Component | Min Specs | Recommended |
|-----------|-----------|-------------|
| **Laravel API** (each) | 4 vCPU, 8GB RAM | 8 vCPU, 16GB RAM |
| **MySQL Primary** | 4 vCPU, 16GB RAM | 8 vCPU, 32GB RAM |
| **Redis** | 2 vCPU, 4GB RAM | 4 vCPU, 8GB RAM |
| **Kamailio** (each) | 4 vCPU, 8GB RAM | 8 vCPU, 16GB RAM |
| **FreeSWITCH** (each) | 8 vCPU, 16GB RAM | 16 vCPU, 32GB RAM |
| **RTPEngine** (each) | 4 vCPU, 8GB RAM | 8 vCPU, 16GB RAM |
| **SMS Gateway** | 2 vCPU, 4GB RAM | 4 vCPU, 8GB RAM |

---

## 14. Development Phases (Revised for Laravel)

| Phase | Description | Duration | Deliverables |
|-------|-------------|----------|--------------|
| **P1** | Architecture & Setup | 2 weeks | Tech stack finalized, DB schema, Dev environment, CI/CD |
| **P2** | Core Backend | 3 weeks | User auth, Number management, Wallet system, Admin APIs |
| **P3** | VoIP Infrastructure | 3 weeks | Kamailio + FreeSWITCH setup, SIP trunk integration, Call routing |
| **P4** | Mobile Apps - Core | 3 weeks | SIP client integration, Basic calling, Push notifications |
| **P5** | SMS/MMS System | 2 weeks | SMS gateway, MMS handling, Delivery tracking |
| **P6** | Mobile Apps - Features | 2 weeks | Call rules, Number management, Messaging UI |
| **P7** | Web Portals | 2 weeks | User portal, Admin panel |
| **P8** | Integration & Polish | 2 weeks | Payment gateways, Analytics, Device sync |
| **P9** | Security & Testing | 2 weeks | VAPT, Encryption audit, Load testing |
| **P10** | Launch Prep | 1 week | Store submission, Documentation, Go-live |

**Total: ~22 weeks**

---

## 15. Technology Stack Summary

| Layer | Technology |
|-------|------------|
| **Backend Framework** | Laravel 11.x |
| **Database** | MySQL 8.x / PostgreSQL 15.x |
| **Cache/Queue** | Redis + Laravel Horizon |
| **Message Queue** | RabbitMQ (for SMS/VoIP events) |
| **VoIP Signaling** | Kamailio |
| **Media Server** | FreeSWITCH |
| **RTP Proxy** | RTPEngine |
| **SMS Gateway** | Jasmin SMS Gateway |
| **STUN/TURN** | Coturn |
| **Storage** | S3 / MinIO |
| **Mobile - Android** | Kotlin + oLSIP |
| **Mobile - iOS** | Swift + oLSIP |
| **Web Frontend** | Vue.js 3 / React |
| **Admin Panel** | Filament (Laravel) |
| **Payments** | Stripe, PayPal, Apple Pay, etc. |

---

## 16. Marketing Website

### 16.1 Purpose & Features

The marketing website serves as the primary customer acquisition channel and product showcase. It is designed for high conversion rates while maintaining security best practices.

**Key Features:**
- High-conversion landing page with clear value proposition
- Feature breakdowns and use case demonstrations
- SEO-optimized content structure for organic traffic
- Responsive design for all devices
- Fast loading times (optimized for Core Web Vitals)
- Clear CTAs for app downloads and user portal access
- No direct billing/payment on website (security-first approach)
- Seamless redirection to mobile apps and user portal

### 16.2 Website Structure

```
Marketing Website Pages:
├── Home (Landing Page)
│   ├── Hero section with primary CTA
│   ├── Feature highlights
│   ├── Use cases and benefits
│   ├── Pricing overview (redirect to portal for purchase)
│   └── Download links (App Store / Play Store)
│
├── Features
│   ├── Voice calling capabilities
│   ├── SMS/MMS messaging
│   ├── Multi-number management
│   ├── Call routing and forwarding
│   └── Number porting
│
├── Pricing
│   ├── Transparent pricing structure
│   ├── Number costs by region
│   ├── Call/SMS rates
│   └── CTA to sign up (redirect to portal)
│
├── About Us
│   ├── Company mission and vision
│   ├── Team information
│   └── Contact information
│
├── Legal
│   ├── Privacy Policy (GDPR compliant)
│   ├── Terms of Service
│   ├── Cookie Policy
│   └── Acceptable Use Policy
│
└── Resources
    ├── FAQ
    ├── Documentation
    ├── Blog (optional)
    └── Support contact
```

### 16.3 Technology Stack

| Component | Technology |
|-----------|------------|
| **Framework** | Next.js / Nuxt.js (SSR for SEO) |
| **Styling** | Tailwind CSS |
| **Hosting** | Vercel / Netlify / CloudFlare Pages |
| **CMS** | Headless CMS (optional for blog) |
| **Analytics** | Google Analytics / Plausible |

---

## 17. White-Labeling & Customization

### 17.1 White-Labeling Capabilities

The platform supports complete white-labeling to allow clients to brand the application as their own product.

**Customizable Elements:**

```
┌─────────────────────────────────────────────────────────────┐
│                  White-Labeling Components                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  BRANDING                                                   │
│  ├── App Name                                               │
│  ├── App Icon/Logo                                          │
│  ├── Splash Screen                                          │
│  ├── Company Name                                           │
│  └── Legal Entity Information                               │
│                                                             │
│  VISUAL DESIGN                                              │
│  ├── Color Scheme (Primary, Secondary, Accent)             │
│  ├── Typography (Font families)                            │
│  ├── UI Theme (Light/Dark mode defaults)                   │
│  ├── Button Styles                                         │
│  └── Custom Icons and Graphics                             │
│                                                             │
│  CONTENT                                                    │
│  ├── Welcome Messages                                      │
│  ├── Onboarding Screens                                    │
│  ├── Help/Support Text                                     │
│  ├── Email Templates                                       │
│  └── Push Notification Templates                           │
│                                                             │
│  FUNCTIONALITY                                              │
│  ├── Feature Toggles (Enable/Disable features)             │
│  ├── Payment Gateway Selection                             │
│  ├── Pricing Structure                                     │
│  ├── Supported Regions                                     │
│  └── Custom Business Rules                                 │
│                                                             │
│  DOMAINS & URLs                                             │
│  ├── Custom Domain for Web Portal                          │
│  ├── Custom Domain for Marketing Site                      │
│  ├── Custom API Subdomain                                  │
│  └── Deep Links for Mobile Apps                            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 17.2 Configuration Management

```php
// White-Label Configuration
class WhiteLabelConfig
{
    // Branding
    public string $appName;
    public string $companyName;
    public string $logoUrl;
    public string $iconUrl;
    public string $splashScreenUrl;
    
    // Colors
    public string $primaryColor;
    public string $secondaryColor;
    public string $accentColor;
    
    // Typography
    public string $primaryFont;
    public string $secondaryFont;
    
    // Features
    public array $enabledFeatures = [
        'voice_calling' => true,
        'sms' => true,
        'mms' => true,
        'scheduled_sms' => true,
        'call_forwarding' => true,
        'number_porting' => true,
        'multi_number' => true,
    ];
    
    // Payment Gateways
    public array $paymentGateways = [
        'stripe' => true,
        'paypal' => true,
        'apple_pay' => true,
        'cash_app' => false,
        'amazon_pay' => false,
    ];
    
    // Domains
    public string $webPortalDomain;
    public string $marketingSiteDomain;
    public string $apiDomain;
    
    // Support
    public string $supportEmail;
    public string $supportPhone;
    public string $supportUrl;
}
```

### 17.3 Deployment Models

| Model | Description | Use Case |
|-------|-------------|----------|
| **SaaS** | Single shared infrastructure, multiple tenants | Standard offering |
| **Dedicated** | Dedicated infrastructure per client | Enterprise clients |
| **On-Premise** | Client hosts entire platform | Highly regulated industries |
| **Hybrid** | Core on client infrastructure, services on cloud | Mixed requirements |

---

## 18. Security & VAPT

### 18.1 Security Testing Requirements

**Vulnerability Assessment & Penetration Testing (VAPT)** is mandatory before launch to ensure the platform meets enterprise-grade security standards.

### 18.2 VAPT Scope

```
┌─────────────────────────────────────────────────────────────┐
│                    VAPT Testing Scope                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  APPLICATION LAYER                                          │
│  ├── API Security Testing                                  │
│  │   ├── Authentication bypass attempts                    │
│  │   ├── Authorization flaws                               │
│  │   ├── Injection attacks (SQL, NoSQL, Command)           │
│  │   ├── Rate limiting validation                          │
│  │   └── API key/token security                            │
│  │                                                          │
│  ├── Web Application Testing                               │
│  │   ├── XSS (Cross-Site Scripting)                        │
│  │   ├── CSRF (Cross-Site Request Forgery)                 │
│  │   ├── Session management                                │
│  │   ├── File upload vulnerabilities                       │
│  │   └── Business logic flaws                              │
│  │                                                          │
│  └── Mobile Application Testing                            │
│      ├── Insecure data storage                             │
│      ├── Insecure communication                            │
│      ├── Code obfuscation validation                       │
│      ├── Certificate pinning bypass                        │
│      └── Reverse engineering resistance                    │
│                                                             │
│  NETWORK LAYER                                              │
│  ├── VoIP Security                                          │
│  │   ├── SIP message tampering                             │
│  │   ├── RTP/SRTP encryption validation                    │
│  │   ├── Call hijacking attempts                           │
│  │   └── Toll fraud prevention                             │
│  │                                                          │
│  └── Infrastructure Security                                │
│      ├── Server hardening validation                       │
│      ├── Firewall configuration review                     │
│      ├── SSL/TLS configuration                             │
│      └── DDoS protection testing                           │
│                                                             │
│  DATA LAYER                                                 │
│  ├── Encryption at rest validation                         │
│  ├── Encryption in transit validation                      │
│  ├── Database security                                     │
│  ├── Backup security                                       │
│  └── Data leakage prevention                               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 18.3 Security Remediation Process

1. **Pre-Launch VAPT** - Conduct comprehensive security testing
2. **Vulnerability Report** - Document all findings with severity ratings
3. **Critical/High Fixes** - Resolve all critical and high-risk vulnerabilities
4. **Medium/Low Review** - Assess and plan remediation for medium/low risks
5. **Re-Testing** - Verify all fixes with follow-up testing
6. **Security Sign-Off** - Obtain security clearance before production launch
7. **Ongoing Monitoring** - Continuous security monitoring post-launch
8. **Periodic Re-Testing** - Quarterly or bi-annual VAPT cycles

### 18.4 Security Best Practices

- **Principle of Least Privilege** - Minimal access rights for users and services
- **Defense in Depth** - Multiple layers of security controls
- **Secure by Default** - Secure configurations out of the box
- **Regular Updates** - Timely security patches and updates
- **Audit Logging** - Comprehensive logging of security-relevant events
- **Incident Response Plan** - Documented procedures for security incidents
- **Security Training** - Regular training for development and operations teams

---

## 19. Key Challenges & Mitigations

| Challenge | Mitigation |
|-----------|------------|
| **VoIP in restricted regions (GCC)** | Use WebSocket transport (WSS), SRTP, potentially VPN/proxy |
| **NAT traversal** | STUN/TURN servers, ICE candidates |
| **Call quality** | Adaptive codecs, jitter buffers, QoS monitoring |
| **SMS delivery rates** | Multiple carrier routes, intelligent routing |
| **Scalability** | Microservices, horizontal scaling, CDN for media |
| **Security** | Regular VAPT, encryption everywhere, audit logging |

---

## 20. GDPR Compliance Checklist

- [ ] User consent for data collection
- [ ] Right to data export
- [ ] Right to deletion (account + all data)
- [ ] Data minimization (only collect what's needed)
- [ ] Encryption at rest and in transit
- [ ] Privacy policy and terms of service
- [ ] Data processing agreements with third parties
- [ ] Breach notification procedures
- [ ] DPO appointment (if required)

---

## 21. Next Steps

1. **Finalize SIP Trunk Provider** - Evaluate Bandwidth, Telnyx, etc.
2. **Finalize SMS/MMS Provider** - SMPP connections with carriers
3. **Set up Development Environment** - Laravel + VoIP stack
4. **Create Detailed API Specifications** - OpenAPI/Swagger docs
5. **Design Mobile App Wireframes** - UX/UI for Android & iOS
6. **Security Architecture Review** - Before implementation
7. **Cost Analysis** - Infrastructure + carrier costs
8. **VAPT Planning** - Schedule security testing cycles
9. **White-Label Configuration** - Define customization options
10. **Marketing Website Design** - Create landing pages and content

---

*Document Version: 2.0*  
*Created: January 2026*  
*Last Updated: February 2026*  
*Status: Updated with New Requirements*

