# Step 4: Create Detailed API Specifications - OpenAPI/Swagger

## Overview

This document outlines the process of creating comprehensive API specifications for the VoIP platform using OpenAPI 3.0 (Swagger). Well-documented APIs are crucial for frontend development, third-party integrations, and maintaining consistency across the platform.

## Why OpenAPI/Swagger?

**Benefits:**
- ✅ Industry-standard API documentation format
- ✅ Interactive API documentation (Swagger UI)
- ✅ Auto-generate client SDKs
- ✅ API testing and validation
- ✅ Contract-first development
- ✅ Versioning support
- ✅ Mock server generation

---

## Tools & Setup

### 1. Install Required Packages

```bash
cd /home/talha/projects/voIP/backend

# Install L5-Swagger (Laravel OpenAPI generator)
composer require darkaonline/l5-swagger

# Publish configuration
php artisan vendor:publish --provider="L5Swagger\L5SwaggerServiceProvider"

# Install Swagger UI assets
php artisan l5-swagger:generate
```

### 2. Configure L5-Swagger

Edit `config/l5-swagger.php`:

```php
<?php

return [
    'default' => 'default',
    'documentations' => [
        'default' => [
            'api' => [
                'title' => 'VoIP Platform API Documentation',
            ],
            'routes' => [
                'api' => 'api/documentation',
            ],
            'paths' => [
                'use_absolute_path' => env('L5_SWAGGER_USE_ABSOLUTE_PATH', true),
                'docs_json' => 'api-docs.json',
                'docs_yaml' => 'api-docs.yaml',
                'format_to_use_for_docs' => env('L5_FORMAT_TO_USE_FOR_DOCS', 'json'),
                'annotations' => [
                    base_path('app/Http/Controllers'),
                    base_path('app/Models'),
                ],
            ],
        ],
    ],
    'defaults' => [
        'routes' => [
            'docs' => 'docs',
            'oauth2_callback' => 'api/oauth2-callback',
            'middleware' => [
                'api' => [],
                'asset' => [],
                'docs' => [],
                'oauth2_callback' => [],
            ],
            'group_options' => [],
        ],
        'paths' => [
            'docs' => storage_path('api-docs'),
            'views' => base_path('resources/views/vendor/l5-swagger'),
            'base' => env('L5_SWAGGER_BASE_PATH', null),
            'swagger_ui_assets_path' => env('L5_SWAGGER_UI_ASSETS_PATH', 'vendor/swagger-api/swagger-ui/dist/'),
            'excludes' => [],
        ],
        'scanOptions' => [
            'analyser' => null,
            'analysis' => null,
            'processors' => [],
            'pattern' => null,
            'exclude' => [],
        ],
        'securityDefinitions' => [
            'securitySchemes' => [
                'sanctum' => [
                    'type' => 'http',
                    'scheme' => 'bearer',
                    'bearerFormat' => 'JWT',
                ],
            ],
            'security' => [
                ['sanctum' => []],
            ],
        ],
        'generate_always' => env('L5_SWAGGER_GENERATE_ALWAYS', false),
        'generate_yaml_copy' => env('L5_SWAGGER_GENERATE_YAML_COPY', false),
        'proxy' => false,
        'additional_config_url' => null,
        'operations_sort' => env('L5_SWAGGER_OPERATIONS_SORT', null),
        'validator_url' => null,
        'ui' => [
            'display' => [
                'dark_mode' => env('L5_SWAGGER_UI_DARK_MODE', false),
                'doc_expansion' => env('L5_SWAGGER_UI_DOC_EXPANSION', 'none'),
                'filter' => env('L5_SWAGGER_UI_FILTERS', true),
            ],
            'authorization' => [
                'persist_authorization' => env('L5_SWAGGER_UI_PERSIST_AUTHORIZATION', false),
            ],
        ],
        'constants' => [
            'L5_SWAGGER_CONST_HOST' => env('L5_SWAGGER_CONST_HOST', 'http://localhost:8000'),
        ],
    ],
];
```

---

