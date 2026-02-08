# Billing System

## Overview
Complete billing system architecture including wallet management, payment processing, and transaction tracking.

## Billing System Architecture

```mermaid
graph TB
    subgraph UserActions["User Actions"]
        Recharge[Wallet Recharge]
        MakeCall[Make Call]
        SendSMS[Send SMS]
        SendMMS[Send MMS]
        BuyNumber[Buy Number]
        PortNumber[Port Number]
    end
    
    subgraph WalletSystem["Wallet System"]
        Wallet[(User Wallet<br/>Balance)]
        AutoRecharge[Auto-Recharge<br/>System]
        LowBalanceAlert[Low Balance<br/>Alerts]
    end
    
    subgraph PaymentGateways["Payment Gateways"]
        Stripe[Stripe]
        PayPal[PayPal]
        ApplePay[Apple Pay]
        CashApp[Cash App]
        AmazonPay[Amazon Pay]
    end
    
    subgraph PricingEngine["Pricing Engine"]
        CallRates[Call Rates<br/>Per Minute]
        SMSRates[SMS Rates<br/>Per Message]
        MMSRates[MMS Rates<br/>Per Message]
        NumberPricing[Number Pricing<br/>Monthly]
    end
    
    subgraph TransactionSystem["Transaction System"]
        TransactionLog[(Transaction<br/>History)]
        InvoiceGen[Invoice<br/>Generator]
        Receipt[Receipt<br/>Generator]
    end
    
    Recharge --> PaymentGateways
    PaymentGateways --> Wallet
    
    Wallet --> AutoRecharge
    Wallet --> LowBalanceAlert
    
    MakeCall --> CallRates
    SendSMS --> SMSRates
    SendMMS --> MMSRates
    BuyNumber --> NumberPricing
    PortNumber --> NumberPricing
    
    CallRates --> Wallet
    SMSRates --> Wallet
    MMSRates --> Wallet
    NumberPricing --> Wallet
    
    Wallet --> TransactionLog
    TransactionLog --> InvoiceGen
    TransactionLog --> Receipt

    classDef actionStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef walletStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef paymentStyle fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef pricingStyle fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef transactionStyle fill:#fff9c4,stroke:#f57f17,stroke-width:2px

    class Recharge,MakeCall,SendSMS,SendMMS,BuyNumber,PortNumber actionStyle
    class Wallet,AutoRecharge,LowBalanceAlert walletStyle
    class Stripe,PayPal,ApplePay,CashApp,AmazonPay paymentStyle
    class CallRates,SMSRates,MMSRates,NumberPricing pricingStyle
    class TransactionLog,InvoiceGen,Receipt transactionStyle
```

## Payment Flow

```mermaid
sequenceDiagram
    participant User
    participant App
    participant Laravel as Laravel API
    participant Wallet as Wallet Service
    participant Payment as Payment Gateway
    participant DB as Database

    User->>App: 1. Initiate Recharge<br/>Amount: $50
    
    App->>Laravel: 2. POST /wallet/recharge<br/>{amount: 50, method: 'stripe'}
    
    activate Laravel
    Laravel->>Wallet: 3. Create Recharge Intent
    
    activate Wallet
    Wallet->>Payment: 4. Create Payment Intent<br/>Stripe API
    
    activate Payment
    Payment-->>Wallet: 5. Payment Intent<br/>{client_secret, id}
    deactivate Payment
    
    Wallet-->>Laravel: 6. Payment Intent Data
    deactivate Wallet
    
    Laravel-->>App: 7. Client Secret
    deactivate Laravel
    
    App->>Payment: 8. Confirm Payment<br/>Stripe.js
    
    activate Payment
    Note over Payment: 9. Process Payment<br/>Card Charge
    
    alt Payment Success
        Payment-->>App: 10a. Payment Successful
        
        Payment->>Laravel: 11a. Webhook: payment_intent.succeeded
        
        activate Laravel
        Laravel->>Wallet: 12a. Credit Wallet
        
        activate Wallet
        Wallet->>DB: 13a. Update Balance<br/>+$50
        
        Wallet->>DB: 14a. Create Transaction<br/>Type: credit
        
        Wallet-->>Laravel: 15a. Wallet Updated
        deactivate Wallet
        
        Laravel->>App: 16a. Push Notification:<br/>Recharge Successful
        
        Laravel-->>Payment: 17a. 200 OK
        deactivate Laravel
        
    else Payment Failed
        Payment-->>App: 10b. Payment Failed
        
        Payment->>Laravel: 11b. Webhook: payment_intent.failed
        
        activate Laravel
        Laravel->>App: 12b. Push Notification:<br/>Payment Failed
        
        Laravel-->>Payment: 13b. 200 OK
        deactivate Laravel
    end
    
    deactivate Payment
```

