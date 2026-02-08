# Step 2: Finalize SMS/MMS Provider - SMPP Connections

## Overview

Selecting the right SMS/MMS provider and establishing SMPP (Short Message Peer-to-Peer) connections with carriers is essential for reliable messaging capabilities. This document outlines the evaluation criteria, provider options, and implementation strategy.

## Understanding SMPP

### What is SMPP?

**SMPP (Short Message Peer-to-Peer Protocol)** is an industry-standard protocol for exchanging SMS messages between SMS peer entities such as:
- Short Message Service Centers (SMSC)
- External Short Messaging Entities (ESME) - our application
- Routing entities

### SMPP vs HTTP API

| Aspect | SMPP | HTTP API |
|--------|------|----------|
| **Connection** | Persistent TCP connection | Request/response |
| **Performance** | Higher throughput (1000+ msg/sec) | Lower throughput |
| **Reliability** | More reliable, immediate delivery status | Polling required |
| **Complexity** | More complex to implement | Easier to implement |
| **Cost** | Often cheaper for high volume | Higher per-message cost |
| **Use Case** | High-volume applications | Low-volume, simple integration |

**Recommendation:** Use SMPP for production due to higher volume and lower costs.

---

## Evaluation Criteria

### 1. Geographic Coverage

| Criterion | Requirement | Details |
|-----------|-------------|---------|
| **Global Reach** | Critical | Support for all target markets |
| **GCC Region Support** | Critical | Reliable delivery in UAE, Saudi Arabia, etc. |
| **Carrier Connections** | High | Direct connections to major carriers |
| **Number Types** | Required | Support for local, toll-free, short codes |

### 2. Technical Capabilities

#### SMPP Features

| Feature | Requirement | Notes |
|---------|-------------|-------|
| **SMPP Version** | 3.4 or 5.0 | Industry standard |
| **Connection Type** | Transceiver | Bidirectional messaging |
| **TLS/SSL Support** | Required | Encrypted connections |
| **Throughput** | 100+ TPS | Transactions per second |
| **Concatenated SMS** | Required | Long messages (>160 chars) |
| **Unicode Support** | Required | International characters |
| **DLR (Delivery Reports)** | Required | Message status tracking |

#### MMS Features

| Feature | Requirement | Notes |
|---------|-------------|-------|
| **MMS Support** | Required | Image, video, audio |
| **Max File Size** | ≥5MB | Per MMS message |
| **Supported Formats** | JPEG, PNG, GIF, MP4, MP3 | Common formats |
| **MMS to Email** | Optional | Additional capability |

### 3. Delivery & Performance

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Delivery Rate** | ≥98% | Successful deliveries |
| **Delivery Speed** | <5 seconds | Average delivery time |
| **Uptime SLA** | ≥99.9% | Service availability |
| **Throughput** | Scalable | Support for growth |
| **Queue Management** | Required | Handle traffic spikes |

### 4. Pricing Structure

| Component | Evaluation Points |
|-----------|-------------------|
| **Per-Message Rates** | Compare by destination country |
| **Inbound SMS** | Often free or low cost |
| **Outbound SMS** | Primary cost driver |
| **MMS Pricing** | Typically 3-5x SMS cost |
| **Long Code Rental** | Monthly fees for numbers |
| **Short Code Rental** | Higher monthly fees |
| **Setup Fees** | One-time costs |
| **Volume Discounts** | Tiered pricing |

### 5. Features & Services

**Essential Features:**
- ✅ Two-way messaging (send/receive)
- ✅ Delivery receipts (DLR)
- ✅ Message scheduling
- ✅ Sender ID customization
- ✅ Opt-out management
- ✅ Message templates
- ✅ Concatenated messages

**Advanced Features:**
- 🔹 Smart routing (least-cost routing)
- 🔹 Fallback routing
- 🔹 Message retry logic
- 🔹 Spam filtering
- 🔹 Analytics and reporting
- 🔹 A2P (Application-to-Person) optimization
- 🔹 Campaign management

---

## Provider Options

### Option 1: Integrated with SIP Trunk Provider

**Providers:**
- Telnyx (SMS/MMS + Voice)
- Bandwidth (SMS/MMS + Voice)
- Plivo (SMS/MMS + Voice)