## API Structure

### Base OpenAPI Configuration

Create `app/Http/Controllers/Controller.php` with base annotations:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Foundation\Auth\Access\AuthorizesRequests;
use Illuminate\Foundation\Validation\ValidatesRequests;
use Illuminate\Routing\Controller as BaseController;

/**
 * @OA\Info(
 *     version="1.0.0",
 *     title="VoIP Platform API",
 *     description="Comprehensive API documentation for the VoIP communication platform",
 *     @OA\Contact(
 *         email="support@voipplatform.com"
 *     ),
 *     @OA\License(
 *         name="Proprietary",
 *         url="https://voipplatform.com/license"
 *     )
 * )
 *
 * @OA\Server(
 *     url=L5_SWAGGER_CONST_HOST,
 *     description="Development Server"
 * )
 *
 * @OA\Server(
 *     url="https://api.voipplatform.com",
 *     description="Production Server"
 * )
 *
 * @OA\SecurityScheme(
 *     securityScheme="sanctum",
 *     type="http",
 *     scheme="bearer",
 *     bearerFormat="JWT",
 *     description="Enter your bearer token in the format: Bearer {token}"
 * )
 *
 * @OA\Tag(
 *     name="Authentication",
 *     description="User authentication and authorization endpoints"
 * )
 *
 * @OA\Tag(
 *     name="Users",
 *     description="User management endpoints"
 * )
 *
 * @OA\Tag(
 *     name="Phone Numbers",
 *     description="Phone number management and provisioning"
 * )
 *
 * @OA\Tag(
 *     name="Calls",
 *     description="Voice call management and history"
 * )
 *
 * @OA\Tag(
 *     name="SMS",
 *     description="SMS messaging endpoints"
 * )
 *
 * @OA\Tag(
 *     name="MMS",
 *     description="MMS messaging endpoints"
 * )
 *
 * @OA\Tag(
 *     name="Voicemail",
 *     description="Voicemail management"
 * )
 *
 * @OA\Tag(
 *     name="Call Rules",
 *     description="Call forwarding and routing rules"
 * )
 *
 * @OA\Tag(
 *     name="Billing",
 *     description="Wallet, transactions, and billing"
 * )
 *
 * @OA\Tag(
 *     name="Number Porting",
 *     description="Number porting requests and management"
 * )
 *
 * @OA\Tag(
 *     name="Webhooks",
 *     description="Webhook endpoints for external integrations"
 * )
 */
class Controller extends BaseController
{
    use AuthorizesRequests, ValidatesRequests;
}
```

---

## Example API Documentation

### 1. Authentication Endpoints

```php
<?php

namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class AuthController extends Controller
{
    /**
     * @OA\Post(
     *     path="/api/v1/auth/register",
     *     tags={"Authentication"},
     *     summary="Register a new user",
     *     description="Create a new user account",
     *     operationId="register",
     *     @OA\RequestBody(
     *         required=true,
     *         description="User registration data",
     *         @OA\JsonContent(
     *             required={"email","password","password_confirmation"},
     *             @OA\Property(property="email", type="string", format="email", example="user@example.com"),
     *             @OA\Property(property="password", type="string", format="password", example="SecurePass123!"),
     *             @OA\Property(property="password_confirmation", type="string", format="password", example="SecurePass123!"),
     *             @OA\Property(property="phone", type="string", example="+1234567890"),
     *         )
     *     ),
     *     @OA\Response(
     *         response=201,
     *         description="User registered successfully",
     *         @OA\JsonContent(
     *             @OA\Property(property="success", type="boolean", example=true),
     *             @OA\Property(property="message", type="string", example="User registered successfully"),
     *             @OA\Property(
     *                 property="data",
     *                 type="object",
     *                 @OA\Property(property="user", ref="#/components/schemas/User"),
     *                 @OA\Property(property="token", type="string", example="1|abcdefghijklmnopqrstuvwxyz")
     *             )
     *         )
     *     ),
     *     @OA\Response(
     *         response=422,
     *         description="Validation error",
     *         @OA\JsonContent(
     *             @OA\Property(property="success", type="boolean", example=false),
     *             @OA\Property(property="message", type="string", example="Validation failed"),
     *             @OA\Property(
     *                 property="errors",
     *                 type="object",
     *                 @OA\Property(
     *                     property="email",
     *                     type="array",
     *                     @OA\Items(type="string", example="The email has already been taken.")
     *                 )
     *             )
     *         )
     *     )
     * )
     */
    public function register(Request $request)
    {
        // Implementation
    }

    /**
     * @OA\Post(
     *     path="/api/v1/auth/login",
     *     tags={"Authentication"},
     *     summary="Login user",
     *     description="Authenticate user and return access token",
     *     operationId="login",
     *     @OA\RequestBody(
     *         required=true,
     *         @OA\JsonContent(
     *             required={"email","password"},
     *             @OA\Property(property="email", type="string", format="email", example="user@example.com"),
     *             @OA\Property(property="password", type="string", format="password", example="SecurePass123!"),
     *         )
     *     ),
     *     @OA\Response(
     *         response=200,
     *         description="Login successful",
     *         @OA\JsonContent(
     *             @OA\Property(property="success", type="boolean", example=true),
     *             @OA\Property(property="message", type="string", example="Login successful"),
     *             @OA\Property(
     *                 property="data",
     *                 type="object",
     *                 @OA\Property(property="user", ref="#/components/schemas/User"),
     *                 @OA\Property(property="token", type="string", example="1|abcdefghijklmnopqrstuvwxyz")
     *             )
     *         )
     *     ),
     *     @OA\Response(
     *         response=401,
     *         description="Invalid credentials",
     *         @OA\JsonContent(
     *             @OA\Property(property="success", type="boolean", example=false),
     *             @OA\Property(property="message", type="string", example="Invalid credentials")
     *         )
     *     )
     * )
     */
    public function login(Request $request)
    {
        // Implementation
    }

    /**
     * @OA\Post(
     *     path="/api/v1/auth/logout",
     *     tags={"Authentication"},
     *     summary="Logout user",
     *     description="Revoke user's access token",
     *     operationId="logout",
     *     security={{"sanctum":{}}},
     *     @OA\Response(
     *         response=200,
     *         description="Logout successful",
     *         @OA\JsonContent(
     *             @OA\Property(property="success", type="boolean", example=true),
     *             @OA\Property(property="message", type="string", example="Logout successful")
     *         )
     *     ),
     *     @OA\Response(
     *         response=401,
     *         description="Unauthenticated",
     *         @OA\JsonContent(
     *             @OA\Property(property="message", type="string", example="Unauthenticated")
     *         )
     *     )
     * )
     */
    public function logout(Request $request)
    {
        // Implementation
    }
}
```

### 2. Phone Number Endpoints

```php
<?php

namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;

class PhoneNumberController extends Controller
{
    /**
     * @OA\Get(
     *     path="/api/v1/numbers",
     *     tags={"Phone Numbers"},
     *     summary="List user's phone numbers",
     *     description="Get all phone numbers owned by the authenticated user",
     *     operationId="listPhoneNumbers",
     *     security={{"sanctum":{}}},
     *     @OA\Parameter(
     *         name="status",
     *         in="query",
     *         description="Filter by status",
     *         required=false,
     *         @OA\Schema(type="string", enum={"active", "inactive", "porting", "released"})
     *     ),
     *     @OA\Response(
     *         response=200,
     *         description="Phone numbers retrieved successfully",
     *         @OA\JsonContent(
     *             @OA\Property(property="success", type="boolean", example=true),
     *             @OA\Property(
     *                 property="data",
     *                 type="array",
     *                 @OA\Items(ref="#/components/schemas/PhoneNumber")
     *             ),
     *             @OA\Property(
     *                 property="meta",
     *                 type="object",
     *                 @OA\Property(property="total", type="integer", example=5),
     *                 @OA\Property(property="active", type="integer", example=3)
     *             )
     *         )
     *     )
     * )
     */
    public function index()
    {
        // Implementation
    }

