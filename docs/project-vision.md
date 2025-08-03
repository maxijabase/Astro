# Astro Project Vision Document (Revised)

## 1. Problem Statement

### Current Pain Points
- **No Centralized Item Management**: Communities lack a unified system for managing virtual items across different distribution scenarios
- **Manual Distribution Overhead**: All item distribution (prizes, rewards, payments) requires manual Steam friend management and trading
- **Fragmented Solutions**: Each distribution method (giveaways, VIP purchases, tournament prizes) needs custom implementation
- **Limited Community Participation**: No systematic way for community members to contribute items for distribution
- **No Audit Trail**: No centralized tracking of item flows, assignments, or distribution history
- **Poor Integration**: Existing systems can't easily integrate with Steam for automated item operations

### The Core Need
Communities need a **centralized Steam bot platform** that can:
- Accept and manage item donations from the community
- Provide flexible item assignment capabilities for multiple use cases
- Automate all Steam-related operations (trades, inventory, friends)
- Track complete item lifecycle with audit trails
- Support extensible functionality through plugin architecture

### Why a Dedicated Platform?
Rather than building Steam integration into each individual system, a dedicated Steam bot platform with plugins allows:
- **Reusable Infrastructure**: One system handles all Steam complexity
- **Consistent Experience**: Same donation and delivery process regardless of distribution method
- **Better Reliability**: Specialized focus on item management and Steam operations
- **Extensible Design**: New functionality added through plugins without core changes

## 2. Vision & Goals

### Vision Statement
Astro is a Steam bot platform with a plugin architecture that provides automated item management and distribution services to gaming communities. It serves as a unified foundation for all community item-related operations through extensible plugin modules.

### Primary Goals
- **Unified Steam Bot Platform**: Single system for all community Steam item operations
- **Plugin-Based Extensibility**: Support any distribution scenario through modular plugins
- **Automated Operations**: Eliminate manual intervention in item distribution workflows
- **Community Enablement**: Provide infrastructure for community-driven item contributions
- **Flexible Item Management**: Shared item pool accessible by any plugin
- **Reliable Operations**: Maintain perfect item tracking and audit trails

### Key Principles
- **Plugin-First Design**: Core provides Steam infrastructure, plugins add business logic
- **Domain Separation**: Each plugin owns its specific business domain and data
- **Shared Resources**: All plugins access common item pool for maximum flexibility
- **Clean Boundaries**: Clear separation between generic core and domain-specific functionality
- **Community-Centric**: Easy participation for both contributors and recipients

## 3. System Overview

### Core Architecture
Astro is a **monolithic application** with **internal plugin architecture**:

```
┌─────────────────────────────────────────────────────────────┐
│                    ASTRO APPLICATION                        │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                ASTRO CORE                           │    │
│  │  - Steam Bot (SteamKit2)                           │    │
│  │  - Item Management Services                        │    │
│  │  - Assignment Engine                                │    │
│  │  - Database Layer                                   │    │
│  │  - Plugin Infrastructure                            │    │
│  └─────────────────────────────────────────────────────┘    │
│                           │                                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                INTERNAL PLUGINS                     │    │
│  │                                                     │    │
│  │  ┌─────────────────┐    ┌───────────────────────┐   │    │
│  │  │ Giveaway Plugin │    │     VIP Plugin        │   │    │
│  │  │ - HTTP API      │    │   - HTTP API          │   │    │
│  │  │ - Own Database  │    │   - Own Database      │   │    │
│  │  │ - Domain Logic  │    │   - Domain Logic      │   │    │
│  │  └─────────────────┘    └───────────────────────┘   │    │
│  │                                                     │    │
│  │  ┌─────────────────┐    ┌───────────────────────┐   │    │
│  │  │  Manual Plugin  │    │   Future Plugins...   │   │    │
│  │  │ - HTTP API      │    │                       │   │    │
│  │  │ - Own Database  │    │                       │   │    │
│  │  │ - Domain Logic  │    │                       │   │    │
│  │  └─────────────────┘    └───────────────────────┘   │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
                               │
                          Plugin APIs
                               │
                    ┌─────────────────┐
                    │ SourceMod Plugin│
                    │ (HTTP Client)   │
                    └─────────────────┘
```

