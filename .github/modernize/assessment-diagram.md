# Photo Album Application - Architecture Diagram

## Current Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        Browser[Web Browser]
    end
    
    subgraph "Application Layer"
        SpringBoot[Spring Boot 2.7.18<br/>Java 8]
        
        subgraph "Controllers"
            HomeCtrl[HomeController]
            DetailCtrl[DetailController]
            FileCtrl[PhotoFileController]
        end
        
        subgraph "Service Layer"
            PhotoSvc[PhotoService]
        end
        
        subgraph "Data Access"
            PhotoRepo[PhotoRepository<br/>Spring Data JPA]
        end
        
        subgraph "View Layer"
            Thymeleaf[Thymeleaf Templates]
        end
    end
    
    subgraph "Data Layer"
        OracleDB[(Oracle Database 21c XE<br/>BLOB Storage)]
    end
    
    Browser -->|HTTP Requests| SpringBoot
    SpringBoot --> Thymeleaf
    Thymeleaf -->|HTML Response| Browser
    
    HomeCtrl --> PhotoSvc
    DetailCtrl --> PhotoSvc
    FileCtrl --> PhotoSvc
    
    PhotoSvc --> PhotoRepo
    PhotoRepo -->|JDBC| OracleDB
    
    style SpringBoot fill:#6db33f
    style OracleDB fill:#f80000
    style Browser fill:#4285f4
```

## Technology Stack

```mermaid
graph LR
    subgraph "Backend"
        Java[Java 8]
        SB[Spring Boot 2.7.18]
        JPA[Spring Data JPA]
        Hibernate[Hibernate ORM]
        Thyme[Thymeleaf]
    end
    
    subgraph "Database"
        Oracle[Oracle Database 21c XE]
    end
    
    subgraph "Frontend"
        Bootstrap[Bootstrap 5.3.0]
        JS[Vanilla JavaScript]
        CSS[Custom CSS]
    end
    
    subgraph "Build & Deploy"
        Maven[Maven]
        Docker[Docker]
        DockerCompose[Docker Compose]
    end
    
    Java --> SB
    SB --> JPA
    JPA --> Hibernate
    SB --> Thyme
    
    Hibernate --> Oracle
    
    style Java fill:#007396
    style Oracle fill:#f80000
    style Maven fill:#c71a36
    style Docker fill:#2496ed
```

## Data Flow - Photo Upload

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant Controller
    participant Service
    participant Repository
    participant Database
    
    User->>Browser: Select and upload photo
    Browser->>Controller: POST multipart file
    Controller->>Service: processUpload(file)
    Service->>Service: Validate file type and size
    Service->>Service: Generate UUID
    Service->>Service: Read image metadata
    Service->>Repository: save(Photo entity with BLOB)
    Repository->>Database: INSERT with BLOB data
    Database-->>Repository: Success
    Repository-->>Service: Photo entity
    Service-->>Controller: UploadResult
    Controller-->>Browser: JSON response
    Browser-->>User: Display success message
```

## Key Components

### Application Components
- **Controllers**: Handle HTTP requests for home, detail, and file upload endpoints
- **Service Layer**: Business logic for photo processing and validation
- **Repository**: Spring Data JPA interface for database operations
- **Model**: Photo entity with metadata and BLOB data

### Database Schema
- **PHOTOS Table**: Stores photo metadata and binary data
  - ID (UUID primary key)
  - File metadata (name, size, MIME type)
  - Image dimensions (width, height)
  - PHOTO_DATA (BLOB - stores actual image bytes)
  - Upload timestamp

### Storage Architecture
- Photos stored as BLOBs directly in Oracle Database
- No file system dependencies
- UUID-based unique identifiers for cache management

## Configuration Details

### Database Connection
- **Driver**: Oracle JDBC (ojdbc8)
- **URL**: jdbc:oracle:thin:@oracle-db:1521/FREEPDB1
- **Dialect**: Oracle Dialect
- **Schema Management**: Hibernate DDL auto-create

### File Upload Settings
- **Max File Size**: 10MB per file
- **Max Request Size**: 50MB total
- **Allowed Types**: JPEG, PNG, GIF, WebP
- **Max Files**: 10 files per upload

## Deployment Architecture

```mermaid
graph TB
    subgraph "Docker Environment"
        subgraph "Application Container"
            App[Photo Album App<br/>Port 8080<br/>Spring Boot]
        end
        
        subgraph "Database Container"
            DB[(Oracle 21c XE<br/>Port 1521<br/>Enterprise Manager 5500)]
        end
    end
    
    Client[Client Browser] -->|HTTP 8080| App
    App -->|JDBC 1521| DB
    Admin[Database Admin] -.->|HTTP 5500| DB
    
    style App fill:#6db33f
    style DB fill:#f80000
    style Client fill:#4285f4
```

## Assessment Summary

Based on the AppCAT assessment report:

- **Total Issues**: 7 issues identified
- **Total Incidents**: 26 code locations requiring attention
- **Story Points**: 84 (estimated effort for modernization)

### Issue Categories
- **Database Migration**: 6 incidents - Oracle to Azure-compatible database
- **Framework Upgrade**: 10 incidents - Spring Boot modernization
- **Java Version Upgrade**: 3 incidents - Java 8 to modern LTS version
- **Local Credentials**: 3 incidents - Hardcoded credentials to secure storage
- **Spring Migration**: 4 incidents - Spring framework best practices

### Severity Distribution
- **Mandatory**: 13 incidents (must fix)
- **Potential**: 13 incidents (recommended to fix)
- **Optional**: 0 incidents
- **Information**: 0 incidents

### Target Platforms Assessed
- Azure Kubernetes Service (AKS)
- Azure App Service
- Azure Container Apps

---

*Generated on: 2026-02-12*
*Assessment Tool: Java AppCAT CLI v1.0.0*