    /**
     * @OA\Get(
     *     path="/api/v1/numbers/search",
     *     tags={"Phone Numbers"},
     *     summary="Search available phone numbers",
     *     description="Search for available phone numbers to purchase",
     *     operationId="searchPhoneNumbers",
     *     security={{"sanctum":{}}},
     *     @OA\Parameter(
     *         name="country_code",
     *         in="query",
     *         description="Country code (ISO 3166-1 alpha-2)",
     *         required=true,
     *         @OA\Schema(type="string", example="US")
     *     ),
     *     @OA\Parameter(
     *         name="area_code",
     *         in="query",
     *         description="Area code",
     *         required=false,
     *         @OA\Schema(type="string", example="212")
     *     ),
     *     @OA\Parameter(
     *         name="contains",
     *         in="query",
     *         description="Number must contain these digits",
     *         required=false,
     *         @OA\Schema(type="string", example="555")
     *     ),
     *     @OA\Parameter(
     *         name="type",
     *         in="query",
     *         description="Number type",
     *         required=false,
     *         @OA\Schema(type="string", enum={"local", "toll_free", "mobile"})
     *     ),
     *     @OA\Parameter(
     *         name="limit",
     *         in="query",
     *         description="Maximum results to return",
     *         required=false,
     *         @OA\Schema(type="integer", example=10, minimum=1, maximum=100)
     *     ),
     *     @OA\Response(
     *         response=200,
     *         description="Available numbers found",
     *         @OA\JsonContent(
     *             @OA\Property(property="success", type="boolean", example=true),
     *             @OA\Property(
     *                 property="data",
     *                 type="array",
     *                 @OA\Items(ref="#/components/schemas/AvailablePhoneNumber")
     *             )
     *         )
     *     )
     * )
     */
    public function search()
    {
        // Implementation
    }

    /**
     * @OA\Post(
     *     path="/api/v1/numbers/purchase",
     *     tags={"Phone Numbers"},
     *     summary="Purchase a phone number",
     *     description="Purchase an available phone number",
     *     operationId="purchasePhoneNumber",
     *     security={{"sanctum":{}}},
     *     @OA\RequestBody(
     *         required=true,
     *         @OA\JsonContent(
     *             required={"number"},
     *             @OA\Property(property="number", type="string", example="+12125551234"),
     *             @OA\Property(property="is_primary", type="boolean", example=false)
     *         )
     *     ),
     *     @OA\Response(
     *         response=201,
     *         description="Number purchased successfully",
     *         @OA\JsonContent(
     *             @OA\Property(property="success", type="boolean", example=true),
     *             @OA\Property(property="message", type="string", example="Number purchased successfully"),
     *             @OA\Property(property="data", ref="#/components/schemas/PhoneNumber")
     *         )
     *     ),
     *     @OA\Response(
     *         response=402,
     *         description="Insufficient balance",
     *         @OA\JsonContent(
     *             @OA\Property(property="success", type="boolean", example=false),
     *             @OA\Property(property="message", type="string", example="Insufficient wallet balance")
     *         )
     *     ),
     *     @OA\Response(
     *         response=409,
     *         description="Number no longer available",
     *         @OA\JsonContent(
     *             @OA\Property(property="success", type="boolean", example=false),
     *             @OA\Property(property="message", type="string", example="Number is no longer available")
     *         )
     *     )
     * )
     */
    public function purchase()
    {
        // Implementation
    }
}
```

### 3. SMS Endpoints

```php
<?php

namespace App\Http\Controllers\Api\V1;

use App\Http\Controllers\Controller;

