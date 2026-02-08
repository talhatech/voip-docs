# White-Labeling Components

## Overview
Complete white-labeling system allowing full customization of the platform for different brands.

## White-Label Architecture

```mermaid
graph TB
    subgraph Configuration["White-Label Configuration"]
        Config[Configuration<br/>Management]
        Database[(Tenant Config<br/>Database)]
    end
    
    subgraph Branding["Branding Elements"]
        AppName[App Name]
        Logo[Logo/Icon]
        Colors[Color Scheme]
        Typography[Typography]
        Splash[Splash Screen]
    end
    
    subgraph Content["Content Customization"]
        Welcome[Welcome Messages]
        Onboarding[Onboarding Screens]
        HelpText[Help/Support Text]
        EmailTemplates[Email Templates]
        PushTemplates[Push Notifications]
    end
    
    subgraph Features["Feature Toggles"]
        VoiceToggle[Voice Calling]
        SMSToggle[SMS/MMS]
        ScheduledToggle[Scheduled SMS]
        ForwardingToggle[Call Forwarding]
        PortingToggle[Number Porting]
        MultiNumberToggle[Multi-Number]
    end
    
    subgraph Payments["Payment Configuration"]
        StripeConfig[Stripe]
        PayPalConfig[PayPal]
        ApplePayConfig[Apple Pay]
        CashAppConfig[Cash App]
        AmazonPayConfig[Amazon Pay]
    end
    
    subgraph Domains["Domain Configuration"]
        WebDomain[Web Portal Domain]
        MarketingDomain[Marketing Site]
        APIDomain[API Subdomain]
        DeepLinks[Mobile Deep Links]
    end
    
    Config --> Database
    
    Database --> Branding
    Database --> Content
    Database --> Features
    Database --> Payments
    Database --> Domains

    classDef configStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef brandStyle fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef contentStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef featureStyle fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef paymentStyle fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    classDef domainStyle fill:#ffcdd2,stroke:#c62828,stroke-width:2px

    class Config,Database configStyle
    class AppName,Logo,Colors,Typography,Splash brandStyle
    class Welcome,Onboarding,HelpText,EmailTemplates,PushTemplates contentStyle
    class VoiceToggle,SMSToggle,ScheduledToggle,ForwardingToggle,PortingToggle,MultiNumberToggle featureStyle
    class StripeConfig,PayPalConfig,ApplePayConfig,CashAppConfig,AmazonPayConfig paymentStyle
    class WebDomain,MarketingDomain,APIDomain,DeepLinks domainStyle
```

## Tenant Configuration Structure

```mermaid
graph LR
    Tenant[Tenant/Client] --> TenantConfig[Tenant Configuration]
    
    TenantConfig --> BrandingConfig[Branding Config]
    TenantConfig --> FeatureConfig[Feature Config]
    TenantConfig --> PaymentConfig[Payment Config]
    TenantConfig --> DomainConfig[Domain Config]
    TenantConfig --> PricingConfig[Pricing Config]
    TenantConfig --> RegionConfig[Region Config]
    
    BrandingConfig --> AppIdentity[App Identity<br/>Name, Logo, Icon]
    BrandingConfig --> ColorScheme[Color Scheme<br/>Primary, Secondary, Accent]
    BrandingConfig --> Fonts[Typography<br/>Font Families]
    BrandingConfig --> Assets[Custom Assets<br/>Images, Icons]
    
    FeatureConfig --> EnabledFeatures[Enabled Features<br/>Voice, SMS, MMS, etc.]
    FeatureConfig --> FeatureLimits[Feature Limits<br/>Max Numbers, etc.]
    
    PaymentConfig --> Gateways[Payment Gateways<br/>Stripe, PayPal, etc.]
    PaymentConfig --> Currency[Default Currency<br/>USD, EUR, etc.]
    
    DomainConfig --> CustomDomains[Custom Domains<br/>Web, API, Marketing]
    DomainConfig --> SSLCerts[SSL Certificates]
    
    PricingConfig --> CallRates[Call Rates]
    PricingConfig --> MessageRates[Message Rates]
    PricingConfig --> NumberPrices[Number Prices]
    
    RegionConfig --> SupportedRegions[Supported Regions]
    RegionConfig --> Carriers[Carrier Selection]

    classDef tenantStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef configStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef detailStyle fill:#fff3e0,stroke:#e65100,stroke-width:2px

    class Tenant,TenantConfig tenantStyle
    class BrandingConfig,FeatureConfig,PaymentConfig,DomainConfig,PricingConfig,RegionConfig configStyle
    class AppIdentity,ColorScheme,Fonts,Assets,EnabledFeatures,FeatureLimits,Gateways,Currency,CustomDomains,SSLCerts,CallRates,MessageRates,NumberPrices,SupportedRegions,Carriers detailStyle
```

