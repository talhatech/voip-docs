# Device Data Sync Architecture

## Overview
Architecture for device-local data storage and encrypted backup/transfer between devices.

## Data Storage Architecture

```mermaid
graph TB
    subgraph MobileDevice["Mobile Device"]
        subgraph AppLayer["Application Layer"]
            UI[User Interface]
            BL[Business Logic]
        end
        
        subgraph DataLayer["Data Layer"]
            SQLite[(Encrypted SQLite<br/>Database)]
            FileSystem[File System<br/>Media Files]
            SecureStorage[Secure Storage<br/>Keychain/Keystore]
        end
        
        subgraph EncryptionLayer["Encryption Layer"]
            AES[AES-256 Encryption]
            KeyManager[Key Manager]
        end
    end
    
    subgraph CloudBackup["Temporary Cloud Backup"]
        S3[S3/MinIO Storage]
        EncryptedBackup[Encrypted Backup File]
    end
    
    subgraph LaravelBackend["Laravel Backend"]
        TransferAPI[Transfer API]
        TokenManager[Token Manager]
    end
    
    UI --> BL
    BL --> SQLite
    BL --> FileSystem
    BL --> SecureStorage
    
    SQLite --> AES
    FileSystem --> AES
    AES --> KeyManager
    KeyManager --> SecureStorage
    
    BL -.->|Backup Request| TransferAPI
    TransferAPI -.->|Upload| EncryptedBackup
    EncryptedBackup --> S3
    
    TransferAPI -.->|Generate Token| TokenManager

    classDef appStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef dataStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef encStyle fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef cloudStyle fill:#f3e5f5,stroke:#4a148c,stroke-width:2px

    class UI,BL appStyle
    class SQLite,FileSystem,SecureStorage dataStyle
    class AES,KeyManager,EncryptionLayer encStyle
    class S3,EncryptedBackup,TransferAPI,TokenManager cloudStyle
```

## Device Transfer Flow

```mermaid
sequenceDiagram
    participant OldDevice as Old Device
    participant User as User
    participant Laravel as Laravel API
    participant S3 as S3 Storage
    participant NewDevice as New Device

    Note over OldDevice,NewDevice: Device Transfer Process

    User->>OldDevice: 1. Initiate Transfer
    
    activate OldDevice
    Note over OldDevice: 2. Create Local Backup<br/>- Export SQLite DB<br/>- Collect media files<br/>- Export settings
    
    OldDevice->>OldDevice: 3. Encrypt Backup<br/>AES-256 with User Key
    
    OldDevice->>Laravel: 4. POST /device/transfer/initiate<br/>{backup_size, device_info}
    deactivate OldDevice
    
    activate Laravel
    Laravel->>Laravel: 5. Generate Transfer Token<br/>Valid for 24 hours
    
    Laravel->>S3: 6. Generate Presigned URL<br/>for Upload
    
    Laravel-->>OldDevice: 7. Transfer Token + Upload URL
    deactivate Laravel
    
    activate OldDevice
    OldDevice->>S3: 8. Upload Encrypted Backup<br/>via Presigned URL
    
    S3-->>OldDevice: 9. Upload Complete
    
    OldDevice->>OldDevice: 10. Generate QR Code<br/>{token, encryption_key}
    
    OldDevice->>User: 11. Display QR Code
    deactivate OldDevice
    
    User->>NewDevice: 12. Scan QR Code
    
    activate NewDevice
    NewDevice->>NewDevice: 13. Extract Token<br/>and Encryption Key
    
    NewDevice->>Laravel: 14. POST /device/transfer/complete<br/>{token}
    deactivate NewDevice
    
    activate Laravel
    Laravel->>Laravel: 15. Validate Token<br/>Check Expiry
    
    Laravel->>S3: 16. Generate Download URL
    
    Laravel-->>NewDevice: 17. Download URL
    deactivate Laravel
    
    activate NewDevice
    NewDevice->>S3: 18. Download Encrypted<br/>Backup
    
    S3-->>NewDevice: 19. Encrypted Backup File
    
    NewDevice->>NewDevice: 20. Decrypt Backup<br/>Using Encryption Key
    
    NewDevice->>NewDevice: 21. Import Data<br/>- Restore SQLite DB<br/>- Download media<br/>- Apply settings
    
    NewDevice->>Laravel: 22. POST /device/transfer/confirm<br/>{token, success}
    deactivate NewDevice
    
    activate Laravel
    Laravel->>S3: 23. Delete Backup File
    
    Laravel->>Laravel: 24. Invalidate Token
    
    Laravel-->>NewDevice: 25. Transfer Complete
    
    Laravel->>OldDevice: 26. Push Notification:<br/>Transfer Successful
    deactivate Laravel
    
    Note over NewDevice: Device Ready to Use
```

## Local Data Structure

