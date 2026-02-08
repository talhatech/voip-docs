# Step 9: White-Label Configuration - Define Customization Options

## Overview

This document defines the comprehensive white-labeling and customization options for the VoIP platform, enabling clients to fully brand and customize the platform as their own product.

## White-Label Philosophy

**Goal:** Enable clients to launch their own branded VoIP service without building from scratch.

**Approach:**
- Complete branding control
- Flexible feature configuration
- Multiple deployment models
- Scalable architecture
- Minimal technical expertise required

---

## Customization Layers

### 1. Branding & Visual Identity

#### 1.1 Brand Assets

**Customizable Elements:**

| Element | Format | Usage |
|---------|--------|-------|
| **App Name** | Text | Mobile apps, web portal, emails |
| **Company Name** | Text | Legal documents, support |
| **Logo (Primary)** | SVG, PNG (512x512) | App icon, header, splash screen |
| **Logo (Secondary)** | SVG, PNG | Footer, emails, documents |
| **Favicon** | ICO, PNG (32x32) | Browser tab |
| **Splash Screen** | PNG (1080x1920) | Mobile app launch |
| **App Icon** | PNG (1024x1024) | iOS/Android app stores |

**Configuration:**
```json
{
  "branding": {
    "app_name": "MyVoIP",
    "company_name": "MyVoIP Inc.",
    "tagline": "Communication Made Simple",
    "logo_url": "https://cdn.example.com/logo.svg",
    "logo_dark_url": "https://cdn.example.com/logo-dark.svg",
    "favicon_url": "https://cdn.example.com/favicon.ico",
    "splash_screen_url": "https://cdn.example.com/splash.png",
    "app_icon_url": "https://cdn.example.com/icon.png"
  }
}
```

#### 1.2 Color Scheme

**Customizable Colors:**

```json
{
  "colors": {
    "primary": "#2563EB",
    "secondary": "#10B981",
    "accent": "#F59E0B",
    "success": "#10B981",
    "warning": "#F59E0B",
    "error": "#EF4444",
    "background": "#FFFFFF",
    "surface": "#F9FAFB",
    "text_primary": "#111827",
    "text_secondary": "#6B7280",
    "border": "#E5E7EB"
  },
  "dark_mode": {
    "background": "#111827",
    "surface": "#1F2937",
    "text_primary": "#F9FAFB",
    "text_secondary": "#9CA3AF",
    "border": "#374151"
  }
}
```

**Color Picker UI:**
- Visual color picker
- Preset themes (Professional, Vibrant, Minimal)
- Preview mode
- Accessibility checker (contrast ratio)

#### 1.3 Typography

**Customizable Fonts:**

```json
{
  "typography": {
    "font_family": "Inter",
    "font_url": "https://fonts.googleapis.com/css2?family=Inter",
    "headings": {
      "font_family": "Inter",
      "font_weight": "700"
    },
    "body": {
      "font_family": "Inter",
      "font_weight": "400"
    }
  }
}
```

**Supported Fonts:**
- Google Fonts library
- Custom font upload
- System fonts

---

### 2. Content Customization

#### 2.1 Text & Messaging

**Customizable Text:**

| Category | Examples |
|----------|----------|
| **Welcome Messages** | Onboarding screens, first login |
| **Email Templates** | Welcome email, password reset, receipts |
| **Push Notifications** | Call missed, message received |
| **In-App Messages** | Tooltips, help text, errors |
| **Legal Documents** | Terms of Service, Privacy Policy |

**Configuration:**
```json
{
  "content": {
    "welcome_message": "Welcome to MyVoIP!",
    "onboarding_title_1": "Make Calls Anywhere",
    "onboarding_description_1": "Connect with anyone using virtual numbers",
    "support_email": "support@myvoip.com",
    "support_phone": "+1-800-MYVOIP",
    "terms_url": "https://myvoip.com/terms",
    "privacy_url": "https://myvoip.com/privacy"
  }
}
```

#### 2.2 Email Templates

**Customizable Templates:**
- Welcome email
- Email verification
- Password reset
- Invoice/receipt
- Number purchased
- Low balance alert
- Call/SMS notifications

**Template Variables:**
```html
<!DOCTYPE html>
<html>
<head>
    <style>
        /* Custom styles */
    </style>
</head>
<body>
    <div class="header">
        <img src="{{logo_url}}" alt="{{app_name}}">
    </div>
    <div class="content">
        <h1>Welcome to {{app_name}}!</h1>
        <p>Hi {{user_name}},</p>
        <p>{{welcome_message}}</p>
        <a href="{{action_url}}" class="button">Get Started</a>
    </div>
    <div class="footer">
        <p>{{company_name}} | {{support_email}}</p>
    </div>
</body>
</html>
```

---

### 3. Feature Configuration

#### 3.1 Feature Toggles

**Customizable Features:**