**Advantages:**
- ✅ Single vendor relationship
- ✅ Unified billing
- ✅ Consistent API
- ✅ Easier integration
- ✅ Better support

**Disadvantages:**
- ⚠️ Vendor lock-in
- ⚠️ May not be cheapest for SMS
- ⚠️ Limited routing flexibility

**Recommendation:** **Best for simplicity and unified management**

---

### Option 2: Dedicated SMS/MMS Provider

**Providers:**
- Twilio (if allowed)
- MessageBird
- Vonage (Nexmo)
- Infobip
- Sinch

**Advantages:**
- ✅ Specialized in messaging
- ✅ Advanced features
- ✅ Better global coverage
- ✅ Competitive pricing

**Disadvantages:**
- ⚠️ Additional vendor to manage
- ⚠️ Separate billing
- ⚠️ More complex integration

**Recommendation:** **Best for high-volume, global messaging**

---

### Option 3: Direct Carrier SMPP Connections

**Approach:**
- Connect directly to mobile carriers via SMPP
- Use aggregators for multi-carrier access

**Advantages:**
- ✅ Lowest cost per message
- ✅ Highest delivery rates
- ✅ Direct carrier relationships
- ✅ Full control

**Disadvantages:**
- ⚠️ Complex setup and management
- ⚠️ Requires technical expertise
- ⚠️ Separate agreements with each carrier
- ⚠️ Compliance and legal requirements

**Recommendation:** **Best for very high volume, specific markets**

---

### Option 4: Open-Source SMS Gateway (Jasmin)

**Jasmin SMS Gateway:**
- Open-source SMPP gateway
- Can connect to multiple SMPP providers
- Routing and load balancing
- Free to use

**Advantages:**
- ✅ Free software
- ✅ Full control
- ✅ Multi-provider routing
- ✅ Customizable
- ✅ No vendor lock-in

**Disadvantages:**
- ⚠️ Requires hosting and maintenance
- ⚠️ Need technical expertise
- ⚠️ Still need SMPP provider connections
- ⚠️ No built-in support

**Recommendation:** **Best for in-house infrastructure requirement**

---

## Recommended Architecture

### Hybrid Approach: Jasmin + Multiple Providers

```
┌─────────────────────────────────────────────────────────────┐
│                     Laravel Application                      │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTP API / Queue
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                  Jasmin SMS Gateway                          │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  • Message routing                                      │ │
│  │  • Load balancing                                       │ │
│  │  • Failover handling                                    │ │
│  │  • DLR processing                                       │ │
│  │  • Rate limiting                                        │ │
│  └────────────────────────────────────────────────────────┘ │
└───┬──────────────────┬──────────────────┬──────────────────┘
    │ SMPP             │ SMPP             │ SMPP
    ▼                  ▼                  ▼
┌─────────┐      ┌─────────┐      ┌─────────┐
│Provider │      │Provider │      │Provider │
│    1    │      │    2    │      │    3    │
│(Telnyx) │      │(Plivo)  │      │(Backup) │
└─────────┘      └─────────┘      └─────────┘
```

**Benefits:**
- ✅ In-house control via Jasmin
- ✅ Multi-provider redundancy
- ✅ Cost optimization (route to cheapest)
- ✅ Failover capability
- ✅ No vendor lock-in

---

## Provider Comparison Matrix

### 1. Telnyx (Recommended Primary)

**SMPP Support:** ✅ Yes  
**API Support:** ✅ REST API also available

**Strengths:**
- ✅ Excellent global coverage
- ✅ Competitive pricing
- ✅ SMPP 3.4 support
- ✅ High throughput
- ✅ Good GCC support
- ✅ Integrated with voice (if using for SIP)

**Pricing (Estimated):**
- Outbound SMS (US): $0.0079/message
- Outbound SMS (International): $0.01 - $0.15/message
- Inbound SMS: $0.0079/message
- MMS: $0.02 - $0.05/message
- Long Code: $1.00 - $3.00/month

**SMPP Connection Details:**
- Host: smpp.telnyx.com
- Port: 2775 (TLS)
- Version: SMPP 3.4
- Throughput: 200 TPS