### Core Components
1. **Steam Bot**: Manages Steam inventory, trades, and friend relationships using SteamKit2
2. **Item Management Service**: Tracks items in shared pool, handles assignments
3. **Assignment Engine**: Processes item assignments and delivery workflows
4. **Database Layer**: Stores core item and assignment data
5. **Plugin Infrastructure**: Loads, configures, and manages internal plugins
6. **Plugin System**: Modular components that add domain-specific functionality

### Plugin Examples
- **Giveaway Plugin**: Manages giveaway lifecycle, participant tracking, winner selection
- **VIP Plugin**: Handles VIP purchases through item payments, VIP status management
- **Manual Assignment Plugin**: Provides admin interface for direct item assignments
- **Future Plugins**: Tournament rewards, achievement systems, marketplace features

### External Integration
- **SourceMod Plugin**: Lightweight HTTP client that communicates with plugin APIs
- **Discord Bots**: Can integrate with plugin APIs for notifications and commands
- **Web Dashboards**: Admin interfaces provided by individual plugins

### System Boundaries
**In Scope:**
- Steam bot operations and inventory management
- Item donation and shared pool management
- Plugin architecture and lifecycle management
- Assignment of items to users via multiple methods
- Automated delivery to recipients
- Transaction history and audit trails
- Plugin-specific business logic and data

**Out of Scope (Initially):**
- Game-side mechanics (handled by external plugins like SourceMod)
- User authentication beyond Steam ID
- Complex approval workflows
- Multi-game support (focus on single game first)

## 4. Use Cases & Scenarios

### Primary Use Cases

#### UC1: Community Item Donation
**Actor**: Community Member  
**Flow**:
1. User sends trade offer to Steam bot with donation keyword
2. Bot detects donation intent and accepts trade
3. Core system adds items to shared pool with status 'available'
4. Items become available for assignment by any plugin

#### UC2: Automated Giveaway Distribution
**Actor**: Admin, SourceMod Plugin  
**Flow**:
1. Admin starts giveaway via SourceMod plugin
2. SourceMod plugin calls Giveaway Plugin API
3. Giveaway Plugin queries Core for available items
4. Admin selects items, giveaway runs and selects winner
5. Giveaway Plugin creates assignment via Core services
6. Core system queues delivery, Steam bot handles trade
7. Items delivered and marked as 'delivered' in shared pool

#### UC3: VIP Purchase System
**Actor**: Community Member  
**Flow**:
1. User sends trade offer to bot with VIP payment keyword
2. Steam bot accepts trade, items added to shared pool
3. VIP Plugin calculates VIP duration based on item values
4. VIP Plugin records purchase and updates user's VIP status
5. SourceMod VIP plugin queries VIP status via API
6. VIP automatically granted/extended on game server

#### UC4: Manual Prize Assignment
**Actor**: Admin  
**Flow**:
1. Admin accesses Manual Assignment Plugin interface
2. Selects recipient and items from shared pool
3. Plugin creates assignment via Core services
4. Core system handles delivery through Steam bot
5. Transaction logged for audit purposes

### Edge Cases & Error Scenarios
- **Trade Failures**: Handle Steam trade holds, user privacy settings, full inventories
- **Inventory Desync**: Detect and reconcile differences between bot inventory and database
- **Unclaimed Items**: Handle items assigned to users who never claim them
- **Plugin Failures**: Isolate plugin errors from core system operations
- **Bot Downtime**: Queue operations and process when bot returns online

## 5. Key Requirements

### Functional Requirements

#### Core Item Management
- **FR1**: Accept item donations via Steam trades with configurable keywords
- **FR2**: Maintain 1:1 synchronization between bot inventory and shared pool database
- **FR3**: Support assignment of items to users from shared pool
- **FR4**: Automatically deliver assigned items to recipients via Steam bot
- **FR5**: Track complete history of all item transactions