## Multi-Tenant Data Isolation

```mermaid
graph TB
    subgraph SharedInfra["Shared Infrastructure"]
        LB[Load Balancer]
        AppServers[Application Servers]
        VoIPServers[VoIP Servers]
    end
    
    subgraph TenantA["Tenant A (Brand X)"]
        ConfigA[Config A]
        DomainA[brandx.com]
        ThemeA[Theme: Blue]
        UsersA[(Users A)]
    end
    
    subgraph TenantB["Tenant B (Brand Y)"]
        ConfigB[Config B]
        DomainB[brandy.com]
        ThemeB[Theme: Red]
        UsersB[(Users B)]
    end
    
    subgraph TenantC["Tenant C (Brand Z)"]
        ConfigC[Config C]
        DomainC[brandz.com]
        ThemeC[Theme: Green]
        UsersC[(Users C)]
    end
    
    subgraph SharedData["Shared Data Layer"]
        SharedDB[(Shared Database<br/>with tenant_id)]
        SharedCache[(Shared Cache)]
        SharedStorage[Shared Storage]
    end
    
    LB --> AppServers
    LB --> VoIPServers
    
    DomainA --> LB
    DomainB --> LB
    DomainC --> LB
    
    AppServers --> ConfigA
    AppServers --> ConfigB
    AppServers --> ConfigC
    
    ConfigA --> UsersA
    ConfigB --> UsersB
    ConfigC --> UsersC
    
    UsersA --> SharedDB
    UsersB --> SharedDB
    UsersC --> SharedDB
    
    AppServers --> SharedCache
    AppServers --> SharedStorage

    classDef infraStyle fill:#e0e0e0,stroke:#424242,stroke-width:2px
    classDef tenantAStyle fill:#bbdefb,stroke:#1976d2,stroke-width:2px
    classDef tenantBStyle fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    classDef tenantCStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef dataStyle fill:#fff9c4,stroke:#f57f17,stroke-width:2px

    class LB,AppServers,VoIPServers infraStyle
    class ConfigA,DomainA,ThemeA,UsersA tenantAStyle
    class ConfigB,DomainB,ThemeB,UsersB tenantBStyle
    class ConfigC,DomainC,ThemeC,UsersC tenantCStyle
    class SharedDB,SharedCache,SharedStorage dataStyle
```

## Branding Customization Flow

```mermaid
sequenceDiagram
    participant Admin as Tenant Admin
    participant Portal as Admin Portal
    participant API as Laravel API
    participant CDN as CDN/Storage
    participant App as Mobile App

    Admin->>Portal: 1. Access Branding Settings
    
    Portal->>API: 2. GET /admin/branding
    
    API-->>Portal: 3. Current Branding Config
    
    Admin->>Portal: 4. Upload New Logo
    
    Portal->>API: 5. POST /admin/branding/logo<br/>{file}
    
    activate API
    API->>CDN: 6. Upload to CDN
    
    CDN-->>API: 7. CDN URL
    
    API->>API: 8. Update Config<br/>logo_url = cdn_url
    
    API-->>Portal: 9. Logo Updated
    deactivate API
    
    Admin->>Portal: 10. Update Color Scheme<br/>Primary: #1976D2
    
    Portal->>API: 11. PUT /admin/branding/colors<br/>{primary, secondary, accent}
    
    activate API
    API->>API: 12. Update Config<br/>Save Colors
    
    API-->>Portal: 13. Colors Updated
    deactivate API
    
    Admin->>Portal: 14. Publish Changes
    
    Portal->>API: 15. POST /admin/branding/publish
    
    activate API
    API->>API: 16. Generate Config Bundle
    
    API->>CDN: 17. Deploy Config
    
    API->>App: 18. Push Notification:<br/>New Branding Available
    
    API-->>Portal: 19. Published Successfully
    deactivate API
    
    App->>API: 20. GET /config/branding
    
    API-->>App: 21. New Branding Config
    
    Note over App: 22. Apply New Branding<br/>Restart UI
```