---

### 2. MessageBird

**SMPP Support:** ✅ Yes  
**API Support:** ✅ REST API also available

**Strengths:**
- ✅ Excellent global coverage (190+ countries)
- ✅ Very competitive pricing
- ✅ Strong in Europe and Asia
- ✅ Good developer experience
- ✅ Omnichannel platform

**Pricing (Estimated):**
- Outbound SMS (US): $0.0075/message
- Outbound SMS (International): Varies
- Inbound SMS: Free or low cost
- MMS: $0.03/message

**SMPP Connection Details:**
- Host: smpp.messagebird.com
- Port: 2775
- Version: SMPP 3.4
- Throughput: 300 TPS

---

### 3. Bandwidth

**SMPP Support:** ⚠️ Limited (primarily REST API)  
**API Support:** ✅ Excellent REST API

**Strengths:**
- ✅ Strong US coverage
- ✅ Integrated with voice
- ✅ Enterprise reliability
- ✅ Good documentation

**Pricing (Estimated):**
- Outbound SMS (US): $0.0075/message
- Inbound SMS: $0.0040/message
- MMS: $0.03/message

**Note:** Primarily REST API, may not offer SMPP

---

### 4. Plivo

**SMPP Support:** ⚠️ No (REST API only)  
**API Support:** ✅ REST API

**Strengths:**
- ✅ Good global coverage
- ✅ Competitive pricing
- ✅ Simple API
- ✅ Integrated with voice

**Pricing (Estimated):**
- Outbound SMS (US): $0.0065/message
- Inbound SMS: $0.0030/message
- MMS: $0.025/message

**Note:** No SMPP, would need to use REST API or find SMPP provider

---

### 5. Infobip

**SMPP Support:** ✅ Yes  
**API Support:** ✅ REST API also available

**Strengths:**
- ✅ Excellent global coverage
- ✅ Strong in emerging markets
- ✅ Enterprise-focused
- ✅ Advanced routing
- ✅ Omnichannel platform

**Pricing (Estimated):**
- Outbound SMS: Varies by volume and destination
- Generally competitive for high volume
- Enterprise pricing model

**SMPP Connection Details:**
- Multiple SMPP endpoints globally
- Version: SMPP 3.4 and 5.0
- High throughput support

---

## SMPP Implementation Guide

### Setting Up Jasmin SMS Gateway

#### Installation

```bash
# Install Jasmin on Ubuntu/Debian
sudo apt-get update
sudo apt-get install python3 python3-pip redis-server rabbitmq-server

# Install Jasmin
sudo pip3 install jasmin

# Initialize Jasmin
sudo jasmind.py --username admin --password admin
```

#### Configuration

```python
# /etc/jasmin/jasmin.cfg

[sm-listener]
# HTTP API for Laravel to send messages
host = 0.0.0.0
port = 1401

[smpp-server]
# SMPP server for receiving messages
host = 0.0.0.0
port = 2775

[redis-client]
host = 127.0.0.1
port = 6379

[amqp-broker]
# RabbitMQ for message queuing
host = 127.0.0.1
port = 5672
```

#### Adding SMPP Connectors

```bash
# Connect to Jasmin management console
telnet localhost 8990

# Add SMPP connector for Telnyx
jcli : smppccm -a
> cid telnyx_primary
> host smpp.telnyx.com
> port 2775
> username YOUR_USERNAME
> password YOUR_PASSWORD
> submit_throughput 100
> ssl yes
> ok

# Add backup connector
jcli : smppccm -a
> cid messagebird_backup
> host smpp.messagebird.com
> port 2775
> username YOUR_USERNAME
> password YOUR_PASSWORD
> submit_throughput 100
> ok
```

#### Routing Configuration

```bash
# Create routing rules
jcli : mtrouter -a
> type DefaultRoute
> connector telnyx_primary
> rate 0.0
> ok

# Add failover route
jcli : mtrouter -a
> type StaticMTRoute
> filters <filter_id>
> connector messagebird_backup
> rate 0.0
> ok
```

---

### Laravel Integration with Jasmin

#### HTTP API Integration