class SmsController extends Controller
{
    /**
     * @OA\Post(
     *     path="/api/v1/sms/send",
     *     tags={"SMS"},
     *     summary="Send SMS message",
     *     description="Send an SMS message",
     *     operationId="sendSms",
     *     security={{"sanctum":{}}},
     *     @OA\RequestBody(
     *         required=true,
     *         @OA\JsonContent(
     *             required={"from","to","body"},
     *             @OA\Property(property="from", type="string", example="+12125551234", description="Sender phone number (must be owned by user)"),
     *             @OA\Property(property="to", type="string", example="+19175555678", description="Recipient phone number"),
     *             @OA\Property(property="body", type="string", example="Hello, this is a test message", description="Message content (max 1600 characters)")
     *         )
     *     ),
     *     @OA\Response(
     *         response=201,
     *         description="SMS sent successfully",
     *         @OA\JsonContent(
     *             @OA\Property(property="success", type="boolean", example=true),
     *             @OA\Property(property="message", type="string", example="SMS sent successfully"),
     *             @OA\Property(property="data", ref="#/components/schemas/SmsMessage")
     *         )
     *     ),
     *     @OA\Response(
     *         response=402,
     *         description="Insufficient balance",
     *         @OA\JsonContent(
     *             @OA\Property(property="success", type="boolean", example=false),
     *             @OA\Property(property="message", type="string", example="Insufficient wallet balance")
     *         )
     *     ),
     *     @OA\Response(
     *         response=403,
     *         description="Sender number not owned by user",
     *         @OA\JsonContent(
     *             @OA\Property(property="success", type="boolean", example=false),
     *             @OA\Property(property="message", type="string", example="You do not own this phone number")
     *         )
     *     )
     * )
     */
    public function send()
    {
        // Implementation
    }

    /**
     * @OA\Post(
     *     path="/api/v1/sms/schedule",
     *     tags={"SMS"},
     *     summary="Schedule SMS message",
     *     description="Schedule an SMS message for future delivery",
     *     operationId="scheduleSms",
     *     security={{"sanctum":{}}},
     *     @OA\RequestBody(
     *         required=true,
     *         @OA\JsonContent(
     *             required={"from","to","body","scheduled_at"},
     *             @OA\Property(property="from", type="string", example="+12125551234"),
     *             @OA\Property(property="to", type="string", example="+19175555678"),
     *             @OA\Property(property="body", type="string", example="Scheduled message"),
     *             @OA\Property(property="scheduled_at", type="string", format="date-time", example="2026-02-10T10:00:00Z")
     *         )
     *     ),
     *     @OA\Response(
     *         response=201,
     *         description="SMS scheduled successfully",
     *         @OA\JsonContent(
     *             @OA\Property(property="success", type="boolean", example=true),
     *             @OA\Property(property="message", type="string", example="SMS scheduled successfully"),
     *             @OA\Property(property="data", ref="#/components/schemas/SmsMessage")
     *         )
     *     )
     * )
     */
    public function schedule()
    {
        // Implementation
    }
}
```

---

## Schema Definitions

### User Schema

```php
<?php

namespace App\Models;

/**
 * @OA\Schema(
 *     schema="User",
 *     type="object",
 *     title="User",
 *     description="User model",
 *     required={"id", "email"},
 *     @OA\Property(property="id", type="integer", format="int64", example=1),
 *     @OA\Property(property="uuid", type="string", format="uuid", example="550e8400-e29b-41d4-a716-446655440000"),
 *     @OA\Property(property="email", type="string", format="email", example="user@example.com"),
 *     @OA\Property(property="phone", type="string", example="+1234567890"),
 *     @OA\Property(property="status", type="string", enum={"active", "suspended", "deleted"}, example="active"),
 *     @OA\Property(property="created_at", type="string", format="date-time", example="2026-02-09T00:00:00Z"),
 *     @OA\Property(property="updated_at", type="string", format="date-time", example="2026-02-09T00:00:00Z")
 * )
 */
class User extends Authenticatable
{
    // Model implementation
}
```

### PhoneNumber Schema

```php
<?php

namespace App\Models;

