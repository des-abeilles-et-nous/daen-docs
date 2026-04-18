# CLAUDE.md - daen-docs

Comprehensive documentation repository for the DAEN-SCOUT (Beefree) ecosystem architecture, data model, and development framework.

> **Start here**: [ECOSYSTEM_CONTEXT.md](ECOSYSTEM_CONTEXT.md) contains shared ecosystem documentation that all repositories reference. After reading that, explore detailed documentation in this repository for specific domains.

## Purpose

`daen-docs` is the **single source of truth** for:
- High-level architecture and system design
- Data model definitions and schemas
- Firebase configuration and management
- Development framework conventions
- Environment setup and deployment procedures
- Integration patterns and best practices

This repository documents the "why" and "how" of the ecosystem, serving both developers and architects.

## Repository Structure

```
daen-docs/
├── data model/                    # Business object definitions
│   ├── POI schema
│   ├── User model
│   ├── Subscription definitions
│   └── Feed and event structures
├── data pipelines/                # Data flow and ETL documentation
│   ├── POI creation pipeline
│   ├── Notification flow
│   └── Data synchronization patterns
├── dev framework/                 # Developer guidelines and standards
│   ├── React Native Coding Conventions
│   ├── Firebase best practices
│   ├── Testing strategies
│   └── Performance optimization guides
├── environments/                  # Environment and deployment documentation
│   ├── EnvironmentsManagement.md
│   ├── Firebase project configuration
│   ├── Deployment procedures
│   └── High-level design diagrams
├── LICENSE                        # Repository license
└── README.md                      # Overview
```

## Key Documentation Areas

### 1. Data Model (`data model/`)

Defines the structure and semantics of business objects:

- **POI (Point of Interest)**
  - Structure and required fields
  - Validation rules
  - Lifecycle and state transitions
  - Relationships to other entities

- **User**
  - Profile structure
  - Authentication attributes
  - Preferences and settings
  - Subscription relationships

- **Subscriptions**
  - Subscription types and triggers
  - Notification event definitions
  - Filter and trigger configuration

- **Feeds**
  - Feed event log structure
  - Event types and payloads
  - Feed triggers and actions

### 2. Data Pipelines (`data pipelines/`)

Describes data flow and transformation:

- **POI Creation Pipeline**
  - How POIs are created (mobile app vs. backend)
  - Firestore writes and triggers
  - Tile generation and clustering
  - Notification emission

- **Notification Flow**
  - How notifications are triggered
  - Subscription matching logic
  - Push notification delivery via Expo
  - Event logging and auditing

- **Data Synchronization**
  - Real-time sync between client and server
  - Firestore/RTDB data partitioning
  - Conflict resolution
  - Offline behavior

### 3. Development Framework (`dev framework/`)

Coding standards and best practices:

- **React Native Coding Conventions**
  - Component structure and patterns
  - State management practices
  - Naming conventions
  - File organization

- **Firebase Best Practices**
  - Security rules design
  - Index optimization
  - Data structure guidelines
  - Cost optimization

- **Testing Strategies**
  - Unit testing patterns
  - Integration testing with Firebase
  - End-to-end testing approaches
  - Emulator-based testing

- **Performance Optimization**
  - Mobile app performance
  - Cloud functions optimization
  - Database query optimization
  - Network efficiency

### 4. Environments (`environments/`)

Environment setup and deployment:

- **EnvironmentsManagement.md**
  - Four-tier environment system (dev, sandbox, staging, live)
  - Environment-specific configuration
  - Firebase project mapping
  - Secret and credential management

- **Firebase Configuration**
  - Project setup procedures
  - Security rules deployment
  - Index management
  - Backup and recovery

- **Deployment Procedures**
  - Mobile app release process
  - Cloud functions deployment
  - Rollback procedures
  - Canary deployments

- **Architecture Diagrams**
  - System high-level design (HLD)
  - Component interactions
  - Data flow visualization
  - Deployment topology

## When to Reference This Repository

### For Developers
- Understanding system architecture and data flow
- Finding coding conventions and best practices
- Learning about environment setup
- Understanding Firebase data structure
- Troubleshooting integration issues

### For Architects
- System design decisions and rationale
- Data model and relationship design
- Pipeline and workflow orchestration
- Security and compliance considerations
- Performance and scalability analysis