```php
<?php

namespace App\Services\Messaging;

use GuzzleHttp\Client;

class JasminSmsService
{
    protected $client;
    protected $baseUrl;
    protected $username;
    protected $password;
    
    public function __construct()
    {
        $this->client = new Client();
        $this->baseUrl = config('jasmin.api_url'); // http://jasmin-server:1401
        $this->username = config('jasmin.username');
        $this->password = config('jasmin.password');
    }
    
    public function sendSms(string $from, string $to, string $message): array
    {
        $response = $this->client->post($this->baseUrl . '/send', [
            'form_params' => [
                'username' => $this->username,
                'password' => $this->password,
                'to' => $this->formatNumber($to),
                'from' => $from,
                'content' => $message,
                'dlr' => 'yes',
                'dlr-url' => route('webhooks.sms.dlr'),
                'dlr-level' => 3, // Get all delivery reports
            ]
        ]);
        
        $result = json_decode($response->getBody(), true);
        
        return [
            'success' => $response->getStatusCode() === 200,
            'message_id' => $result['data']['messageid'] ?? null,
        ];
    }
    
    public function sendMms(string $from, string $to, string $message, array $media): array
    {
        // MMS implementation
        // Note: Jasmin may not support MMS directly, use provider API
        return $this->sendViaProviderApi($from, $to, $message, $media);
    }
    
    protected function formatNumber(string $number): string
    {
        // Remove non-numeric characters
        $number = preg_replace('/[^0-9]/', '', $number);
        
        // Add country code if missing
        if (strlen($number) === 10) {
            $number = '1' . $number; // Assume US
        }
        
        return $number;
    }
    
    protected function sendViaProviderApi(string $from, string $to, string $message, array $media): array
    {
        // Fallback to provider's REST API for MMS
        // Implementation depends on chosen provider
    }
}
```

#### Delivery Receipt Webhook

```php
<?php

namespace App\Http\Controllers\Webhooks;

use App\Http\Controllers\Controller;
use App\Models\SmsMessage;
use Illuminate\Http\Request;

class SmsWebhookController extends Controller
{
    public function handleDlr(Request $request)
    {
        // Jasmin DLR format
        $messageId = $request->input('id');
        $status = $request->input('dlvrd'); // 1 = delivered, 0 = failed
        $errorCode = $request->input('err');
        
        $sms = SmsMessage::where('external_id', $messageId)->first();
        
        if ($sms) {
            $sms->update([
                'status' => $status == 1 ? 'delivered' : 'failed',
                'delivered_at' => $status == 1 ? now() : null,
                'error_code' => $errorCode,
            ]);
        }
        
        return response()->json(['status' => 'ok']);
    }
    
    public function handleInbound(Request $request)
    {
        // Handle incoming SMS
        $from = $request->input('from');
        $to = $request->input('to');
        $message = $request->input('content');
        
        SmsMessage::create([
            'direction' => 'inbound',
            'from_number' => $from,
            'to_number' => $to,
            'body' => $message,
            'status' => 'received',
            'received_at' => now(),
        ]);
        
        // Trigger event for real-time notification
        event(new SmsReceived($from, $to, $message));
        
        return response()->json(['status' => 'ok']);
    }
}
```

---

## Direct Provider Integration (Alternative)

### If Not Using Jasmin

```php
<?php

namespace App\Services\Messaging;

use GuzzleHttp\Client;

class TelnyxSmsService
{
    protected $client;
    protected $apiKey;
    protected $baseUrl = 'https://api.telnyx.com/v2';
    
    public function __construct()
    {
        $this->client = new Client();
        $this->apiKey = config('telnyx.api_key');
    }
    
    public function sendSms(string $from, string $to, string $message): array
    {
        $response = $this->client->post($this->baseUrl . '/messages', [
            'headers' => [
                'Authorization' => 'Bearer ' . $this->apiKey,
                'Content-Type' => 'application/json',
            ],
            'json' => [
                'from' => $from,
                'to' => $to,
                'text' => $message,
                'webhook_url' => route('webhooks.telnyx.sms'),
            ]
        ]);
        
        $result = json_decode($response->getBody(), true);
        
        return [
            'success' => true,
            'message_id' => $result['data']['id'],
        ];
    }
    
    public function sendMms(string $from, string $to, string $message, array $mediaUrls): array
    {
        $response = $this->client->post($this->baseUrl . '/messages', [
            'headers' => [
                'Authorization' => 'Bearer ' . $this->apiKey,
                'Content-Type' => 'application/json',
            ],
            'json' => [
                'from' => $from,
                'to' => $to,
                'text' => $message,
                'media_urls' => $mediaUrls,
                'webhook_url' => route('webhooks.telnyx.mms'),
            ]
        ]);
        
        $result = json_decode($response->getBody(), true);
        
        return [
            'success' => true,
            'message_id' => $result['data']['id'],
        ];
    }
}
```

