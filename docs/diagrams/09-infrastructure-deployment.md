# Infrastructure Deployment

## Overview
Production infrastructure architecture with high availability and scalability.

## Complete Infrastructure Architecture

```mermaid
graph TB
    subgraph Internet["Internet"]
        Users[Users]
        PSTN[PSTN Network]
    end
    
    subgraph CDN["CDN Layer"]
        CloudFlare[CloudFlare CDN<br/>DDoS Protection]
    end
    
    subgraph LoadBalancer["Load Balancer Layer"]
        LB1[Nginx LB 1<br/>Primary]
        LB2[Nginx LB 2<br/>Standby]
    end
    
    subgraph AppServers["Application Servers"]
        App1[Laravel App 1<br/>8 vCPU, 16GB RAM]
        App2[Laravel App 2<br/>8 vCPU, 16GB RAM]
        App3[Laravel App 3<br/>8 vCPU, 16GB RAM]
    end
    
    subgraph VoIPCluster["VoIP Infrastructure"]
        Kam1[Kamailio 1<br/>8 vCPU, 16GB RAM]
        Kam2[Kamailio 2<br/>8 vCPU, 16GB RAM]
        FS1[FreeSWITCH 1<br/>16 vCPU, 32GB RAM]
        FS2[FreeSWITCH 2<br/>16 vCPU, 32GB RAM]
        RTP1[RTPEngine 1<br/>8 vCPU, 16GB RAM]
        RTP2[RTPEngine 2<br/>8 vCPU, 16GB RAM]
    end
    
    subgraph MessagingCluster["Messaging Infrastructure"]
        Jasmin[Jasmin SMS Gateway<br/>4 vCPU, 8GB RAM]
        RabbitMQ[RabbitMQ Cluster<br/>4 vCPU, 8GB RAM]
    end
    
    subgraph DataLayer["Data Layer"]
        MySQLPrimary[(MySQL Primary<br/>8 vCPU, 32GB RAM)]
        MySQLReplica[(MySQL Replica<br/>8 vCPU, 32GB RAM)]
        RedisCluster[(Redis Cluster<br/>4 vCPU, 8GB RAM)]
        S3[S3/MinIO Storage<br/>Object Storage]
    end
    
    subgraph Monitoring["Monitoring & Logging"]
        Prometheus[Prometheus<br/>Metrics]
        Grafana[Grafana<br/>Dashboards]
        ELK[ELK Stack<br/>Logging]
    end
    
    subgraph External["External Services"]
        SIPTrunk[SIP Trunk<br/>Providers]
        SMPPCarrier[SMPP<br/>Carriers]
        PaymentGW[Payment<br/>Gateways]
    end
    
    Users --> CloudFlare
    CloudFlare --> LB1
    CloudFlare -.->|Failover| LB2
    
    LB1 --> App1
    LB1 --> App2
    LB1 --> App3
    
    LB2 -.-> App1
    LB2 -.-> App2
    LB2 -.-> App3
    
    App1 --> MySQLPrimary
    App2 --> MySQLPrimary
    App3 --> MySQLPrimary
    
    MySQLPrimary -.->|Replication| MySQLReplica
    
    App1 --> RedisCluster
    App2 --> RedisCluster
    App3 --> RedisCluster
    
    App1 --> S3
    App2 --> S3
    App3 --> S3
    
    App1 --> RabbitMQ
    App2 --> RabbitMQ
    App3 --> RabbitMQ
    
    Users --> Kam1
    Users --> Kam2
    
    Kam1 --> FS1
    Kam1 --> FS2
    Kam2 --> FS1
    Kam2 --> FS2
    
    FS1 --> RTP1
    FS1 --> RTP2
    FS2 --> RTP1
    FS2 --> RTP2
    
    RTP1 --> SIPTrunk
    RTP2 --> SIPTrunk
    SIPTrunk --> PSTN
    
    RabbitMQ --> Jasmin
    Jasmin --> SMPPCarrier
    SMPPCarrier --> PSTN
    
    App1 --> PaymentGW
    App2 --> PaymentGW
    App3 --> PaymentGW
    
    App1 -.-> Prometheus
    App2 -.-> Prometheus
    App3 -.-> Prometheus
    Kam1 -.-> Prometheus
    Kam2 -.-> Prometheus
    FS1 -.-> Prometheus
    FS2 -.-> Prometheus
    
    Prometheus --> Grafana
    
    App1 -.-> ELK
    App2 -.-> ELK
    App3 -.-> ELK

    classDef lbStyle fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    classDef appStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef voipStyle fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef msgStyle fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef dataStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef monitorStyle fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    classDef extStyle fill:#e0e0e0,stroke:#424242,stroke-width:2px

    class LB1,LB2,CloudFlare lbStyle
    class App1,App2,App3 appStyle
    class Kam1,Kam2,FS1,FS2,RTP1,RTP2 voipStyle
    class Jasmin,RabbitMQ msgStyle
    class MySQLPrimary,MySQLReplica,RedisCluster,S3 dataStyle
    class Prometheus,Grafana,ELK monitorStyle
    class SIPTrunk,SMPPCarrier,PaymentGW extStyle
```

