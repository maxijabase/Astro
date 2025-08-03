# ADR-001: Technology Stack Selection

## Status
**Accepted**

## Context
Astro requires a robust technology stack that can handle Steam API integration, real-time event processing, database operations, and provide clean REST APIs for external systems. The chosen stack must support:

- Steam bot operations with inventory management
- Real-time trading and friend management
- Database operations with complex relationships
- REST API endpoints for SourceMod and other client integrations
- Event-driven architecture for loose coupling
- Long-running background services

## Decision

### Core Platform: **.NET 9 (C#)**
- **Framework**: ASP.NET Core for web APIs and background services
- **Runtime**: .NET 9 (latest version, with future upgrades as needed)
- **Deployment**: Self-contained deployments for easy server setup

### Steam Integration: **SteamKit2**
- **Library**: SteamKit2 for comprehensive Steam protocol support
- **Features**: Handles authentication, trading, inventory, friends management
- **Rationale**: Most mature and feature-complete .NET Steam library

### Database: **PostgreSQL**
- **ORM**: Entity Framework Core for database operations
- **Migration**: EF Core migrations for schema versioning
- **Features**: JSONB support for flexible item metadata storage

### API Framework: **ASP.NET Core Web API**
- **Version**: .NET 9 with traditional MVC controllers
- **Authentication**: JWT tokens for API security
- **Documentation**: Swagger/OpenAPI for client integration
- **Validation**: FluentValidation for request validation

### Additional Components

#### Plugin System
- **Plugin Loading**: Dynamic assembly loading for plugin discovery
- **Plugin Infrastructure**: IAstroPlugin interface and registration system
- **Plugin Isolation**: Separate database contexts and service scopes per plugin

#### Background Services
- **Hosted Services**: .NET BackgroundService for Steam bot operations
- **Task Scheduling**: Hangfire for scheduled tasks (inventory sync, cleanup)
- **Message Queue**: Built-in Channel<T> for internal event processing

#### Configuration & Logging
- **Configuration**: appsettings.json + environment variables
- **Logging**: Serilog with structured logging
- **Monitoring**: Health checks for Steam connectivity and database

#### Development & Deployment
- **Containerization**: Docker for consistent deployments
- **Reverse Proxy**: NGINX for production API hosting
- **Database Migrations**: Automated via EF Core during startup

## Implementation Architecture

```
┌─────────────────┐    ┌──────────────────────────────────────┐    ┌─────────────────┐
│   SourceMod     │───▶│         ASTRO APPLICATION            │───▶│   PostgreSQL    │
│   Plugin        │    │                                      │    │   Databases     │
└─────────────────┘    │  ┌────────────────────────────────┐  │    └─────────────────┘
                       │  │        PLUGIN APIS             │  │
                       │  │  ┌──────────┐  ┌──────────┐    │  │
                       │  │  │Giveaway  │  │   VIP    │    │  │
                       │  │  │  API     │  │   API    │    │  │
                       │  │  └──────────┘  └──────────┘    │  │
                       │  └────────────────────────────────┘  │
                       │              │                       │
                       │  ┌────────────────────────────────┐  │
                       │  │        ASTRO CORE              │  │    ┌─────────────────┐
                       │  │  - Item Management Services    │  │───▶│   SteamKit2     │
                       │  │  - Assignment Engine           │  │    │   Steam Bot     │
                       │  │  - Plugin Infrastructure       │  │    └─────────────────┘
                       │  │  - Background Services         │  │
                       │  └────────────────────────────────┘  │
                       └──────────────────────────────────────┘
```