```json
{
  "features": {
    "voice_calls": true,
    "video_calls": false,
    "sms": true,
    "mms": true,
    "voicemail": true,
    "call_recording": true,
    "call_forwarding": true,
    "do_not_disturb": true,
    "scheduled_sms": true,
    "group_messaging": false,
    "conference_calling": false,
    "number_porting": true,
    "multi_number_support": true,
    "max_numbers_per_user": 5,
    "international_calling": true,
    "toll_free_numbers": true
  }
}
```

**Admin UI:**
- Toggle switches for each feature
- Feature dependencies (e.g., call recording requires voice calls)
- Impact warnings

#### 3.2 Pricing Configuration

**Customizable Pricing:**

```json
{
  "pricing": {
    "subscription_plans": [
      {
        "id": "basic",
        "name": "Basic",
        "price": 9.99,
        "currency": "USD",
        "billing_period": "monthly",
        "features": {
          "numbers": 1,
          "minutes": 100,
          "sms": 100
        }
      },
      {
        "id": "plus",
        "name": "Plus",
        "price": 19.99,
        "currency": "USD",
        "billing_period": "monthly",
        "features": {
          "numbers": 3,
          "minutes": 500,
          "sms": 500
        }
      }
    ],
    "pay_as_you_go": {
      "per_minute": 0.02,
      "per_sms": 0.02,
      "per_mms": 0.05,
      "number_rental": 2.99
    }
  }
}
```

#### 3.3 Payment Gateways

**Supported Gateways:**

```json
{
  "payment_gateways": {
    "stripe": {
      "enabled": true,
      "public_key": "pk_live_...",
      "secret_key": "sk_live_..."
    },
    "paypal": {
      "enabled": true,
      "client_id": "...",
      "secret": "..."
    },
    "apple_pay": {
      "enabled": true,
      "merchant_id": "..."
    },
    "google_pay": {
      "enabled": false
    }
  }
}
```

---

### 4. Domain & URLs

#### 4.1 Custom Domains

**Customizable Domains:**

| Service | Default | Custom Example |
|---------|---------|----------------|
| **Web Portal** | portal.voipplatform.com | portal.myvoip.com |
| **API** | api.voipplatform.com | api.myvoip.com |
| **Marketing Site** | www.voipplatform.com | www.myvoip.com |
| **CDN** | cdn.voipplatform.com | cdn.myvoip.com |

**Configuration:**
```json
{
  "domains": {
    "web_portal": "portal.myvoip.com",
    "api": "api.myvoip.com",
    "marketing": "www.myvoip.com",
    "cdn": "cdn.myvoip.com"
  }
}
```

**SSL Certificates:**
- Auto-provision via Let's Encrypt
- Custom certificate upload
- Wildcard certificate support

#### 4.2 Deep Links (Mobile Apps)

**Custom URL Schemes:**
```json
{
  "deep_links": {
    "ios_scheme": "myvoip://",
    "android_scheme": "myvoip://",
    "universal_link": "https://myvoip.com/app"
  }
}
```

---

### 5. Regional Configuration

#### 5.1 Localization

**Supported Languages:**
- English (default)
- Spanish
- French
- German
- Arabic
- Chinese
- Japanese
- (Add more as needed)

**Configuration:**
```json
{
  "localization": {
    "default_language": "en",
    "supported_languages": ["en", "es", "fr", "de", "ar"],
    "currency": "USD",
    "date_format": "MM/DD/YYYY",
    "time_format": "12h",
    "timezone": "America/New_York"
  }
}
```

#### 5.2 Regional Restrictions

**Customizable Restrictions:**

```json
{
  "regional_settings": {
    "allowed_countries": ["US", "CA", "GB", "AU"],
    "blocked_countries": ["KP", "IR"],
    "default_country": "US",
    "number_availability": {
      "US": true,
      "CA": true,
      "GB": true,
      "AU": false
    }
  }
}
```

---

### 6. Mobile App Customization

#### 6.1 App Store Metadata

**Customizable Metadata:**

```json
{
  "app_store": {
    "ios": {
      "app_name": "MyVoIP",
      "subtitle": "Virtual Phone Numbers",
      "description": "...",
      "keywords": "voip, phone, calls, sms",
      "category": "Business",
      "bundle_id": "com.myvoip.app",
      "app_store_url": "https://apps.apple.com/app/myvoip/id123456789"
    },
    "android": {
      "app_name": "MyVoIP",
      "short_description": "...",
      "full_description": "...",
      "category": "Communication",
      "package_name": "com.myvoip.app",
      "play_store_url": "https://play.google.com/store/apps/details?id=com.myvoip.app"
    }
  }
}
```

#### 6.2 App Configuration

**Build-time Configuration:**
```json
{
  "mobile_app": {
    "api_base_url": "https://api.myvoip.com",
    "websocket_url": "wss://api.myvoip.com/ws",
    "pusher_key": "...",
    "sentry_dsn": "...",
    "analytics_key": "...",
    "certificate_pinning": true,
    "ssl_pinning_hashes": ["sha256/..."]
  }
}
```

