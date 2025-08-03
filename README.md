# Astro Project Vision Document

## 1. Problem Statement

### Current Pain Points
- **No Centralized Item Management**: Communities lack a unified system for managing virtual items across different distribution scenarios
- **Manual Distribution Overhead**: All item distribution (prizes, rewards, payments) requires manual Steam friend management and trading
- **Fragmented Solutions**: Each distribution method (giveaways, VIP rewards, tournament prizes) needs custom implementation
- **Limited Community Participation**: No systematic way for community members to contribute items for distribution
- **No Audit Trail**: No centralized tracking of item flows, assignments, or distribution history
- **Poor Integration**: Existing systems can't easily integrate with Steam for automated item operations

### The Core Need
Communities need a **centralized item management platform** that can:
- Accept and manage item donations from the community
- Provide clean APIs for various systems to assign items to users
- Automate all Steam-related operations (trades, inventory, friends)
- Track complete item lifecycle with audit trails
- Support multiple distribution scenarios through a unified interface

### Why a Dedicated Platform?
Rather than building Steam integration into each individual system (giveaway plugins, VIP systems, etc.), a dedicated platform allows:
- **Reusable Infrastructure**: One system handles all Steam complexity
- **Consistent Experience**: Same donation and delivery process regardless of distribution method
- **Better Reliability**: Specialized focus on item management and Steam operations
- **Community Benefits**: Single place for users to donate items and claim rewards

## 2. Vision & Goals

### Vision Statement
Astro is a centralized item management platform that provides Steam integration and automated distribution services to gaming communities. It serves as the foundation that various community systems can leverage for any item-related operations.

### Primary Goals
- **Unified Item Management**: Single platform for all community item operations
- **Steam Integration Hub**: Centralized Steam bot services for multiple client systems
- **Automated Operations**: Eliminate manual intervention in item distribution workflows
- **Community Enablement**: Provide infrastructure for community-driven item contributions
- **Extensible Platform**: Support any distribution scenario through clean APIs and plugin architecture
- **Reliable Operations**: Maintain perfect item tracking and audit trails

### Key Principles
- **Platform-First Design**: Build reusable infrastructure, not point solutions
- **API-Driven**: All functionality exposed through clean, versioned APIs
- **Event-Driven Architecture**: Loose coupling through event-based communication
- **Separation of Concerns**: Client systems handle their logic, Astro handles Steam operations
- **Community-Centric**: Easy participation for both contributors and recipients

## 3. System Overview

### Core Components
1. **Steam Bot**: Manages Steam inventory, trades, and friend relationships
2. **API Gateway**: Provides clean REST APIs for client systems to integrate
3. **Assignment Engine**: Handles item assignment logic and validation
4. **Database Layer**: Tracks items, assignments, users, and transaction history
5. **Event System**: Publishes real-time events for client system integration
6. **Admin Dashboard**: Web interface for platform administration and monitoring

### Client System Examples
- **Giveaway Plugins**: SourceMod plugins that need to distribute prizes
- **VIP Systems**: Platforms that reward premium members with items
- **Tournament Systems**: Competition platforms that need to award prizes
- **Achievement Systems**: Game progression systems with item rewards
- **Admin Tools**: Manual distribution interfaces for special circumstances
- **Community Bots**: Discord bots or other community management tools

### Key Integrations
- **Existing Giveaway Plugin**: Via event forwards/hooks
- **Steam API**: For inventory management and trading
- **Community Systems**: Future integration with forums, Discord, etc.

### System Boundaries
**In Scope:**
- Item donation and storage
- Assignment of items to users via multiple methods
- Automated delivery to winners/recipients
- Transaction history and audit trails
- Admin tools for manual operations

**Out of Scope (Initially):**
- Game-side giveaway mechanics (handled by existing plugin)
- User authentication beyond Steam ID
- Complex approval workflows
- Multi-game support (focus on single game first)

## 4. Use Cases & Scenarios

### Primary Use Cases

#### UC1: Community Item Donation
**Actor**: Community Member<br>
**Flow**:
1. User sends trade offer to bot with donation keyword
2. Bot detects donation intent and accepts trade
3. System records items in database marked for distribution
4. Items become available for assignment by admins

