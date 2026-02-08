# Step 6: Security Architecture Review - Before Implementation

## Overview

This document outlines the comprehensive security architecture review process that must be completed before implementing the VoIP platform. Security is critical for protecting user data, preventing fraud, and ensuring compliance.

## Security Review Objectives

1. ✅ Identify potential security vulnerabilities
2. ✅ Define security controls and mitigations
3. ✅ Establish security best practices
4. ✅ Ensure compliance with regulations (GDPR)
5. ✅ Plan for VAPT (Vulnerability Assessment & Penetration Testing)
6. ✅ Document security policies and procedures

---

## Security Domains

### 1. Application Security

#### 1.1 Authentication & Authorization

**Requirements:**
- Multi-factor authentication (MFA) support
- Secure password storage (bcrypt/Argon2)
- JWT token management
- Session management
- Role-based access control (RBAC)
- API key authentication for public endpoints

**Implementation:**
```php
// Password hashing
use Illuminate\Support\Facades\Hash;
$hashedPassword = Hash::make($password, ['rounds' => 12]);

// JWT token generation
use Laravel\Sanctum\HasApiTokens;
$token = $user->createToken('device-name')->plainTextToken;

// Role-based access
use Spatie\Permission\Traits\HasRoles;
$user->assignRole('admin');
$user->hasPermissionTo('manage-numbers');
```

**Security Controls:**
- Password complexity requirements (min 8 chars, uppercase, lowercase, number, special char)
- Account lockout after 5 failed attempts
- Token expiration (access: 15 min, refresh: 7 days)
- Secure token storage (HttpOnly cookies for web)
- IP whitelisting for admin panel

#### 1.2 Input Validation & Sanitization

**Threats:**
- SQL Injection
- XSS (Cross-Site Scripting)
- Command Injection
- Path Traversal
- XML External Entity (XXE)

**Mitigations:**
```php
// Laravel validation
$validated = $request->validate([
    'email' => 'required|email|max:255',
    'phone' => 'required|regex:/^\+[1-9]\d{1,14}$/',
    'amount' => 'required|numeric|min:0|max:10000',
]);

// Sanitize output
{{ $userInput }} // Auto-escaped in Blade
{!! $trustedHtml !!} // Only for trusted content

// Parameterized queries (prevent SQL injection)
DB::table('users')->where('email', $email)->first();
```

#### 1.3 API Security

**Requirements:**
- Rate limiting
- Request throttling
- API versioning
- Input validation
- Output encoding
- CORS configuration
- API key rotation

**Implementation:**
```php
// Rate limiting
Route::middleware('throttle:60,1')->group(function () {
    // 60 requests per minute
});

// API key middleware
class VerifyApiKey
{
    public function handle($request, Closure $next)
    {
        $apiKey = $request->header('X-API-Key');
        
        if (!ApiKey::where('key', $apiKey)->where('is_active', true)->exists()) {
            return response()->json(['error' => 'Invalid API key'], 401);
        }
        
        return $next($request);
    }
}

// CORS configuration
'cors' => [
    'paths' => ['api/*'],
    'allowed_origins' => ['https://app.voipplatform.com'],
    'allowed_methods' => ['GET', 'POST', 'PUT', 'DELETE'],
    'allowed_headers' => ['Content-Type', 'Authorization'],
    'max_age' => 3600,
],
```

---

### 2. Network Security

#### 2.1 Transport Layer Security

**Requirements:**
- TLS 1.3 for all connections
- HTTPS enforcement
- Certificate management
- HSTS (HTTP Strict Transport Security)
- Certificate pinning (mobile apps)

**Implementation:**
```nginx
# Nginx TLS configuration
server {
    listen 443 ssl http2;
    server_name api.voipplatform.com;
    
    ssl_certificate /etc/letsencrypt/live/voipplatform.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/voipplatform.com/privkey.pem;
    
    ssl_protocols TLSv1.3 TLSv1.2;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256';
    ssl_prefer_server_ciphers on;
    
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
}
```

#### 2.2 VoIP Security