```mermaid
graph TB
    subgraph LocalDatabase["Local SQLite Database"]
        CallsTable[Calls Table<br/>Call History]
        SMSTable[SMS Table<br/>Message History]
        MMSTable[MMS Table<br/>MMS History]
        VoicemailTable[Voicemail Table<br/>Voicemail Records]
        ContactsTable[Contacts Table<br/>App Contacts]
        SettingsTable[Settings Table<br/>User Preferences]
    end
    
    subgraph FileStorage["File System Storage"]
        VoicemailAudio[Voicemail Audio<br/>Files]
        MMSMedia[MMS Media<br/>Images/Videos]
        CallRecordings[Call Recordings<br/>Audio Files]
        ProfilePics[Profile Pictures]
    end
    
    subgraph SecureKeychain["Secure Keychain/Keystore"]
        EncryptionKeys[Encryption Keys]
        AuthTokens[Auth Tokens]
        SIPCredentials[SIP Credentials]
        PINHash[PIN Hash]
    end
    
    subgraph SyncStatus["Sync Metadata"]
        LastSync[Last Sync Timestamp]
        SyncQueue[Pending Sync Items]
        ConflictLog[Conflict Resolution Log]
    end
    
    CallsTable -.->|References| VoicemailAudio
    MMSTable -.->|References| MMSMedia
    CallsTable -.->|References| CallRecordings
    ContactsTable -.->|References| ProfilePics

    classDef dbStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef fileStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    classDef secureStyle fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    classDef syncStyle fill:#fff9c4,stroke:#f57f17,stroke-width:2px

    class CallsTable,SMSTable,MMSTable,VoicemailTable,ContactsTable,SettingsTable dbStyle
    class VoicemailAudio,MMSMedia,CallRecordings,ProfilePics fileStyle
    class EncryptionKeys,AuthTokens,SIPCredentials,PINHash secureStyle
    class LastSync,SyncQueue,ConflictLog syncStyle
```

## Backup Structure

```mermaid
graph LR
    Backup[Encrypted Backup File] --> Metadata[Metadata<br/>Version, Timestamp,<br/>Device Info]
    
    Backup --> DatabaseDump[SQLite Database<br/>Dump]
    
    Backup --> MediaManifest[Media Manifest<br/>File List + Checksums]
    
    Backup --> MediaFiles[Compressed Media<br/>Files Archive]
    
    Backup --> Settings[Settings Export<br/>JSON]
    
    Backup --> Checksum[Backup Checksum<br/>SHA-256]

    classDef backupStyle fill:#f3e5f5,stroke:#4a148c,stroke-width:2px

    class Backup,Metadata,DatabaseDump,MediaManifest,MediaFiles,Settings,Checksum backupStyle
```

## Data Sync Strategy

```mermaid
stateDiagram-v2
    [*] --> LocalOnly: App Installed
    
    LocalOnly --> BackupRequested: User Initiates Transfer
    
    BackupRequested --> CreatingBackup: Generate Backup
    
    CreatingBackup --> Uploading: Backup Created
    
    Uploading --> BackupStored: Upload Complete
    
    BackupStored --> QRDisplayed: Generate QR Code
    
    QRDisplayed --> Transferring: New Device Scans
    
    Transferring --> Downloading: Token Validated
    
    Downloading --> Decrypting: Download Complete
    
    Decrypting --> Importing: Decryption Success
    
    Importing --> LocalOnly: Import Complete
    
    BackupStored --> Expired: 24 Hours Elapsed
    
    Expired --> [*]: Backup Deleted
    
    Importing --> [*]: Transfer Complete

    note right of LocalOnly
        All data stored locally
        No server-side storage
        Encrypted at rest
    end note

    note right of BackupStored
        Temporary storage only
        Encrypted with user key
        Auto-deleted after 24h
    end note

    note right of Importing
        Decrypt and import
        Verify checksums
        Update local DB
    end note
```

## Security Measures

### Encryption
```mermaid
graph TB
    UserPassword[User Password] --> PBKDF2[PBKDF2 Key<br/>Derivation]
    
    PBKDF2 --> MasterKey[Master Encryption<br/>Key]
    
    MasterKey --> DBEncryption[Database<br/>Encryption]
    MasterKey --> FileEncryption[File<br/>Encryption]
    MasterKey --> BackupEncryption[Backup<br/>Encryption]
    
    DBEncryption --> EncryptedDB[(Encrypted SQLite)]
    FileEncryption --> EncryptedFiles[Encrypted Media]
    BackupEncryption --> EncryptedBackup[Encrypted Backup]
    
    MasterKey --> Keychain[Secure Keychain/<br/>Keystore]

    classDef keyStyle fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    classDef encStyle fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px

    class UserPassword,PBKDF2,MasterKey,Keychain keyStyle
    class DBEncryption,FileEncryption,BackupEncryption,EncryptedDB,EncryptedFiles,EncryptedBackup encStyle
```

## Key Features

### Local Storage
1. **Encrypted SQLite**: All structured data encrypted at rest
2. **File Encryption**: Media files encrypted individually
3. **Secure Keychain**: Credentials stored in platform secure storage
4. **No Cloud Sync**: Data stays on device unless explicitly transferred

### Device Transfer
1. **QR Code Pairing**: Simple, secure device-to-device transfer
2. **Time-Limited**: Transfer tokens expire after 24 hours
3. **One-Time Use**: Tokens invalidated after successful transfer
4. **Automatic Cleanup**: Server backups deleted immediately after transfer

### Data Integrity
1. **Checksums**: SHA-256 checksums for all files
2. **Verification**: Data integrity verified during import
3. **Atomic Operations**: All-or-nothing import process
4. **Rollback**: Failed imports don't corrupt existing data

### Privacy
1. **User-Controlled**: User initiates all transfers
2. **End-to-End Encrypted**: Server cannot decrypt data
3. **Temporary Storage**: Server storage is temporary only
4. **Audit Logging**: All transfers logged for security