#### UC2: Automated Giveaway Distribution
**Actor**: Admin, Giveaway Plugin<br>
**Flow**:
1. Admin starts giveaway via existing SourceMod plugin
2. Plugin queries Astro API for available items
3. Admin selects items from available inventory
4. Giveaway runs and selects winner
5. Plugin notifies Astro of winner and prizes
6. Bot automatically friends winner and initiates trade
7. Items are delivered and removed from inventory

#### UC3: Manual Prize Assignment
**Actor**: Admin<br>
**Flow**:
1. Admin accesses Astro admin interface
2. Selects recipient and items to assign
3. System records assignment in database
4. Bot handles delivery process automatically
5. Transaction logged for audit purposes

#### UC4: VIP Reward Distribution
**Actor**: VIP System, Admin<br>
**Flow**:
1. VIP system determines user earned reward
2. System queries available items or uses designated VIP items
3. Items automatically assigned to user
4. Standard delivery process initiated
5. VIP transaction recorded separately

### Edge Cases & Error Scenarios
- **Trade Failures**: Handle Steam trade holds, user privacy settings, full inventories
- **Inventory Desync**: Detect and reconcile differences between bot inventory and database
- **Unclaimed Items**: Handle items assigned to users who never claim them
- **Duplicate Donations**: Prevent or handle same item donated multiple times
- **Bot Downtime**: Queue operations and process when bot returns online

## 5. Key Requirements

### Functional Requirements

#### Core Item Management
- **FR1**: Accept item donations via Steam trades with specific keywords
- **FR2**: Maintain 1:1 synchronization between bot inventory and database
- **FR3**: Support assignment of items to users via multiple methods
- **FR4**: Automatically deliver assigned items to recipients
- **FR5**: Track complete history of all item transactions

#### Integration Requirements
- **FR6**: Integrate with existing SourceMod giveaway plugin via event hooks
- **FR7**: Provide REST API for external systems to assign items
- **FR8**: Support plugin architecture for new assignment methods
- **FR9**: Handle Steam friend management automatically

#### Administrative Features
- **FR10**: Web interface for manual item assignments
- **FR11**: Administrative oversight of all transactions
- **FR12**: Ability to recall/reassign unclaimed items
- **FR13**: Reporting and analytics on distribution patterns

### Non-Functional Requirements

#### Performance
- **NFR1**: Handle concurrent trade operations without conflicts
- **NFR2**: Respond to giveaway queries within 2 seconds
- **NFR3**: Support up to 1000 items in bot inventory efficiently

#### Reliability
- **NFR4**: 99.9% uptime for core item tracking
- **NFR5**: Automatic recovery from Steam API outages
- **NFR6**: Data consistency even during system failures
- **NFR7**: Complete audit trail for all operations

#### Security
- **NFR8**: Secure API access with authentication
- **NFR9**: Protection against item duplication or loss
- **NFR10**: Validation of all trade operations
- **NFR11**: Rate limiting on API endpoints

#### Usability
- **NFR12**: Simple donation process for community members
- **NFR13**: Intuitive admin interface for manual operations
- **NFR14**: Clear feedback on trade status for users

### Technical Constraints
- Must integrate with existing SourceMod infrastructure
- Limited to Steam platform capabilities and API restrictions
- Database must handle high-frequency inventory updates
- Real-time event processing between components

## 6. System Architecture

### High-Level Architecture
```
[SourceMod Plugin] --> [Astro API] --> [Steam Bot]
       |                    |              |
       |                    |              |
    [Game Server]      [Database]    [Steam API]
```

### Data Flow Patterns

#### Event-Driven Communication
- SourceMod plugin emits events (giveaway start, end, cancel)
- Astro consumes events and updates internal state
- Steam bot reacts to state changes and performs operations
- All components publish status events for monitoring

#### API-First Design
- All external interactions go through REST API
- Internal components communicate via events
- Clear separation between public API and internal operations
- Versioned APIs for backward compatibility

### Module Architecture

#### Assignment Method Plugins
Each distribution method implemented as separate module:
- **Giveaway Module**: Integrates with SourceMod plugin
- **Manual Assignment Module**: Admin interface for direct assignments
- **VIP Module**: Integrates with VIP systems
- **Achievement Module**: Future integration with game achievements
- **Purchase Module**: Future marketplace/point redemption system

