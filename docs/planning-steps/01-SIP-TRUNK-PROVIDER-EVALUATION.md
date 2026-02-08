# Step 1: Finalize SIP Trunk Provider Evaluation

## Overview

Selecting the right SIP trunk provider is critical for the VoIP platform's success. The provider will handle the connection between our VoIP infrastructure and the Public Switched Telephone Network (PSTN), enabling calls to/from regular cell phones and landlines worldwide.

## Evaluation Criteria

### 1. Geographic Coverage

| Criterion | Importance | Details |
|-----------|------------|---------|
| **Global Reach** | Critical | Must support calls to/from all target markets |
| **GCC Region Support** | Critical | Must work reliably in restricted VoIP regions (UAE, Saudi Arabia, Qatar, etc.) |
| **Local Number Availability** | High | DIDs available in target countries |
| **Termination Quality** | Critical | High-quality call routing to PSTN |

### 2. Technical Capabilities

| Feature | Requirement | Notes |
|---------|-------------|-------|
| **SIP Protocol Support** | Required | RFC 3261 compliant |
| **Codec Support** | Required | G.711, Opus, G.729 minimum |
| **SRTP Encryption** | Required | Secure media transmission |
| **TLS for Signaling** | Required | Encrypted SIP signaling |
| **IPv4/IPv6 Support** | Preferred | Future-proofing |
| **WebRTC Support** | Preferred | For web-based calling |
| **Fax Support (T.38)** | Optional | If fax feature needed |

### 3. Reliability & Performance

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Uptime SLA** | ≥99.95% | Monthly uptime guarantee |
| **Call Completion Rate** | ≥99% | Successful call connections |
| **Average Latency** | <150ms | Round-trip time |
| **Jitter** | <30ms | Voice quality metric |
| **Packet Loss** | <1% | Network quality |
| **Concurrent Calls** | Scalable | Support for growth |

### 4. Pricing Structure

| Component | Evaluation Points |
|-----------|-------------------|
| **Per-Minute Rates** | Compare inbound/outbound rates by destination |
| **DID Costs** | Monthly rental fees by country/region |
| **Setup Fees** | One-time activation costs |
| **Minimum Commitments** | Monthly minimums or prepayment requirements |
| **Volume Discounts** | Pricing tiers based on usage |
| **Overage Charges** | Costs beyond committed volumes |

### 5. Features & Services

**Essential Features:**
- ✅ Inbound/Outbound calling
- ✅ DID provisioning API
- ✅ Real-time CDR (Call Detail Records)
- ✅ Number porting support
- ✅ Emergency calling (E911/E112)
- ✅ CNAM (Caller ID Name) lookup
- ✅ SMS/MMS capabilities (if integrated)

**Advanced Features:**
- 🔹 Fraud detection and prevention
- 🔹 Call recording
- 🔹 Transcription services
- 🔹 Analytics and reporting
- 🔹 Failover and redundancy
- 🔹 Custom routing rules

### 6. Integration & API

| Aspect | Requirements |
|--------|--------------|
| **API Documentation** | Comprehensive, up-to-date |
| **API Protocols** | REST, WebHooks |
| **SDKs/Libraries** | PHP, Node.js preferred |
| **Provisioning API** | Automated number management |
| **Real-time Events** | WebHooks for call events |
| **CDR Access** | Programmatic access to call records |

### 7. Support & SLA

| Support Aspect | Requirement |
|----------------|-------------|
| **24/7 Support** | Required |
| **Response Time** | <1 hour for critical issues |
| **Support Channels** | Phone, Email, Chat, Ticketing |
| **Technical Documentation** | Detailed setup guides |
| **Onboarding Assistance** | Dedicated support during setup |
| **Account Manager** | Dedicated contact for enterprise |

---

## Provider Comparison Matrix

### Top Providers to Evaluate

#### 1. Bandwidth

**Strengths:**
- ✅ Excellent US/Canada coverage
- ✅ Enterprise-grade reliability (99.999% uptime)
- ✅ Robust API and documentation
- ✅ Strong number porting support
- ✅ Integrated SMS/MMS
- ✅ STIR/SHAKEN compliance

**Weaknesses:**
- ⚠️ Limited international coverage compared to others
- ⚠️ Higher pricing for international calls
- ⚠️ May have restrictions in GCC regions

**Pricing (Estimated):**
- Inbound: $0.0035 - $0.0085/min
- Outbound US: $0.0060 - $0.0120/min
- DID Rental: $0.50 - $2.00/month
- SMS: $0.0075/message

**Best For:** US-focused applications, enterprise reliability

---

#### 2. Telnyx

**Strengths:**
- ✅ Global coverage (140+ countries)
- ✅ Competitive pricing
- ✅ Modern API (REST, GraphQL)
- ✅ Real-time WebHooks
- ✅ Excellent developer experience
- ✅ SMS/MMS included
- ✅ Good GCC region support

**Weaknesses:**
- ⚠️ Newer company (less track record)
- ⚠️ Support can be slower for non-enterprise