### Project Structure
```
Astro.sln
├── src/
│   ├── Astro.Api/              # Main application startup & plugin hosting
│   ├── Astro.Core/             # Core services (no HTTP endpoints)
│   ├── Astro.Infrastructure/   # Database & external services
│   ├── Astro.Steam/            # SteamKit2 integration
│   └── Astro.Shared/           # Common utilities & DTOs
├── plugins/
│   ├── Astro.Plugins.Giveaway/ # Giveaway plugin with HTTP API
│   ├── Astro.Plugins.VIP/      # VIP plugin with HTTP API
│   └── Astro.Plugins.Manual/   # Manual assignment plugin
├── tests/
│   ├── Astro.Api.Tests/
│   ├── Astro.Core.Tests/
│   ├── Astro.Steam.Tests/
│   └── Astro.Plugins.Tests/
└── docker/
    ├── Dockerfile
    └── docker-compose.yml
```

## Rationale

### Why C# / .NET 9?
- **Performance**: High-performance runtime suitable for real-time operations
- **Ecosystem**: Rich ecosystem with mature libraries for all requirements
- **Async/Await**: Native support for asynchronous operations (critical for Steam API)
- **Cross-Platform**: Runs on Linux for cost-effective server deployment
- **Latest Features**: .NET 9 provides the newest language features and performance improvements
- **Developer Experience**: Excellent tooling and debugging capabilities

### Why SteamKit2?
- **Maturity**: Most established .NET library for Steam protocol
- **Features**: Complete coverage of Steam features (trading, inventory, friends)
- **Community**: Active development and community support
- **Documentation**: Well-documented with examples
- **Protocol Support**: Handles Steam protocol updates automatically
- **Event-Driven**: Natural fit for our event-driven architecture

### Why PostgreSQL?
- **JSON Support**: Native JSONB for flexible item metadata storage
- **Performance**: Excellent performance for complex queries
- **ACID Compliance**: Ensures data consistency for critical operations
- **Scalability**: Can handle growth in item volumes and transactions
- **EF Core Support**: First-class Entity Framework Core integration
- **Open Source**: No licensing costs for deployment

### Why ASP.NET Core?
- **Performance**: One of the fastest web frameworks available
- **Built-in Features**: Authentication, validation, OpenAPI generation
- **Dependency Injection**: Native DI container for clean architecture
- **Background Services**: Built-in support for long-running services
- **Health Checks**: Native monitoring and health check capabilities
- **Controller-Based**: Traditional MVC controllers for structured plugin APIs
- **Plugin Integration**: Excellent support for modular application architecture

## Implementation Considerations

### Phase 1 Scope
- **Core Services**: Item management, assignment engine, Steam bot integration
- **Plugin Infrastructure**: Plugin loading, registration, and lifecycle management
- **First Plugin**: Giveaway plugin with HTTP API for SourceMod integration
- **Database**: Core schema design with plugin-specific database support
- **Steam Bot**: Essential trading and inventory management

### Performance Targets
- **Plugin API Response Time**: < 200ms for basic operations
- **Steam Operations**: < 5 seconds for trade operations
- **Database**: Support for 10,000+ items with sub-second queries
- **Concurrent Users**: Handle 100+ concurrent API requests
- **Plugin Loading**: < 5 seconds for plugin initialization

### Security Considerations
- **API Authentication**: JWT tokens with configurable expiration
- **Steam Credentials**: Secure storage of Steam account credentials
- **Database**: Connection string encryption and secure defaults
- **HTTPS**: Enforce HTTPS for all API communications

### Development Workflow
- **IDE**: Visual Studio 2022 or JetBrains Rider
- **Version Control**: Git with conventional commits
- **Testing**: xUnit for unit tests, integration tests for Steam operations
- **CI/CD**: GitHub Actions for automated testing and deployment

## Migration Strategy
Since this is a greenfield project, no migration is required. However, we'll design the system with future migration needs in mind:

- **Database**: Use EF Core migrations from the start
- **API Versioning**: Implement versioning strategy for backward compatibility
- **Configuration**: Environment-based configuration for easy deployment changes

## Success Metrics
- **Development Velocity**: Time to implement Phase 1 features
- **System Reliability**: Uptime and error rates in production
- **Performance**: Meeting response time targets under load
- **Maintainability**: Ease of adding new features and fixing bugs