**Requirements:**
- SRTP (Secure RTP) for media encryption
- TLS for SIP signaling
- SIP authentication
- Firewall rules
- DDoS protection
- Toll fraud prevention

**Implementation:**
```cfg
# Kamailio TLS configuration
loadmodule "tls.so"

modparam("tls", "config", "/etc/kamailio/tls.cfg")
modparam("tls", "tls_method", "TLSv1.2+")
modparam("tls", "verify_certificate", 1)

# SRTP enforcement
if (is_method("INVITE")) {
    if (!has_body("application/sdp")) {
        sl_send_reply("415", "Unsupported Media Type");
        exit;
    }
    
    # Require SRTP
    if (!sdp_with_transport("RTP/SAVP")) {
        sl_send_reply("488", "SRTP Required");
        exit;
    }
}
```

**Toll Fraud Prevention:**
```php
class TollFraudDetection
{
    public function detectFraud(Call $call): bool
    {
        // Check for suspicious patterns
        $checks = [
            $this->checkCallVolume($call->user_id),
            $this->checkDestinationPattern($call->to_number),
            $this->checkCallDuration($call),
            $this->checkGeolocation($call),
            $this->checkCostThreshold($call),
        ];
        
        return in_array(true, $checks);
    }
    
    protected function checkCallVolume(int $userId): bool
    {
        // More than 50 calls in 1 hour
        $count = Call::where('user_id', $userId)
            ->where('created_at', '>', now()->subHour())
            ->count();
            
        return $count > 50;
    }
    
    protected function checkDestinationPattern(string $number): bool
    {
        // High-cost destinations
        $highCostPrefixes = ['+881', '+882', '+883']; // Satellite phones
        
        foreach ($highCostPrefixes as $prefix) {
            if (str_starts_with($number, $prefix)) {
                return true;
            }
        }
        
        return false;
    }
}
```

---

### 3. Data Security

#### 3.1 Encryption at Rest

**Requirements:**
- Database encryption
- File encryption
- Backup encryption
- Key management

**Implementation:**
```php
// Laravel encryption
use Illuminate\Support\Facades\Crypt;

// Encrypt sensitive data
$encrypted = Crypt::encryptString($sensitiveData);

// Decrypt
$decrypted = Crypt::decryptString($encrypted);

// Database column encryption
class User extends Model
{
    protected $casts = [
        'ssn' => 'encrypted',
        'credit_card' => 'encrypted',
    ];
}
```

**MySQL Encryption:**
```sql
-- Enable encryption at rest
ALTER TABLE users ENCRYPTION='Y';
ALTER TABLE phone_numbers ENCRYPTION='Y';
ALTER TABLE calls ENCRYPTION='Y';
```

#### 3.2 Encryption in Transit

**Requirements:**
- TLS 1.3 for API
- SRTP for VoIP media
- Encrypted database connections
- Encrypted Redis connections

**Implementation:**
```env
# Database SSL
DB_SSL_CA=/path/to/ca-cert.pem
DB_SSL_CERT=/path/to/client-cert.pem
DB_SSL_KEY=/path/to/client-key.pem

# Redis TLS
REDIS_SCHEME=tls
REDIS_TLS_CA=/path/to/ca-cert.pem
```

#### 3.3 Data Retention & Deletion

**GDPR Requirements:**
- Right to be forgotten
- Data minimization
- Purpose limitation
- Storage limitation

**Implementation:**
```php
class DataRetentionService
{
    public function deleteUserData(User $user): void
    {
        DB::transaction(function () use ($user) {
            // Delete or anonymize user data
            $user->calls()->delete();
            $user->smsMessages()->delete();
            $user->mmsMessages()->delete();
            $user->voicemails()->delete();
            $user->phoneNumbers()->delete();
            $user->transactions()->delete();
            
            // Anonymize user record
            $user->update([
                'email' => 'deleted_' . $user->id . '@deleted.local',
                'phone' => null,
                'status' => 'deleted',
            ]);
            
            // Log deletion for compliance
            AuditLog::create([
                'action' => 'user_data_deleted',
                'user_id' => $user->id,
                'performed_at' => now(),
            ]);
        });
    }
    
    public function autoDeleteOldData(): void
    {
        // Delete call records older than 2 years
        Call::where('created_at', '<', now()->subYears(2))->delete();
        
        // Delete SMS/MMS older than 1 year
        SmsMessage::where('created_at', '<', now()->subYear())->delete();
        MmsMessage::where('created_at', '<', now()->subYear())->delete();
    }
}
```