**Pricing (Estimated):**
- Inbound: $0.0040/min
- Outbound US: $0.0040/min
- International: Varies by destination
- DID Rental: $1.00 - $3.00/month
- SMS: $0.0079/message

**Best For:** Global applications, developer-friendly, cost-conscious

---

#### 3. Twilio (Elastic SIP Trunking)

**Strengths:**
- ✅ Massive global coverage
- ✅ Excellent documentation
- ✅ Mature platform
- ✅ Strong developer community
- ✅ Integrated ecosystem (SMS, Video, etc.)
- ✅ Reliable infrastructure

**Weaknesses:**
- ⚠️ Higher pricing
- ⚠️ Against "in-house" requirement (but can use just SIP trunking)
- ⚠️ Complex pricing structure

**Pricing (Estimated):**
- Inbound: $0.0085/min
- Outbound US: $0.0140/min
- DID Rental: $1.00/month
- SMS: $0.0079/message

**Best For:** If willing to use some Twilio services, global reach

**Note:** May conflict with "no Twilio" requirement, but SIP trunking is separate from their higher-level services.

---

#### 4. Plivo

**Strengths:**
- ✅ Good global coverage
- ✅ Competitive pricing
- ✅ Simple API
- ✅ DID provisioning
- ✅ SMS/MMS support
- ✅ Decent GCC support

**Weaknesses:**
- ⚠️ Support quality varies
- ⚠️ Less robust than Bandwidth/Telnyx
- ⚠️ Smaller network

**Pricing (Estimated):**
- Inbound: $0.0030/min
- Outbound US: $0.0070/min
- DID Rental: $0.80/month
- SMS: $0.0065/message

**Best For:** Budget-conscious, good balance of features and price

---

#### 5. SignalWire

**Strengths:**
- ✅ Built on FreeSWITCH (same as our media server)
- ✅ Developer-friendly
- ✅ Modern API
- ✅ Competitive pricing
- ✅ Video capabilities
- ✅ Good US coverage

**Weaknesses:**
- ⚠️ Limited international coverage
- ⚠️ Newer platform
- ⚠️ Smaller network than competitors

**Pricing (Estimated):**
- Inbound: $0.0040/min
- Outbound US: $0.0050/min
- DID Rental: $1.00/month
- SMS: $0.0075/message

**Best For:** FreeSWITCH integration, US-focused, developer experience

---

#### 6. VoIP.ms

**Strengths:**
- ✅ Very competitive pricing
- ✅ No contracts or commitments
- ✅ Good North American coverage
- ✅ Flexible routing options
- ✅ Prepaid model

**Weaknesses:**
- ⚠️ Less enterprise-focused
- ⚠️ Limited international reach
- ⚠️ Basic API
- ⚠️ Support primarily community-based

**Pricing (Estimated):**
- Inbound: $0.0085/min
- Outbound US: $0.0090/min
- DID Rental: $0.85/month
- SMS: $0.0075/message

**Best For:** Small-scale deployments, testing, budget constraints

---

## Evaluation Process

### Phase 1: Initial Research (Week 1)

**Tasks:**
1. ✅ Review provider websites and documentation
2. ✅ Compare pricing for target markets
3. ✅ Check GCC region support explicitly
4. ✅ Review API documentation quality
5. ✅ Read customer reviews and case studies

**Deliverable:** Shortlist of 3-4 providers

---

### Phase 2: Technical Evaluation (Week 2)

**Tasks:**
1. **Create Test Accounts**
   - Sign up for trial/test accounts with shortlisted providers
   - Request demo credentials if available

2. **Test Call Quality**
   - Make test calls to various destinations
   - Test from different geographic locations
   - Measure latency, jitter, packet loss
   - Test during peak and off-peak hours

3. **API Testing**
   - Test number provisioning API
   - Test call initiation/termination
   - Test WebHook reliability
   - Test CDR retrieval
   - Evaluate API response times

4. **Integration Testing**
   - Test SIP trunk connection with Kamailio
   - Test codec negotiation
   - Test SRTP encryption
   - Test failover scenarios

**Deliverable:** Technical evaluation report with test results

---

### Phase 3: Business Evaluation (Week 3)

**Tasks:**
1. **Pricing Analysis**
   - Calculate costs for projected usage
   - Compare total cost of ownership
   - Negotiate volume discounts
   - Review contract terms

2. **Support Evaluation**
   - Test support response times
   - Evaluate documentation quality
   - Check availability of technical resources
   - Request reference customers

3. **Compliance Check**
   - Verify GDPR compliance
   - Check emergency calling support
   - Review data residency options
   - Confirm number porting capabilities

4. **Risk Assessment**
   - Evaluate provider stability
   - Check redundancy options
   - Review SLA terms
   - Assess vendor lock-in risks

**Deliverable:** Business case document

---

### Phase 4: Final Selection (Week 4)

**Decision Matrix:**