## Deployment Models

```mermaid
graph TB
    subgraph SaaS["SaaS Model"]
        SaaSInfra[Shared Infrastructure]
        SaaSTenants[Multiple Tenants]
        SaaSDB[(Single Database<br/>tenant_id column)]
        
        SaaSInfra --> SaaSTenants
        SaaSTenants --> SaaSDB
    end
    
    subgraph Dedicated["Dedicated Model"]
        DedInfra[Dedicated Infrastructure]
        DedTenant[Single Tenant]
        DedDB[(Dedicated Database)]
        
        DedInfra --> DedTenant
        DedTenant --> DedDB
    end
    
    subgraph OnPrem["On-Premise Model"]
        OnPremInfra[Client Infrastructure]
        OnPremTenant[Single Tenant]
        OnPremDB[(Client Database)]
        
        OnPremInfra --> OnPremTenant
        OnPremTenant --> OnPremDB
    end
    
    subgraph Hybrid["Hybrid Model"]
        HybridCore[Core on Cloud]
        HybridData[Data on Client]
        HybridVoIP[VoIP on Client]
        
        HybridCore --> HybridData
        HybridCore --> HybridVoIP
    end

    classDef saasStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef dedicatedStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef onpremStyle fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef hybridStyle fill:#f3e5f5,stroke:#4a148c,stroke-width:2px

    class SaaSInfra,SaaSTenants,SaaSDB saasStyle
    class DedInfra,DedTenant,DedDB dedicatedStyle
    class OnPremInfra,OnPremTenant,OnPremDB onpremStyle
    class HybridCore,HybridData,HybridVoIP hybridStyle
```

## Configuration Management

### Configuration Hierarchy
```
Global Config (Platform-wide)
  └── Tenant Config (Brand-specific)
      └── User Config (User preferences)
```

### Configuration Storage
```json
{
  "tenant_id": "brand_x",
  "branding": {
    "app_name": "Brand X VoIP",
    "company_name": "Brand X Inc.",
    "logo_url": "https://cdn.example.com/brandx/logo.png",
    "icon_url": "https://cdn.example.com/brandx/icon.png",
    "splash_screen_url": "https://cdn.example.com/brandx/splash.png",
    "colors": {
      "primary": "#1976D2",
      "secondary": "#424242",
      "accent": "#FF4081"
    },
    "typography": {
      "primary_font": "Roboto",
      "secondary_font": "Open Sans"
    }
  },
  "features": {
    "voice_calling": true,
    "sms": true,
    "mms": true,
    "scheduled_sms": true,
    "call_forwarding": true,
    "number_porting": true,
    "multi_number": true,
    "max_numbers_per_user": 5
  },
  "payment_gateways": {
    "stripe": {
      "enabled": true,
      "public_key": "pk_live_...",
      "webhook_secret": "whsec_..."
    },
    "paypal": {
      "enabled": true,
      "client_id": "..."
    }
  },
  "domains": {
    "web_portal": "portal.brandx.com",
    "marketing_site": "www.brandx.com",
    "api": "api.brandx.com"
  },
  "support": {
    "email": "support@brandx.com",
    "phone": "+1-800-BRANDX",
    "url": "https://help.brandx.com"
  }
}
```

## Customizable Components

### Mobile App
- App name and bundle ID
- App icon and splash screen
- Color scheme
- Typography
- Onboarding flow
- Feature availability
- Deep link scheme

### Web Portal
- Domain and branding
- Theme colors
- Logo and favicon
- Custom CSS
- Feature toggles
- Help documentation

### Marketing Site
- Custom domain
- Complete design control
- SEO settings
- Analytics integration
- Content management

### Email Templates
- Branded email design
- Custom sender name/email
- Localized content
- Dynamic content blocks

## White-Label Management

### Admin Capabilities
- Create new tenant
- Configure branding
- Enable/disable features
- Set pricing
- Manage domains
- Monitor usage
- Generate reports

### Tenant Isolation
- Data segregation by tenant_id
- Separate file storage paths
- Isolated cache keys
- Tenant-specific queues
- Independent rate limits

### Billing
- Per-tenant billing
- Custom pricing models
- Revenue sharing options
- Usage-based pricing
- Subscription tiers