#### Common Plugin Interface
```
AssignmentMethod {
  - validateAssignment(user, items)
  - executeAssignment(assignment)
  - handleAssignmentFailure(assignment, error)
  - getAssignmentHistory(filters)
}
```

## 7. Implementation Phases

### Phase 1: Core Platform (MVP)
**Goals**: Build foundational platform with first client integration
**Scope**:
- Steam bot with inventory management
- Core API for item assignment and querying
- Database design and item tracking
- Basic assignment and delivery workflow
- Integration SDK/documentation for client systems
- First client: existing giveaway plugin integration

**Success Criteria**: Platform can handle basic item assignments from any client system

### Phase 2: Platform Enhancement
**Goals**: Add community features and improve platform capabilities
**Scope**:
- Community donation system
- Enhanced admin dashboard
- Multiple client system support
- Advanced error handling and recovery
- Real-time event streaming
- Comprehensive API documentation

**Success Criteria**: Multiple different client systems can integrate and operate simultaneously

### Phase 3: Advanced Features
**Goals**: Support complex distribution scenarios
**Scope**:
- VIP integration module
- Advanced admin tools and analytics
- Multiple assignment method support
- Enhanced security and validation
- Performance optimizations

**Success Criteria**: System supports multiple distribution methods seamlessly

### Phase 4: Platform Features
**Goals**: Make system broadly useful for communities
**Scope**:
- Plugin marketplace for custom assignment methods
- Advanced reporting and analytics
- Integration with external systems (Discord, forums)
- Multi-community support
- Advanced administrative controls

**Success Criteria**: Other communities can easily adopt and extend the system

## 8. Success Metrics

### Technical Metrics
- **Inventory Sync Accuracy**: 100% synchronization between bot and database
- **Distribution Success Rate**: >95% of assignments successfully delivered
- **API Response Time**: <2 seconds for all giveaway operations
- **System Uptime**: >99.9% availability

### User Experience Metrics
- **Admin Time Savings**: Reduce manual distribution time to zero
- **Community Participation**: Track donation volume and frequency
- **Error Rates**: <1% failed trades or assignments
- **User Satisfaction**: Surveys of admin and community experience

### Business Metrics
- **Giveaway Frequency**: Increase in giveaway activity due to easier management
- **Community Engagement**: Increased participation in server activities
- **Operational Efficiency**: Reduction in admin workload
- **System Adoption**: Usage by other communities (Phase 4)

## 9. Risk Assessment

### Technical Risks
- **Steam API Changes**: Mitigation through abstraction layers and monitoring
- **Inventory Desynchronization**: Automated reconciliation and manual recovery procedures
- **Trade Failures**: Comprehensive error handling and retry mechanisms
- **Database Performance**: Proper indexing and query optimization

### Operational Risks
- **Bot Account Restrictions**: Backup accounts and compliance with Steam ToS
- **Admin Misuse**: Audit trails and approval workflows for sensitive operations
- **Community Abuse**: Rate limiting and validation of donation patterns
- **Data Loss**: Regular backups and disaster recovery procedures

### Project Risks
- **Scope Creep**: Clear phase definitions and feature prioritization
- **Over-Engineering**: Focus on MVP first, iterate based on usage
- **Integration Complexity**: Thorough testing of SourceMod integration
- **Maintenance Burden**: Documentation and modular design for sustainability

## 10. Future Possibilities

### Near-term Enhancements
- **Discord Integration**: Notifications and commands via Discord bot
- **Web Dashboard**: Enhanced admin interface with analytics
- **Mobile Support**: Basic mobile-friendly interfaces
- **API Webhooks**: Real-time notifications to external systems

### Long-term Vision
- **Multi-Game Support**: Extend beyond single game to platform
- **Marketplace Features**: Item exchange and trading systems
- **AI-Powered Distribution**: Smart assignment based on user preferences
- **Community Marketplace**: User-to-user trading through bot
- **Economic Analytics**: Advanced insights into item flow and value

### Platform Evolution
- **SaaS Offering**: Hosted version for communities without technical expertise
- **Plugin Ecosystem**: Third-party developers creating assignment methods
- **Integration Platform**: Connect with various gaming and community platforms
- **White-Label Solutions**: Customized versions for large gaming organizations

---

*This document represents the complete vision for the Astro system. Implementation will follow the phased approach outlined above, starting with Phase 1 to address the immediate pain points while building the foundation for future capabilities.*