| Criteria | Weight | Bandwidth | Telnyx | Plivo | SignalWire |
|----------|--------|-----------|--------|-------|------------|
| **Global Coverage** | 20% | | | | |
| **GCC Support** | 15% | | | | |
| **Pricing** | 20% | | | | |
| **API Quality** | 15% | | | | |
| **Reliability** | 15% | | | | |
| **Support** | 10% | | | | |
| **Integration** | 5% | | | | |
| **Total Score** | 100% | | | | |

*Score each criterion 1-10, multiply by weight, sum for total*

**Recommendation Process:**
1. Complete scoring matrix
2. Conduct final negotiations with top 2 providers
3. Get final pricing quotes
4. Review with stakeholders
5. Make final decision
6. Sign contract

---

## Implementation Checklist

### Pre-Production Setup

- [ ] Create production account
- [ ] Configure SIP trunk credentials
- [ ] Set up IP whitelisting/authentication
- [ ] Configure codec preferences
- [ ] Enable SRTP encryption
- [ ] Set up emergency calling
- [ ] Configure failover routing
- [ ] Set up billing alerts
- [ ] Configure WebHooks for events
- [ ] Test number provisioning API
- [ ] Port test numbers (if applicable)

### Kamailio Integration

```bash
# Example Kamailio SIP Trunk Configuration

# Define SIP trunk gateway
modparam("dispatcher", "list", "1 sip:sip.provider.com:5060 0 0")

# Authentication
modparam("auth", "username", "your_username")
modparam("auth", "password", "your_password")

# Routing logic
route[PSTN_OUT] {
    # Route to SIP trunk
    if(!ds_select_dst("1", "4")) {
        send_reply("503", "Service Unavailable");
        exit;
    }
    
    # Add authentication
    append_hf("P-Asserted-Identity: <sip:$fU@yourdomain.com>\r\n");
    
    # Forward to trunk
    t_relay();
}
```

### Monitoring & Alerts

**Key Metrics to Monitor:**
- Call success rate
- Average call duration
- Concurrent calls
- Trunk utilization
- Cost per minute
- Failed call reasons
- Latency/jitter/packet loss

**Alert Thresholds:**
- Call failure rate > 5%
- Latency > 200ms
- Daily spend > budget threshold
- Trunk capacity > 80%

---

## Cost Estimation

### Sample Monthly Cost Calculation

**Assumptions:**
- 1,000 active users
- Average 100 minutes/user/month = 100,000 minutes
- 50% inbound, 50% outbound
- 80% domestic (US), 20% international
- 500 DIDs

**Using Telnyx Pricing:**

| Item | Calculation | Monthly Cost |
|------|-------------|--------------|
| **Inbound Calls** | 50,000 min × $0.0040 | $200 |
| **Outbound Domestic** | 40,000 min × $0.0040 | $160 |
| **Outbound International** | 10,000 min × $0.0150 (avg) | $150 |
| **DID Rental** | 500 × $1.50 | $750 |
| **SMS** | 10,000 × $0.0079 | $79 |
| **Total** | | **$1,339** |

**Per User Cost:** $1.34/month

---

## Recommended Provider

### Primary Recommendation: **Telnyx**

**Rationale:**
1. ✅ Excellent global coverage including GCC regions
2. ✅ Competitive pricing
3. ✅ Modern, developer-friendly API
4. ✅ Real-time WebHooks and events
5. ✅ Integrated SMS/MMS
6. ✅ Good documentation and support
7. ✅ Scalable infrastructure
8. ✅ No conflicts with "in-house" requirement

### Backup Recommendation: **Bandwidth**

**Rationale:**
1. ✅ Enterprise-grade reliability
2. ✅ Excellent for US/Canada
3. ✅ Strong number porting
4. ✅ Robust API
5. ⚠️ Use for US market, supplement with Telnyx for international

### Hybrid Approach

**Consider using multiple providers:**
- **Telnyx**: Primary for global coverage
- **Bandwidth**: US/Canada for premium quality
- **Plivo**: Backup/failover provider

**Benefits:**
- Redundancy and failover
- Cost optimization (route to cheapest)
- Geographic optimization
- Risk mitigation

---

## Next Steps

1. **Week 1**: Complete initial research and create shortlist
2. **Week 2**: Set up test accounts and conduct technical evaluation
3. **Week 3**: Complete business evaluation and cost analysis
4. **Week 4**: Make final decision and sign contract
5. **Week 5**: Begin production setup and integration

---

## Resources

### Documentation Links
- [Telnyx API Docs](https://developers.telnyx.com/)
- [Bandwidth API Docs](https://dev.bandwidth.com/)
- [Plivo API Docs](https://www.plivo.com/docs/)
- [SignalWire API Docs](https://developer.signalwire.com/)

### Testing Tools
- SIPp (SIP testing tool)
- Wireshark (packet analysis)
- VoIP quality testing tools
- Load testing frameworks

### Compliance Resources
- STIR/SHAKEN information
- E911 requirements
- GDPR compliance guides
- Telecom regulations by country

---

*Document Version: 1.0*  
*Created: February 2026*  
*Status: Planning Document*