## High Availability Setup

```mermaid
graph TB
    subgraph Region1["Primary Region (US-East)"]
        subgraph AZ1["Availability Zone 1"]
            LB1_AZ1[Load Balancer 1]
            App1_AZ1[App Server 1]
            App2_AZ1[App Server 2]
            DB1_AZ1[(MySQL Primary)]
            Redis1_AZ1[(Redis Master)]
        end
        
        subgraph AZ2["Availability Zone 2"]
            LB2_AZ2[Load Balancer 2]
            App3_AZ2[App Server 3]
            App4_AZ2[App Server 4]
            DB2_AZ2[(MySQL Standby)]
            Redis2_AZ2[(Redis Replica)]
        end
    end
    
    subgraph Region2["DR Region (US-West)"]
        subgraph AZ3["Availability Zone 3"]
            LB3_AZ3[Load Balancer 3]
            App5_AZ3[App Server 5]
            DB3_AZ3[(MySQL Replica)]
            Redis3_AZ3[(Redis Replica)]
        end
    end
    
    LB1_AZ1 --> App1_AZ1
    LB1_AZ1 --> App2_AZ1
    LB2_AZ2 --> App3_AZ2
    LB2_AZ2 --> App4_AZ2
    
    App1_AZ1 --> DB1_AZ1
    App2_AZ1 --> DB1_AZ1
    App3_AZ2 --> DB1_AZ1
    App4_AZ2 --> DB1_AZ1
    
    DB1_AZ1 -.->|Sync Replication| DB2_AZ2
    DB1_AZ1 -.->|Async Replication| DB3_AZ3
    
    App1_AZ1 --> Redis1_AZ1
    App2_AZ1 --> Redis1_AZ1
    
    Redis1_AZ1 -.->|Replication| Redis2_AZ2
    Redis1_AZ1 -.->|Replication| Redis3_AZ3
    
    LB3_AZ3 -.->|DR Failover| App5_AZ3
    App5_AZ3 -.-> DB3_AZ3
    App5_AZ3 -.-> Redis3_AZ3

    classDef primaryStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef standbyStyle fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    classDef drStyle fill:#ffecb3,stroke:#ff6f00,stroke-width:2px

    class LB1_AZ1,App1_AZ1,App2_AZ1,DB1_AZ1,Redis1_AZ1 primaryStyle
    class LB2_AZ2,App3_AZ2,App4_AZ2,DB2_AZ2,Redis2_AZ2 standbyStyle
    class LB3_AZ3,App5_AZ3,DB3_AZ3,Redis3_AZ3 drStyle
```

## Network Architecture

```mermaid
graph TB
    subgraph PublicZone["Public Zone (DMZ)"]
        PublicLB[Public Load Balancer<br/>Public IP]
        WAF[Web Application<br/>Firewall]
    end
    
    subgraph AppZone["Application Zone"]
        AppServers[Application Servers<br/>Private IPs]
        VoIPServers[VoIP Servers<br/>Public IPs for SIP]
    end
    
    subgraph DataZone["Data Zone"]
        Databases[(Databases<br/>Private IPs)]
        Cache[(Cache Servers<br/>Private IPs)]
        Storage[Object Storage<br/>Private Endpoint]
    end
    
    subgraph ManagementZone["Management Zone"]
        Bastion[Bastion Host<br/>SSH Gateway]
        Monitoring[Monitoring<br/>Servers]
        Backup[Backup<br/>Servers]
    end
    
    Internet[Internet] --> PublicLB
    PublicLB --> WAF
    WAF --> AppServers
    
    Internet -.->|SIP/RTP| VoIPServers
    
    AppServers --> Databases
    AppServers --> Cache
    AppServers --> Storage
    
    VoIPServers --> Databases
    VoIPServers --> Cache
    
    Bastion -.->|SSH| AppServers
    Bastion -.->|SSH| VoIPServers
    Bastion -.->|SSH| Databases
    
    Monitoring -.->|Metrics| AppServers
    Monitoring -.->|Metrics| VoIPServers
    Monitoring -.->|Metrics| Databases
    
    Backup -.->|Backup| Databases
    Backup -.->|Backup| Storage

    classDef publicStyle fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    classDef appStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef dataStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef mgmtStyle fill:#fff9c4,stroke:#f57f17,stroke-width:2px

    class PublicLB,WAF publicStyle
    class AppServers,VoIPServers appStyle
    class Databases,Cache,Storage dataStyle
    class Bastion,Monitoring,Backup mgmtStyle
```

