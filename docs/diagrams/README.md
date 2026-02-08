# VoIP Platform - System Architecture Diagrams

This directory contains comprehensive diagrams for the VoIP platform system architecture. All diagrams are created using Mermaid syntax for easy rendering and maintenance.

## Diagram Index

### 1. [High-Level System Architecture](./01-high-level-architecture.md)
Complete overview of the system layers: Client, Application, Communication, Carrier, and PSTN.

### 2. [VoIP Call Flow](./02-voip-call-flow.md)
Detailed sequence diagram showing how voice calls are processed through Kamailio, FreeSWITCH, and SIP trunks.

### 3. [SMS/MMS Workflow](./03-sms-mms-workflow.md)
Message processing flow through Laravel backend, message queues, and Jasmin SMS Gateway.

### 4. [Authentication Flow](./04-authentication-flow.md)
Complete authentication sequence including JWT tokens and SIP registration.

### 5. [Database Schema](./05-database-schema.md)
Entity relationship diagram showing all database tables and their relationships.

### 6. [Number Porting Workflow](./06-number-porting-workflow.md)
Step-by-step process for porting numbers in/out of the platform.

### 7. [Device Data Sync](./07-device-sync-architecture.md)
Architecture for device-local data storage and encrypted backup/transfer.

### 8. [Security Layers](./08-security-layers.md)
Multi-layered security architecture including TLS, SRTP, and application security.

### 9. [Infrastructure Deployment](./09-infrastructure-deployment.md)
Production infrastructure setup with load balancers, servers, and VoIP components.

### 10. [API Structure](./10-api-structure.md)
Complete API endpoint organization and routing structure.

### 11. [Billing System](./11-billing-system.md)
Wallet, payment gateway, and transaction flow diagrams.

### 12. [White-Labeling Components](./12-white-labeling.md)
Customizable components for white-label deployments.

## Usage

These diagrams can be rendered using:
- GitHub (native Mermaid support)
- Mermaid Live Editor (https://mermaid.live)
- VS Code with Mermaid extension
- Documentation generators that support Mermaid

## Maintenance

When updating the system architecture, ensure corresponding diagrams are updated to maintain accuracy.
