# Database Schema

## Overview
Entity-relationship diagram showing all database tables and their relationships.

## Complete Database Schema

```mermaid
erDiagram
    users ||--o{ phone_numbers : owns
    users ||--o{ calls : makes
    users ||--o{ sms_messages : sends
    users ||--o{ mms_messages : sends
    users ||--o{ voicemails : receives
    users ||--o{ porting_requests : submits
    users ||--|| wallets : has
    users ||--o{ device_registrations : registers
    users ||--o{ transactions : performs

    phone_numbers ||--o{ calls : uses
    phone_numbers ||--o{ sms_messages : uses
    phone_numbers ||--o{ mms_messages : uses
    phone_numbers ||--o{ voicemails : receives_on
    phone_numbers ||--o{ call_rules : has

    mms_messages ||--o{ mms_media : contains

    wallets ||--o{ transactions : records

    users {
        bigint id PK
        uuid uuid UK
        varchar email UK
        varchar phone
        varchar password
        varchar pin
        enum status
        timestamp created_at
        timestamp updated_at
    }

    phone_numbers {
        bigint id PK
        bigint user_id FK
        varchar number UK
        varchar country_code
        enum type
        enum status
        boolean is_primary
        json capabilities
        decimal monthly_cost
        timestamp purchased_at
        timestamp expires_at
        timestamp created_at
        timestamp updated_at
    }

    calls {
        bigint id PK
        uuid uuid UK
        bigint user_id FK
        bigint phone_number_id FK
        enum direction
        varchar from_number
        varchar to_number
        enum status
        int duration_seconds
        decimal cost
        varchar recording_url
        varchar call_sid
        timestamp started_at
        timestamp answered_at
        timestamp ended_at
        json metadata
        timestamp created_at
    }

    sms_messages {
        bigint id PK
        uuid uuid UK
        bigint user_id FK
        bigint phone_number_id FK
        enum direction
        varchar from_number
        varchar to_number
        text body
        int segments
        enum status
        decimal cost
        timestamp sent_at
        timestamp delivered_at
        timestamp scheduled_at
        enum scheduled_status
        json metadata
        timestamp created_at
    }

    mms_messages {
        bigint id PK
        uuid uuid UK
        bigint user_id FK
        bigint phone_number_id FK
        enum direction
        varchar from_number
        varchar to_number
        text body
        enum status
        decimal cost
        timestamp sent_at
        timestamp delivered_at
        json metadata
        timestamp created_at
    }

    mms_media {
        bigint id PK
        bigint mms_message_id FK
        varchar media_url
        varchar media_type
        int file_size
        timestamp created_at
    }

    voicemails {
        bigint id PK
        uuid uuid UK
        bigint user_id FK
        bigint phone_number_id FK
        varchar from_number
        int duration_seconds
        varchar audio_url
        text transcription
        boolean is_read
        timestamp created_at
    }

    call_rules {
        bigint id PK
        bigint phone_number_id FK
        enum rule_type
        boolean is_active
        varchar forward_to
        int timeout_seconds
        json schedule
        int priority
        timestamp created_at
        timestamp updated_at
    }

    wallets {
        bigint id PK
        bigint user_id FK
        decimal balance
        varchar currency
        boolean auto_recharge
        decimal auto_recharge_amount
        decimal auto_recharge_threshold
        timestamp created_at
        timestamp updated_at
    }

    transactions {
        bigint id PK
        uuid uuid UK
        bigint user_id FK
        bigint wallet_id FK
        enum type
        enum category
        decimal amount
        decimal balance_after
        varchar reference_type
        bigint reference_id
        varchar description
        timestamp created_at
    }

    porting_requests {
        bigint id PK
        bigint user_id FK
        varchar number
        enum direction
        enum status
        varchar carrier_name
        varchar account_number
        varchar pin
        json documents
        text admin_notes
        bigint admin_approved_by FK
        timestamp admin_approved_at
        timestamp submitted_at
        timestamp completed_at
        timestamp created_at
        timestamp updated_at
    }

    device_registrations {
        bigint id PK
        bigint user_id FK
        varchar device_id
        enum device_type
        varchar push_token
        varchar sip_username
        varchar sip_password
        boolean is_active
        timestamp last_seen_at
        timestamp created_at
        timestamp updated_at
    }
```