---

### 4. Infrastructure Security

#### 4.1 Server Hardening

**Checklist:**
- [ ] Disable root login
- [ ] Use SSH keys (disable password auth)
- [ ] Configure firewall (UFW/iptables)
- [ ] Install fail2ban
- [ ] Enable automatic security updates
- [ ] Disable unnecessary services
- [ ] Configure SELinux/AppArmor
- [ ] Regular security patches

**Implementation:**
```bash
# Disable root login
sudo sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config

# Disable password authentication
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config

# Configure UFW
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 5060/udp  # SIP
sudo ufw allow 5061/tcp  # SIP TLS
sudo ufw allow 10000:20000/udp  # RTP
sudo ufw enable

# Install fail2ban
sudo apt install fail2ban
sudo systemctl enable fail2ban
```

#### 4.2 Database Security

**Checklist:**
- [ ] Use strong passwords
- [ ] Limit network access
- [ ] Enable SSL/TLS
- [ ] Regular backups
- [ ] Encrypted backups
- [ ] Principle of least privilege
- [ ] Audit logging

**Implementation:**
```sql
-- Create limited user
CREATE USER 'voip_app'@'localhost' IDENTIFIED BY 'strong_password';
GRANT SELECT, INSERT, UPDATE, DELETE ON voip_platform.* TO 'voip_app'@'localhost';
FLUSH PRIVILEGES;

-- Enable audit logging
INSTALL PLUGIN server_audit SONAME 'server_audit.so';
SET GLOBAL server_audit_logging=ON;
SET GLOBAL server_audit_events='CONNECT,QUERY,TABLE';
```

#### 4.3 Monitoring & Logging

**Requirements:**
- Centralized logging
- Real-time monitoring
- Intrusion detection
- Anomaly detection
- Alert system

**Implementation:**
```php
// Laravel logging
use Illuminate\Support\Facades\Log;

// Security event logging
Log::channel('security')->info('Login attempt', [
    'user_id' => $user->id,
    'ip' => $request->ip(),
    'user_agent' => $request->userAgent(),
    'success' => true,
]);

// Failed login
Log::channel('security')->warning('Failed login attempt', [
    'email' => $request->email,
    'ip' => $request->ip(),
    'timestamp' => now(),
]);

// Suspicious activity
Log::channel('security')->alert('Suspicious activity detected', [
    'user_id' => $user->id,
    'activity' => 'multiple_failed_logins',
    'count' => 5,
]);
```

**ELK Stack Setup:**
```yaml
# docker-compose.yml
version: '3.8'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
    ports:
      - "9200:9200"
  
  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5000:5000"
  
  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    ports:
      - "5601:5601"
```

---

### 5. Compliance & Privacy

#### 5.1 GDPR Compliance

**Requirements:**
- Lawful basis for processing
- Data protection by design
- Privacy by default
- Data breach notification
- Data protection impact assessment (DPIA)
- Privacy policy
- Cookie consent

**Implementation:**
```php
class GDPRCompliance
{
    public function exportUserData(User $user): array
    {
        // Right to data portability
        return [
            'user' => $user->toArray(),
            'phone_numbers' => $user->phoneNumbers->toArray(),
            'calls' => $user->calls->toArray(),
            'messages' => $user->smsMessages->toArray(),
            'transactions' => $user->transactions->toArray(),
        ];
    }
    
    public function handleDataBreach(): void
    {
        // Notify within 72 hours
        $affectedUsers = User::whereIn('id', $affectedUserIds)->get();
        
        foreach ($affectedUsers as $user) {
            Mail::to($user->email)->send(new DataBreachNotification($user));
        }
        
        // Notify supervisory authority
        $this->notifySupervisoryAuthority();
    }
}
```