---

## Testing & Validation

### Test Checklist

- [ ] **Basic SMS Sending**
  - [ ] Send SMS to domestic number
  - [ ] Send SMS to international number
  - [ ] Send long SMS (concatenated)
  - [ ] Send Unicode SMS (emoji, non-Latin)

- [ ] **MMS Sending**
  - [ ] Send MMS with image
  - [ ] Send MMS with video
  - [ ] Send MMS with multiple attachments

- [ ] **Delivery Receipts**
  - [ ] Receive delivery confirmation
  - [ ] Handle failed delivery
  - [ ] Track delivery timing

- [ ] **Inbound Messages**
  - [ ] Receive SMS
  - [ ] Receive MMS
  - [ ] Handle multiple simultaneous messages

- [ ] **Performance**
  - [ ] Send 100 messages/second
  - [ ] Handle 1000 concurrent sends
  - [ ] Measure delivery latency

- [ ] **Failover**
  - [ ] Simulate primary provider failure
  - [ ] Verify automatic failover
  - [ ] Verify failback when primary recovers

---

## Cost Estimation

### Monthly Cost Calculation

**Assumptions:**
- 1,000 active users
- 50 SMS/user/month = 50,000 SMS
- 5 MMS/user/month = 5,000 MMS
- 80% domestic, 20% international

**Using Telnyx Pricing:**

| Item | Calculation | Monthly Cost |
|------|-------------|--------------|
| **Outbound SMS (Domestic)** | 40,000 × $0.0079 | $316 |
| **Outbound SMS (International)** | 10,000 × $0.02 (avg) | $200 |
| **Inbound SMS** | 25,000 × $0.0079 | $198 |
| **MMS** | 5,000 × $0.03 | $150 |
| **Long Code Rental** | 100 × $1.50 | $150 |
| **Total** | | **$1,014** |

**Per User Cost:** $1.01/month

---

## Recommended Solution

### Primary Recommendation: **Jasmin + Telnyx**

**Architecture:**
1. **Jasmin SMS Gateway** (self-hosted)
   - Message routing and load balancing
   - DLR processing
   - Rate limiting
   - Multi-provider support

2. **Telnyx** (Primary SMPP Provider)
   - Global coverage
   - Competitive pricing
   - High reliability
   - SMPP + REST API

3. **MessageBird** (Backup Provider)
   - Failover routing
   - Geographic optimization
   - Cost optimization

**Benefits:**
- ✅ In-house control
- ✅ No vendor lock-in
- ✅ Cost optimization
- ✅ High reliability
- ✅ Scalable

---

## Implementation Timeline

### Week 1: Setup & Configuration
- Set up Jasmin server
- Configure SMPP connections
- Test basic connectivity

### Week 2: Integration
- Integrate Laravel with Jasmin
- Implement webhook handlers
- Set up delivery tracking

### Week 3: Testing
- Conduct comprehensive testing
- Load testing
- Failover testing

### Week 4: Production Deployment
- Deploy to production
- Monitor performance
- Fine-tune routing

---

## Monitoring & Maintenance

### Key Metrics
- Message delivery rate
- Average delivery time
- Failed message rate
- Provider uptime
- Cost per message
- Throughput (messages/second)

### Alerts
- Delivery rate < 95%
- Provider connection failure
- High error rate
- Cost threshold exceeded
- Queue backlog

---

*Document Version: 1.0*  
*Created: February 2026*  
*Status: Planning Document*