/**
 * @OA\Schema(
 *     schema="PhoneNumber",
 *     type="object",
 *     title="Phone Number",
 *     description="Phone number model",
 *     @OA\Property(property="id", type="integer", example=1),
 *     @OA\Property(property="user_id", type="integer", example=1),
 *     @OA\Property(property="number", type="string", example="+12125551234"),
 *     @OA\Property(property="country_code", type="string", example="US"),
 *     @OA\Property(property="type", type="string", enum={"local", "toll_free", "mobile"}, example="local"),
 *     @OA\Property(property="status", type="string", enum={"active", "inactive", "porting", "released"}, example="active"),
 *     @OA\Property(property="is_primary", type="boolean", example=false),
 *     @OA\Property(
 *         property="capabilities",
 *         type="object",
 *         @OA\Property(property="voice", type="boolean", example=true),
 *         @OA\Property(property="sms", type="boolean", example=true),
 *         @OA\Property(property="mms", type="boolean", example=true)
 *     ),
 *     @OA\Property(property="monthly_cost", type="number", format="float", example=1.50),
 *     @OA\Property(property="purchased_at", type="string", format="date-time"),
 *     @OA\Property(property="expires_at", type="string", format="date-time"),
 *     @OA\Property(property="created_at", type="string", format="date-time"),
 *     @OA\Property(property="updated_at", type="string", format="date-time")
 * )
 */
class PhoneNumber extends Model
{
    // Model implementation
}
```

### Call Schema

```php
<?php

namespace App\Models;

/**
 * @OA\Schema(
 *     schema="Call",
 *     type="object",
 *     title="Call",
 *     description="Call record model",
 *     @OA\Property(property="id", type="integer", example=1),
 *     @OA\Property(property="uuid", type="string", format="uuid"),
 *     @OA\Property(property="user_id", type="integer", example=1),
 *     @OA\Property(property="phone_number_id", type="integer", example=1),
 *     @OA\Property(property="direction", type="string", enum={"inbound", "outbound"}, example="outbound"),
 *     @OA\Property(property="from_number", type="string", example="+12125551234"),
 *     @OA\Property(property="to_number", type="string", example="+19175555678"),
 *     @OA\Property(property="status", type="string", enum={"initiated", "ringing", "answered", "completed", "failed", "busy", "no_answer"}, example="completed"),
 *     @OA\Property(property="duration_seconds", type="integer", example=120),
 *     @OA\Property(property="cost", type="number", format="float", example=0.05),
 *     @OA\Property(property="recording_url", type="string", format="url", example="https://recordings.example.com/call123.mp3"),
 *     @OA\Property(property="started_at", type="string", format="date-time"),
 *     @OA\Property(property="answered_at", type="string", format="date-time"),
 *     @OA\Property(property="ended_at", type="string", format="date-time"),
 *     @OA\Property(property="created_at", type="string", format="date-time")
 * )
 */
class Call extends Model
{
    // Model implementation
}
```

### SmsMessage Schema

```php
<?php

namespace App\Models;

/**
 * @OA\Schema(
 *     schema="SmsMessage",
 *     type="object",
 *     title="SMS Message",
 *     description="SMS message model",
 *     @OA\Property(property="id", type="integer", example=1),
 *     @OA\Property(property="uuid", type="string", format="uuid"),
 *     @OA\Property(property="user_id", type="integer", example=1),
 *     @OA\Property(property="phone_number_id", type="integer", example=1),
 *     @OA\Property(property="direction", type="string", enum={"inbound", "outbound"}, example="outbound"),
 *     @OA\Property(property="from_number", type="string", example="+12125551234"),
 *     @OA\Property(property="to_number", type="string", example="+19175555678"),
 *     @OA\Property(property="body", type="string", example="Hello, this is a test message"),
 *     @OA\Property(property="segments", type="integer", example=1),
 *     @OA\Property(property="status", type="string", enum={"queued", "sent", "delivered", "failed", "received"}, example="delivered"),
 *     @OA\Property(property="cost", type="number", format="float", example=0.0079),
 *     @OA\Property(property="sent_at", type="string", format="date-time"),
 *     @OA\Property(property="delivered_at", type="string", format="date-time"),
 *     @OA\Property(property="scheduled_at", type="string", format="date-time"),
 *     @OA\Property(property="scheduled_status", type="string", enum={"pending", "sent", "cancelled"}),
 *     @OA\Property(property="created_at", type="string", format="date-time")
 * )
 */