### For DevOps/Deployment
- Environment setup and configuration
- Deployment procedures and runbooks
- Backup and recovery procedures
- Monitoring and alerting setup

## Documentation Standards

### Writing Style

- **Clear and practical** - Focus on actionable information
- **Up-to-date** - Keep aligned with actual implementation
- **Linked** - Reference other docs and code locations
- **Illustrated** - Use diagrams for complex concepts
- **Example-driven** - Show code and configuration examples

### Recommended Links

When documenting:
1. Link to relevant code files using relative paths
2. Link to other documentation sections for context
3. Include Firebase console navigation steps
4. Provide CLI commands where applicable

### Diagrams

Use diagrams for:
- Architecture overviews
- Data flow pipelines
- Entity relationships
- Deployment topology
- Sequence diagrams for complex flows

## Maintaining This Repository

### When to Update Documentation

- ✅ After architectural changes
- ✅ When adding new features or systems
- ✅ When modifying data structures
- ✅ When updating environment setup
- ✅ When discovering better practices
- ❌ Don't document every line of code (that's what code comments are for)

### Update Triggers

Developers should update daen-docs when:
1. **daen-scout** - Major feature affecting mobile architecture
2. **daen-fb-workers** - New worker types or task queue changes
3. **fb-admin** - New administrative operations
4. **Environment changes** - New projects, new configuration schemes

### Keeping Docs Fresh

- Monthly review of critical sections
- Update references when code moves/changes
- Add clarifications based on developer questions
- Archive outdated information (don't delete — preserve history)

## Structure of This Documentation

**See the root CLAUDE.md** for:
- Ecosystem overview
- Development standards (naming, commits, branching)
- Firebase data structures
- Environment configuration

**See repo-specific CLAUDE.md** in each repository for:
- Repository architecture
- Development commands
- Critical integration points
- Code patterns specific to that repo

**See daen-docs** for:
- Detailed data model definitions
- Complete data pipeline documentation
- Comprehensive development framework guides
- Detailed environment and deployment procedures

## Contributing to Documentation

When contributing to daen-docs:

1. **Choose the right section** - Place content in appropriate directory
2. **Follow conventions** - Use Markdown, include examples, link to code
3. **Keep it practical** - Focus on "how" and "why", not just "what"
4. **Update index** - If adding new documents, update directory README
5. **Get review** - Have changes reviewed for accuracy before merging
6. **Announce updates** - Notify team of significant documentation changes

## Documentation Factorization Principle

**All ecosystem-level information lives ONLY in daen-docs, never duplicated in individual repos.**

### What Goes in daen-docs
- Ecosystem architecture and system design
- Environment configuration and setup procedures
- Firebase data structures and schemas
- Shared naming conventions and code standards
- Git workflow and commit message standards
- Development framework guidelines
- Integration patterns and shared libraries

### What Goes in Individual Repos (CLAUDE.md)
- Repository-specific architecture and directory structure
- Development commands and build processes specific to that repo
- Code patterns and conventions for that repo's technology stack
- Critical integration points with other repos
- Key files and components in that repo

### Reference Pattern
Individual repos reference daen-docs via:
- **Local path**: `../daen-docs/ECOSYSTEM_CONTEXT.md` (when repos are cloned together)
- **GitHub link**: `https://github.com/des-abeilles-et-nous/daen-docs/blob/main/ECOSYSTEM_CONTEXT.md` (for single-repo clones)

**Never duplicate ecosystem documentation** — maintain single source of truth in daen-docs.

## Key Files to Reference

- `environments/EnvironmentsManagement.md` - Complete environment setup guide
- `dev framework/React Native Coding Conventions.md` - Mobile development standards
- `data model/` - All business object definitions
- `data pipelines/` - Data flow and integration patterns
- `environments/beefree HLD.drawio.svg` - System architecture diagram

## One-Stop Reference

For questions about:
- **"What does this system do?"** → Root CLAUDE.md (architecture overview)
- **"How do I build/deploy it?"** → Repo-specific CLAUDE.md (dev commands)
- **"What is the data model?"** → daen-docs/data model/
- **"How does data flow?"** → daen-docs/data pipelines/
- **"What are the coding standards?"** → daen-docs/dev framework/
- **"How do I set up environments?"** → daen-docs/environments/