---

## Deployment Models

### 1. SaaS (Multi-Tenant)

**Description:** Shared infrastructure, separate databases per client

**Pros:**
- ✅ Lower cost
- ✅ Faster deployment
- ✅ Automatic updates
- ✅ Managed by platform

**Cons:**
- ⚠️ Shared resources
- ⚠️ Less control

**Pricing:** $X/month + per-user fees

### 2. Dedicated (Single-Tenant)

**Description:** Dedicated infrastructure per client

**Pros:**
- ✅ Isolated resources
- ✅ Better performance
- ✅ More control
- ✅ Custom scaling

**Cons:**
- ⚠️ Higher cost
- ⚠️ Longer setup time

**Pricing:** $Y/month + infrastructure costs

### 3. On-Premise

**Description:** Client hosts on their own infrastructure

**Pros:**
- ✅ Full control
- ✅ Data sovereignty
- ✅ Custom compliance

**Cons:**
- ⚠️ Client manages infrastructure
- ⚠️ Manual updates
- ⚠️ Higher technical requirements

**Pricing:** One-time license fee + annual support

### 4. Hybrid

**Description:** Mix of cloud and on-premise

**Pros:**
- ✅ Flexibility
- ✅ Compliance + convenience

**Cons:**
- ⚠️ Complex setup
- ⚠️ Higher cost

---

## Configuration Management

### Admin Panel

**White-Label Configuration UI:**

```
┌─────────────────────────────────────┐
│  White-Label Configuration          │
├─────────────────────────────────────┤
│                                     │
│  [Branding] [Features] [Pricing]    │
│  [Domains] [Regional] [Advanced]    │
│                                     │
│  ┌─ Branding ─────────────────────┐ │
│  │                                │ │
│  │ App Name: [MyVoIP            ] │ │
│  │ Company:  [MyVoIP Inc.       ] │ │
│  │                                │ │
│  │ Logo:     [Upload] [Preview]   │ │
│  │ Colors:   [Primary: #2563EB]   │ │
│  │           [Secondary: #10B981] │ │
│  │                                │ │
│  │ [Preview Changes]              │ │
│  │ [Save Configuration]           │ │
│  └────────────────────────────────┘ │
└─────────────────────────────────────┘
```

### Configuration API

**Endpoints:**

```php
// Get configuration
GET /api/v1/admin/white-label/config

// Update configuration
PUT /api/v1/admin/white-label/config
{
  "branding": { ... },
  "features": { ... },
  "pricing": { ... }
}

// Upload assets
POST /api/v1/admin/white-label/assets
Content-Type: multipart/form-data

// Preview configuration
POST /api/v1/admin/white-label/preview
```

### Configuration Storage

**Database Schema:**

```sql
CREATE TABLE white_label_configs (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    client_id BIGINT NOT NULL,
    config_key VARCHAR(255) NOT NULL,
    config_value JSON NOT NULL,
    version INT NOT NULL DEFAULT 1,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY (client_id, config_key)
);

CREATE TABLE white_label_assets (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    client_id BIGINT NOT NULL,
    asset_type ENUM('logo', 'icon', 'splash', 'favicon') NOT NULL,
    file_url VARCHAR(500) NOT NULL,
    file_size INT,
    mime_type VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Implementation Guide

### Step 1: Initial Setup

```bash
# Create white-label configuration
php artisan white-label:create --client="MyVoIP Inc."

# Generate configuration template
php artisan white-label:generate-config --client-id=1

# Upload assets
php artisan white-label:upload-assets --client-id=1 \
    --logo=logo.svg \
    --icon=icon.png \
    --splash=splash.png
```

### Step 2: Build Mobile Apps

```bash
# Configure build
php artisan white-label:configure-build --client-id=1

# Build iOS app
cd mobile/ios
fastlane build_white_label client_id:1

# Build Android app
cd mobile/android
./gradlew assembleWhiteLabel -PclientId=1
```

### Step 3: Deploy

```bash
# Deploy web portal
php artisan white-label:deploy-portal --client-id=1

# Configure domains
php artisan white-label:configure-domains --client-id=1

# Provision SSL certificates
php artisan white-label:provision-ssl --client-id=1
```

---

## Checklist

- [ ] Define all customization options
- [ ] Create configuration schema
- [ ] Build admin configuration UI
- [ ] Implement configuration API
- [ ] Create asset upload system
- [ ] Build preview system
- [ ] Implement multi-tenancy
- [ ] Create deployment automation
- [ ] Document white-label process
- [ ] Create client onboarding guide
- [ ] Set up demo environment
- [ ] Define pricing for each deployment model
- [ ] Create white-label agreement template
- [ ] Test complete white-label flow
- [ ] Train support team

---

*Document Version: 1.0*  
*Created: February 2026*  
*Status: Planning Document*