#### Plugin Architecture
- **FR6**: Load and configure plugins dynamically at startup
- **FR7**: Provide core services to plugins via dependency injection
- **FR8**: Allow plugins to register their own HTTP endpoints
- **FR9**: Support plugin-specific database connections and schemas
- **FR10**: Isolate plugin failures from core system stability

#### Integration Requirements
- **FR11**: Provide HTTP APIs for external system integration
- **FR12**: Support SourceMod plugin integration as primary client
- **FR13**: Handle Steam friend management automatically
- **FR14**: Process trade offers with configurable keyword detection

#### Administrative Features
- **FR15**: Plugin-provided interfaces for manual operations
- **FR16**: Administrative oversight of all transactions via core audit trail
- **FR17**: Ability to monitor and manage plugin status
- **FR18**: System health monitoring and diagnostics

### Non-Functional Requirements

#### Performance
- **NFR1**: Handle concurrent trade operations without conflicts
- **NFR2**: Respond to plugin API requests within 2 seconds
- **NFR3**: Support up to 1000 items in shared pool efficiently
- **NFR4**: Plugin isolation should not impact core performance

#### Reliability
- **NFR5**: 99.9% uptime for core Steam bot operations
- **NFR6**: Automatic recovery from Steam API outages
- **NFR7**: Data consistency even during system failures
- **NFR8**: Plugin failures should not crash core system

#### Security
- **NFR9**: Secure API access with authentication for external clients
- **NFR10**: Protection against item duplication or loss
- **NFR11**: Validation of all trade operations
- **NFR12**: Plugin sandboxing to prevent unauthorized core access

#### Usability
- **NFR13**: Simple donation process for community members
- **NFR14**: Intuitive plugin-provided interfaces for operations
- **NFR15**: Clear feedback on trade status for users
- **NFR16**: Easy plugin development and deployment process

### Technical Constraints
- Must integrate with Steam platform using SteamKit2
- Limited to Steam platform capabilities and API restrictions
- Database must handle high-frequency inventory updates
- Plugin system must maintain clear boundaries between domains

## 6. System Architecture

### High-Level Architecture
```
[SourceMod Plugin] ──HTTP──→ [Plugin APIs] ──Services──→ [Astro Core] ──SteamKit2──→ [Steam API]
                                   │                           │
                                   │                           │
                            [Plugin Databases]         [Core Database]
```

### Data Architecture

#### Core Database (Astro Only)
- **Items**: Shared pool of all items (available, assigned, delivered)
- **Users**: Steam user information and basic profiles
- **Assignments**: Generic item-to-user assignments with delivery status
- **System Events**: Audit trail of all item movements and operations

#### External System Data (Plugin Responsibility)
- **SourceMod Databases**: Game servers store their own giveaway, VIP, and gameplay data
- **Plugin Storage**: Each plugin/system chooses its own data storage approach
- **Clean Separation**: Business logic data stays with the systems that own it

### Plugin Architecture

#### Plugin Interface
```csharp
public interface IAstroPlugin
{
    string Name { get; }
    void ConfigureServices(IServiceCollection services);
    void ConfigureEndpoints(IEndpointRouteBuilder endpoints);
    Task InitializeAsync(IServiceProvider services);
}
```

#### Core Services Available to Plugins
```csharp
public interface IItemService
{
    Task<List<Item>> GetAvailableItemsAsync();
    Task<Item> GetItemByIdAsync(Guid id);
}

public interface IAssignmentService
{
    Task<Assignment> CreateAssignmentAsync(long userId, List<Guid> itemIds, string type, object context);
    Task<List<Assignment>> GetPendingAssignmentsAsync(long userId);
}

public interface ISteamBotService
{
    Task SyncInventoryAsync();
    Task<bool> IsOnlineAsync();
}
```

## 7. Implementation Phases

### Phase 1: Core Platform Foundation
**Goals**: Build Astro Core with basic Steam bot functionality
**Scope**:
- Steam bot with SteamKit2 integration
- Core database design and item management
- Basic assignment and delivery workflow
- Plugin infrastructure and loading system
- Shared pool management

**Success Criteria**: Steam bot can accept donations and manage shared item pool