class SmsMessage extends Model
{
    // Model implementation
}
```

---

## Generate API Documentation

```bash
# Generate Swagger documentation
php artisan l5-swagger:generate

# Access documentation
# Navigate to: http://localhost:8000/api/documentation
```

---

## API Documentation Best Practices

### 1. Consistent Response Format

```php
/**
 * Standard success response format
 */
{
    "success": true,
    "message": "Operation successful",
    "data": { /* response data */ },
    "meta": { /* pagination, counts, etc */ }
}

/**
 * Standard error response format
 */
{
    "success": false,
    "message": "Error message",
    "errors": { /* validation errors or details */ }
}
```

### 2. HTTP Status Codes

| Code | Meaning | Usage |
|------|---------|-------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid request format |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 409 | Conflict | Resource conflict |
| 422 | Unprocessable Entity | Validation failed |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |

### 3. Versioning

```php
// Route versioning
Route::prefix('v1')->group(function () {
    // V1 routes
});

Route::prefix('v2')->group(function () {
    // V2 routes
});
```

### 4. Pagination

```php
/**
 * @OA\Parameter(
 *     name="page",
 *     in="query",
 *     description="Page number",
 *     required=false,
 *     @OA\Schema(type="integer", minimum=1, example=1)
 * )
 * @OA\Parameter(
 *     name="per_page",
 *     in="query",
 *     description="Items per page",
 *     required=false,
 *     @OA\Schema(type="integer", minimum=1, maximum=100, example=15)
 * )
 */
```

### 5. Filtering & Sorting

```php
/**
 * @OA\Parameter(
 *     name="filter[status]",
 *     in="query",
 *     description="Filter by status",
 *     required=false,
 *     @OA\Schema(type="string")
 * )
 * @OA\Parameter(
 *     name="sort",
 *     in="query",
 *     description="Sort by field (prefix with - for descending)",
 *     required=false,
 *     @OA\Schema(type="string", example="-created_at")
 * )
 */
```

---

## Testing API Documentation

### 1. Validate OpenAPI Spec

```bash
# Install Swagger CLI
npm install -g @apidevtools/swagger-cli

# Validate spec
swagger-cli validate storage/api-docs/api-docs.json
```

### 2. Generate Postman Collection

```bash
# Install openapi-to-postman
npm install -g openapi-to-postmanv2

# Convert OpenAPI to Postman
openapi2postmanv2 -s storage/api-docs/api-docs.json -o postman-collection.json
```

### 3. Generate Client SDKs

```bash
# Install OpenAPI Generator
npm install @openapitools/openapi-generator-cli -g

# Generate PHP SDK
openapi-generator-cli generate \
  -i storage/api-docs/api-docs.json \
  -g php \
  -o ./sdk/php

# Generate JavaScript SDK
openapi-generator-cli generate \
  -i storage/api-docs/api-docs.json \
  -g javascript \
  -o ./sdk/javascript
```

---

## Checklist

- [ ] Install L5-Swagger package
- [ ] Configure base OpenAPI annotations
- [ ] Document all authentication endpoints
- [ ] Document all user endpoints
- [ ] Document all phone number endpoints
- [ ] Document all call endpoints
- [ ] Document all SMS/MMS endpoints
- [ ] Document all voicemail endpoints
- [ ] Document all call rules endpoints
- [ ] Document all billing endpoints
- [ ] Document all porting endpoints
- [ ] Document all webhook endpoints
- [ ] Define all schema models
- [ ] Add request/response examples
- [ ] Add error response examples
- [ ] Test all endpoints in Swagger UI
- [ ] Validate OpenAPI specification
- [ ] Generate Postman collection
- [ ] Generate client SDKs (if needed)
- [ ] Review with frontend team
- [ ] Publish documentation

---

*Document Version: 1.0*  
*Created: February 2026*  
*Status: Planning Document*