## Usage Billing Flow

```mermaid
sequenceDiagram
    participant User
    participant App
    participant Laravel as Laravel API
    participant Pricing as Pricing Engine
    participant Wallet as Wallet Service
    participant DB as Database

    Note over User,DB: Example: Outbound Call

    User->>App: 1. Initiate Call<br/>To: +1234567890
    
    App->>Laravel: 2. POST /calls/initiate<br/>{to, from}
    
    activate Laravel
    Laravel->>Pricing: 3. Calculate Cost<br/>Destination: +1234567890
    
    activate Pricing
    Note over Pricing: 4. Lookup Rate<br/>US: $0.01/min
    
    Pricing-->>Laravel: 5. Rate: $0.01/min
    deactivate Pricing
    
    Laravel->>Wallet: 6. Check Balance
    
    activate Wallet
    Wallet->>DB: 7. Get Wallet Balance
    
    DB-->>Wallet: 8. Balance: $25.00
    
    alt Sufficient Balance
        Wallet-->>Laravel: 9a. Balance OK
        deactivate Wallet
        
        Laravel->>Laravel: 10a. Create Call Record<br/>Status: initiated
        
        Laravel-->>App: 11a. Call Authorized<br/>Proceed
        
        Note over App: Call in Progress<br/>Duration: 5 minutes
        
        App->>Laravel: 12a. Call Ended<br/>Duration: 300 seconds
        
        Laravel->>Pricing: 13a. Calculate Final Cost<br/>5 min × $0.01
        
        activate Pricing
        Pricing-->>Laravel: 14a. Cost: $0.05
        deactivate Pricing
        
        Laravel->>Wallet: 15a. Deduct Amount
        
        activate Wallet
        Wallet->>DB: 16a. Update Balance<br/>$25.00 - $0.05
        
        Wallet->>DB: 17a. Create Transaction<br/>Type: debit, Amount: $0.05
        
        Wallet-->>Laravel: 18a. Balance Updated
        deactivate Wallet
        
        Laravel->>DB: 19a. Update Call Record<br/>Status: completed, Cost: $0.05
        
        Laravel-->>App: 20a. Call Complete
        
    else Insufficient Balance
        Wallet-->>Laravel: 9b. Insufficient Balance
        deactivate Wallet
        
        Laravel-->>App: 10b. Error: Low Balance<br/>Please recharge
    end
    
    deactivate Laravel
```

## Auto-Recharge System

```mermaid
stateDiagram-v2
    [*] --> MonitorBalance: User Enables<br/>Auto-Recharge
    
    MonitorBalance --> CheckBalance: Every Transaction
    
    CheckBalance --> AboveThreshold: Balance > Threshold
    CheckBalance --> BelowThreshold: Balance ≤ Threshold
    
    AboveThreshold --> MonitorBalance: Continue Monitoring
    
    BelowThreshold --> InitiateRecharge: Trigger Auto-Recharge
    
    InitiateRecharge --> ProcessPayment: Charge Saved<br/>Payment Method
    
    ProcessPayment --> PaymentSuccess: Payment OK
    ProcessPayment --> PaymentFailed: Payment Failed
    
    PaymentSuccess --> CreditWallet: Add Funds
    
    CreditWallet --> NotifyUser: Send Success<br/>Notification
    
    NotifyUser --> MonitorBalance: Resume Monitoring
    
    PaymentFailed --> RetryPayment: Retry (Max 3)
    
    RetryPayment --> ProcessPayment: Attempt Again
    RetryPayment --> NotifyFailure: Max Retries<br/>Reached
    
    NotifyFailure --> DisableAutoRecharge: Disable Feature
    
    DisableAutoRecharge --> [*]: User Must<br/>Manually Recharge

    note right of MonitorBalance
        Settings:
        - Threshold: $10
        - Recharge Amount: $50
        - Payment Method: Saved Card
    end note

    note right of PaymentFailed
        Retry Logic:
        - Attempt 1: Immediate
        - Attempt 2: After 1 hour
        - Attempt 3: After 6 hours
    end note
```

## Pricing Structure