### Phase 2: First Plugin Implementation
**Goals**: Build and integrate Giveaway Plugin as proof of concept
**Scope**:
- Giveaway Plugin with HTTP API
- Plugin database design and management
- SourceMod integration for giveaway workflow
- End-to-end giveaway execution
- Plugin-to-core service integration

**Success Criteria**: Complete giveaway workflow from start to item delivery

### Phase 3: Multi-Plugin System
**Goals**: Add VIP and Manual Assignment plugins
**Scope**:
- VIP Plugin with payment processing
- Manual Assignment Plugin with admin interface
- Plugin coexistence and shared pool usage
- Enhanced monitoring and logging
- Performance optimization

**Success Criteria**: Multiple plugins operating simultaneously with shared resources

### Phase 4: Production Readiness
**Goals**: Prepare system for production deployment
**Scope**:
- Enhanced security and authentication
- Advanced error handling and recovery
- Comprehensive monitoring and alerts
- Documentation and deployment guides
- Performance tuning and optimization

**Success Criteria**: System ready for production community deployment

### Phase 5: Advanced Features
**Goals**: Add sophisticated capabilities and new plugins
**Scope**:
- Advanced analytics and reporting
- New plugin types (Tournament, Achievement)
- External system integrations (Discord, forums)
- Plugin marketplace and discovery
- Advanced administrative tools

**Success Criteria**: Platform supports diverse community needs and use cases

## 8. Success Metrics

### Technical Metrics
- **Inventory Sync Accuracy**: 100% synchronization between bot and shared pool
- **Distribution Success Rate**: >95% of assignments successfully delivered
- **API Response Time**: <2 seconds for all plugin operations
- **System Uptime**: >99.9% availability
- **Plugin Load Time**: <5 seconds for plugin initialization

### User Experience Metrics
- **Admin Time Savings**: Reduce manual distribution time to zero
- **Community Participation**: Track donation volume and frequency
- **Error Rates**: <1% failed trades or assignments
- **Plugin Adoption**: Usage metrics for each plugin type
- **User Satisfaction**: Surveys of admin and community experience

### Business Metrics
- **Operational Efficiency**: Reduction in admin workload
- **Community Engagement**: Increased participation in server activities
- **System Adoption**: Usage by other communities
- **Plugin Development**: Third-party plugin creation
- **Cost Effectiveness**: Operational cost reduction vs. manual processes

## 9. Risk Assessment

### Technical Risks
- **Steam API Changes**: Mitigation through abstraction layers and SteamKit2 updates
- **Plugin Isolation Failures**: Robust error handling and plugin sandboxing
- **Database Performance**: Proper indexing, connection pooling, and query optimization
- **Inventory Desynchronization**: Automated reconciliation and manual recovery procedures

### Operational Risks
- **Bot Account Restrictions**: Backup accounts and compliance with Steam ToS
- **Plugin Quality**: Plugin review process and testing requirements
- **Data Loss**: Regular backups and disaster recovery procedures
- **Security Vulnerabilities**: Regular security audits and updates

### Project Risks
- **Scope Creep**: Clear phase definitions and plugin boundaries
- **Plugin Complexity**: Keep plugin interfaces simple and well-documented
- **Integration Challenges**: Thorough testing of plugin interactions
- **Maintenance Burden**: Modular design for independent plugin updates

## 10. Future Possibilities

### Near-term Enhancements
- **Enhanced Admin Dashboard**: Comprehensive management interface
- **Plugin Hot-Swapping**: Update plugins without system restart
- **Advanced Analytics**: Detailed reporting across all plugins
- **Mobile Support**: Mobile-friendly plugin interfaces

### Long-term Vision
- **Multi-Game Support**: Extend platform to support multiple Steam games
- **Plugin Marketplace**: Community-developed plugins with discovery system
- **AI-Powered Features**: Smart item valuation and distribution optimization
- **Federation Support**: Connect multiple Astro instances across communities

### Platform Evolution
- **Cloud Deployment**: Hosted service options for communities
- **Enterprise Features**: Advanced security and compliance capabilities
- **Integration Platform**: Connect with various gaming and community platforms
- **White-Label Solutions**: Customized deployments for large organizations