## Scaling Strategy

```mermaid
graph LR
    subgraph Metrics["Monitoring Metrics"]
        CPU[CPU Usage > 70%]
        Memory[Memory > 80%]
        Requests[Requests/sec > 1000]
        QueueDepth[Queue Depth > 100]
    end
    
    subgraph AutoScaling["Auto-Scaling Triggers"]
        ScaleUp[Scale Up Decision]
        ScaleDown[Scale Down Decision]
    end
    
    subgraph Actions["Scaling Actions"]
        AddApp[Add App Server]
        RemoveApp[Remove App Server]
        AddVoIP[Add VoIP Server]
        AddWorker[Add Queue Worker]
    end
    
    subgraph LoadBalancer["Load Balancer"]
        UpdatePool[Update Server Pool]
        HealthCheck[Health Checks]
    end
    
    CPU --> ScaleUp
    Memory --> ScaleUp
    Requests --> ScaleUp
    QueueDepth --> ScaleUp
    
    CPU --> ScaleDown
    Memory --> ScaleDown
    Requests --> ScaleDown
    
    ScaleUp --> AddApp
    ScaleUp --> AddVoIP
    ScaleUp --> AddWorker
    
    ScaleDown --> RemoveApp
    
    AddApp --> UpdatePool
    RemoveApp --> UpdatePool
    AddVoIP --> UpdatePool
    
    UpdatePool --> HealthCheck

    classDef metricStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef scaleStyle fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef actionStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef lbStyle fill:#f3e5f5,stroke:#4a148c,stroke-width:2px

    class CPU,Memory,Requests,QueueDepth metricStyle
    class ScaleUp,ScaleDown scaleStyle
    class AddApp,RemoveApp,AddVoIP,AddWorker actionStyle
    class UpdatePool,HealthCheck lbStyle
```

## Server Specifications

### Production Environment

| Component | Quantity | Specs | Purpose |
|-----------|----------|-------|---------|
| **Load Balancer** | 2 | 4 vCPU, 8GB RAM | High availability |
| **Laravel App** | 3-10 | 8 vCPU, 16GB RAM | Auto-scaling |
| **MySQL Primary** | 1 | 8 vCPU, 32GB RAM, 1TB SSD | Main database |
| **MySQL Replica** | 2 | 8 vCPU, 32GB RAM, 1TB SSD | Read replicas |
| **Redis Cluster** | 3 | 4 vCPU, 8GB RAM | Cache & queues |
| **Kamailio** | 2 | 8 vCPU, 16GB RAM | SIP proxy HA |
| **FreeSWITCH** | 2-4 | 16 vCPU, 32GB RAM | Media servers |
| **RTPEngine** | 2-4 | 8 vCPU, 16GB RAM | RTP proxy |
| **Jasmin SMS** | 2 | 4 vCPU, 8GB RAM | SMS gateway |
| **RabbitMQ** | 3 | 4 vCPU, 8GB RAM | Message queue |
| **Monitoring** | 2 | 4 vCPU, 8GB RAM | Prometheus/Grafana |
| **Logging** | 3 | 8 vCPU, 16GB RAM | ELK stack |

### Development Environment

| Component | Quantity | Specs |
|-----------|----------|-------|
| **All-in-One** | 1 | 16 vCPU, 32GB RAM |

### Staging Environment

| Component | Quantity | Specs |
|-----------|----------|-------|
| **Load Balancer** | 1 | 2 vCPU, 4GB RAM |
| **Laravel App** | 2 | 4 vCPU, 8GB RAM |
| **MySQL** | 1 | 4 vCPU, 16GB RAM |
| **Redis** | 1 | 2 vCPU, 4GB RAM |
| **VoIP Stack** | 1 | 8 vCPU, 16GB RAM |

## Deployment Strategy

### Blue-Green Deployment
1. **Blue Environment**: Current production
2. **Green Environment**: New version deployed
3. **Testing**: Validate green environment
4. **Switch**: Update load balancer to green
5. **Rollback**: Switch back to blue if issues

### Rolling Deployment
1. Deploy to 1 server
2. Health check passes
3. Deploy to next server
4. Repeat until all updated
5. Automatic rollback on failure

## Backup Strategy

### Database Backups
- **Full Backup**: Daily at 2 AM UTC
- **Incremental**: Every 6 hours
- **Retention**: 30 days
- **Location**: S3 with cross-region replication

### File Backups
- **Media Files**: Continuous S3 replication
- **Configuration**: Git repository
- **Retention**: 90 days

### Disaster Recovery
- **RTO**: 4 hours (Recovery Time Objective)
- **RPO**: 1 hour (Recovery Point Objective)
- **DR Site**: Secondary region ready
- **Testing**: Quarterly DR drills