## Table Relationships

### Core User Data
- **users**: Central user account table
- **device_registrations**: Multiple devices per user for SIP and push notifications
- **wallets**: One wallet per user for billing

### Phone Numbers
- **phone_numbers**: Virtual numbers owned by users (1-5 per user)
- **call_rules**: Forwarding and routing rules per number

### Communication Records
- **calls**: Voice call history (inbound/outbound)
- **sms_messages**: SMS message history with scheduling support
- **mms_messages**: MMS message history
- **mms_media**: Media attachments for MMS
- **voicemails**: Voicemail recordings and transcriptions

### Billing
- **wallets**: User balance and auto-recharge settings
- **transactions**: Complete transaction history with references

### Number Management
- **porting_requests**: Number porting workflow tracking

## Key Indexes

```sql
-- Performance Indexes
CREATE INDEX idx_calls_user_created ON calls(user_id, created_at DESC);
CREATE INDEX idx_calls_phone_number ON calls(phone_number_id, created_at DESC);
CREATE INDEX idx_calls_status ON calls(status);

CREATE INDEX idx_sms_user_created ON sms_messages(user_id, created_at DESC);
CREATE INDEX idx_sms_phone_number ON sms_messages(phone_number_id, created_at DESC);
CREATE INDEX idx_sms_scheduled ON sms_messages(scheduled_at) WHERE scheduled_status = 'pending';

CREATE INDEX idx_phone_numbers_user ON phone_numbers(user_id, status);
CREATE INDEX idx_phone_numbers_number ON phone_numbers(number);

CREATE INDEX idx_transactions_user ON transactions(user_id, created_at DESC);
CREATE INDEX idx_transactions_wallet ON transactions(wallet_id, created_at DESC);

CREATE INDEX idx_device_registrations_user ON device_registrations(user_id, is_active);
```

## Data Types Reference

### Enums

**User Status**
- `active`: Normal active user
- `suspended`: Temporarily suspended
- `deleted`: Soft deleted

**Phone Number Type**
- `local`: Local number
- `toll_free`: Toll-free number
- `mobile`: Mobile number

**Phone Number Status**
- `active`: Currently active
- `inactive`: Temporarily disabled
- `porting`: Being ported
- `released`: Released back to pool

**Call Direction**
- `inbound`: Incoming call
- `outbound`: Outgoing call

**Call Status**
- `initiated`: Call initiated
- `ringing`: Phone ringing
- `answered`: Call answered
- `completed`: Call completed normally
- `failed`: Call failed
- `busy`: Line busy
- `no_answer`: No answer

**Message Status**
- `queued`: Queued for sending
- `sent`: Sent to carrier
- `delivered`: Delivered to recipient
- `failed`: Delivery failed
- `received`: Received message

**Transaction Type**
- `credit`: Money added
- `debit`: Money deducted
- `refund`: Money refunded

**Transaction Category**
- `recharge`: Wallet recharge
- `call`: Call charges
- `sms`: SMS charges
- `mms`: MMS charges
- `number_purchase`: Number purchase
- `number_renewal`: Monthly renewal
- `porting`: Porting fees

**Porting Status**
- `pending`: Initial request
- `documents_uploaded`: Documents provided
- `admin_review`: Under admin review
- `approved`: Approved by admin
- `rejected`: Rejected by admin
- `submitted_to_carrier`: Sent to carrier
- `completed`: Porting complete
- `cancelled`: Request cancelled

**Device Type**
- `android`: Android device
- `ios`: iOS device
- `web`: Web browser

## Data Retention Policies

| Table | Retention Period | Notes |
|-------|------------------|-------|
| calls | 2 years | Regulatory requirement |
| sms_messages | 1 year | User data |
| mms_messages | 1 year | User data |
| voicemails | Until deleted by user | User-controlled |
| transactions | 7 years | Financial records |
| porting_requests | 5 years | Regulatory requirement |