```mermaid
graph TB
    subgraph PhoneNumbers["Phone Numbers"]
        LocalNum[Local Number<br/>$2/month]
        TollFree[Toll-Free<br/>$5/month]
        MobileNum[Mobile Number<br/>$3/month]
    end
    
    subgraph VoiceCalls["Voice Calls"]
        InboundUS[Inbound US<br/>$0.005/min]
        OutboundUS[Outbound US<br/>$0.01/min]
        OutboundIntl[Outbound Intl<br/>$0.05-0.50/min]
    end
    
    subgraph Messaging["Messaging"]
        SMSOutbound[SMS Outbound<br/>$0.0075/msg]
        SMSInbound[SMS Inbound<br/>Free]
        MMSOutbound[MMS Outbound<br/>$0.02/msg]
        MMSInbound[MMS Inbound<br/>Free]
    end
    
    subgraph Additional["Additional Services"]
        Porting[Number Porting<br/>$5 one-time]
        Recording[Call Recording<br/>$0.002/min]
        Transcription[Voicemail Transcription<br/>$0.05/min]
    end

    classDef numberStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef callStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef msgStyle fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef addStyle fill:#f3e5f5,stroke:#4a148c,stroke-width:2px

    class LocalNum,TollFree,MobileNum numberStyle
    class InboundUS,OutboundUS,OutboundIntl callStyle
    class SMSOutbound,SMSInbound,MMSOutbound,MMSInbound msgStyle
    class Porting,Recording,Transcription addStyle
```

## Transaction Types

```mermaid
graph LR
    subgraph Credits["Credit Transactions"]
        Recharge[Wallet Recharge]
        Refund[Refund]
        Bonus[Promotional Bonus]
    end
    
    subgraph Debits["Debit Transactions"]
        CallCharge[Call Charges]
        SMSCharge[SMS Charges]
        MMSCharge[MMS Charges]
        NumberCharge[Number Fees]
        PortingCharge[Porting Fees]
    end
    
    subgraph Wallet["User Wallet"]
        Balance[Current Balance]
    end
    
    Recharge -->|+| Balance
    Refund -->|+| Balance
    Bonus -->|+| Balance
    
    Balance -->|-| CallCharge
    Balance -->|-| SMSCharge
    Balance -->|-| MMSCharge
    Balance -->|-| NumberCharge
    Balance -->|-| PortingCharge

    classDef creditStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef debitStyle fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    classDef balanceStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px

    class Recharge,Refund,Bonus creditStyle
    class CallCharge,SMSCharge,MMSCharge,NumberCharge,PortingCharge debitStyle
    class Balance balanceStyle
```

## Invoice Generation

```mermaid
graph TB
    Start([Month End]) --> Collect[Collect All<br/>Transactions]
    
    Collect --> Group[Group by<br/>Category]
    
    Group --> Calculate[Calculate<br/>Totals]
    
    Calculate --> Generate[Generate<br/>Invoice PDF]
    
    Generate --> Store[Store in S3]
    
    Store --> Email[Email to User]
    
    Email --> UpdateDB[Update Invoice<br/>Record in DB]
    
    UpdateDB --> End([Complete])

    classDef processStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px

    class Collect,Group,Calculate,Generate,Store,Email,UpdateDB processStyle
```

## Payment Gateway Integration

### Supported Gateways

| Gateway | Features | Use Case |
|---------|----------|----------|
| **Stripe** | Cards, ACH, Apple Pay | Primary payment method |
| **PayPal** | PayPal balance, cards | Alternative payment |
| **Apple Pay** | iOS native | Mobile convenience |
| **Cash App** | Cash App balance | US market |
| **Amazon Pay** | Amazon account | Existing Amazon users |

### Payment Methods
- Credit/Debit Cards (Visa, Mastercard, Amex)
- ACH Bank Transfer
- Digital Wallets (Apple Pay, Google Pay)
- Cash App
- PayPal
- Amazon Pay

## Refund Policy

### Automatic Refunds
- Failed SMS/MMS delivery
- Call connection failures
- Number release (prorated)

### Manual Refunds
- User request (admin approval)
- Service issues
- Overcharges

### Refund Processing
1. User requests refund or automatic trigger
2. Admin reviews (if manual)
3. Refund approved
4. Credit back to wallet or original payment method
5. Transaction record created
6. User notified

## Financial Reports

### Available Reports
- Daily revenue summary
- Monthly financial statements
- Transaction history export
- Tax reports (by jurisdiction)
- Payment gateway reconciliation
- User spending analytics

### Report Formats
- PDF for invoices
- CSV for data export
- Excel for detailed analysis
- JSON via API