#### 5.2 PCI DSS Compliance (if handling cards)

**Requirements:**
- Use payment gateway (Stripe, PayPal)
- Never store CVV
- Tokenize card data
- Secure network
- Regular security testing

**Implementation:**
```php
// Use Stripe for payment processing
use Stripe\Stripe;
use Stripe\PaymentIntent;

Stripe::setApiKey(config('stripe.secret'));

$paymentIntent = PaymentIntent::create([
    'amount' => 5000, // $50.00
    'currency' => 'usd',
    'customer' => $user->stripe_customer_id,
    'payment_method' => $request->payment_method_id,
    'confirm' => true,
]);

// Never store full card numbers
// Only store last 4 digits and token
$user->update([
    'card_last_four' => $paymentIntent->payment_method->card->last4,
    'card_brand' => $paymentIntent->payment_method->card->brand,
    'stripe_payment_method_id' => $paymentIntent->payment_method->id,
]);
```

---

## Security Testing

### 1. Static Application Security Testing (SAST)

**Tools:**
- PHPStan
- Psalm
- SonarQube
- Snyk

**Implementation:**
```bash
# Install PHPStan
composer require --dev phpstan/phpstan

# Run analysis
./vendor/bin/phpstan analyse app

# Install Snyk
npm install -g snyk

# Test for vulnerabilities
snyk test
```

### 2. Dynamic Application Security Testing (DAST)

**Tools:**
- OWASP ZAP
- Burp Suite
- Nikto

**Implementation:**
```bash
# Run OWASP ZAP
docker run -t owasp/zap2docker-stable zap-baseline.py \
    -t https://api.voipplatform.com
```

### 3. Penetration Testing

**Scope:**
- Web application
- Mobile applications
- API endpoints
- VoIP infrastructure
- Network infrastructure

**Timeline:**
- Pre-launch: Comprehensive VAPT
- Quarterly: Focused testing
- Post-major-release: Regression testing

---

## Security Incident Response Plan

### 1. Incident Classification

| Severity | Description | Response Time |
|----------|-------------|---------------|
| **Critical** | Data breach, system compromise | Immediate |
| **High** | Unauthorized access, DoS | < 1 hour |
| **Medium** | Suspicious activity, failed attacks | < 4 hours |
| **Low** | Policy violations, minor issues | < 24 hours |

### 2. Response Procedure

```
1. Detection & Analysis
   ├── Monitor alerts
   ├── Verify incident
   └── Classify severity

2. Containment
   ├── Isolate affected systems
   ├── Block malicious IPs
   └── Revoke compromised credentials

3. Eradication
   ├── Remove malware
   ├── Patch vulnerabilities
   └── Close attack vectors

4. Recovery
   ├── Restore from backups
   ├── Verify system integrity
   └── Resume normal operations

5. Post-Incident
   ├── Document incident
   ├── Lessons learned
   └── Update security controls
```

---

## Security Checklist

### Pre-Implementation Review

- [ ] Authentication & authorization design reviewed
- [ ] Input validation strategy defined
- [ ] API security controls documented
- [ ] TLS/SSL configuration planned
- [ ] VoIP security measures defined
- [ ] Encryption at rest strategy defined
- [ ] Encryption in transit strategy defined
- [ ] Data retention policy created
- [ ] Server hardening checklist completed
- [ ] Database security measures planned
- [ ] Monitoring & logging strategy defined
- [ ] GDPR compliance measures documented
- [ ] Incident response plan created
- [ ] Security testing plan created
- [ ] Third-party security review scheduled

### Implementation Phase

- [ ] Security controls implemented
- [ ] Code review completed
- [ ] SAST tools integrated
- [ ] Security testing performed
- [ ] Penetration testing scheduled
- [ ] Security documentation updated

### Pre-Launch

- [ ] Comprehensive VAPT completed
- [ ] All critical/high vulnerabilities fixed
- [ ] Security sign-off obtained
- [ ] Incident response team trained
- [ ] Monitoring alerts configured
- [ ] Backup & recovery tested

---

*Document Version: 1.0*  
*Created: February 2026*  
*Status: Planning Document*
