# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.4.0] - 2025-12-15

### Added

- **QA Engineer Agent (On-Demand)**: New agent for black-box testing and documentation maintenance
  - Performs user journey simulation, smoke tests, and exploratory testing
  - Generates `qa_report.md` for architect consumption
  - Maintains project documentation (README, guides)
  - Strictly forbidden from modifying application codebase
- **README.md**: Comprehensive project documentation
  - Complete workflow guide for all agents
  - Prerequisites and installation instructions
  - Quick start guide and iterative improvement workflow
  - Known issues and troubleshooting section

### Changed

- **Browser Automation Migration**: Switched from Playwright MCP to chrome-devtools MCP as primary tool
  - Updated coder agent with 26 chrome-devtools MCP tools across 6 categories
  - Added sample prompts for common browser automation tasks
  - Simplified setup: no Docker required for standard usage
  - Playwright MCP remains available as alternative option
- **Port Standardization**: Updated default port from localhost:5173 to localhost:3000
- **Workflow Simplification**: Transitioned from migration-focused to new app creation workflow
  - Removed AI Studio migration agent references
  - Removed database migration agent references
  - Streamlined from 7-agent to 5-agent core workflow (plus on-demand QA)
- **Documentation Updates**:
  - AI_WORKFLOW.md: Added QA Engineer, updated workflow sequences
  - coder.md: Comprehensive chrome-devtools MCP tool reference

### Fixed

- Browser tool syntax: Verified correct `mcp__chrome-devtools__` prefix for all tools

## [0.3.1] - 2025-12-15

### Fixed

- Architect agent: Resolved contradictory typography guidance by replacing Inter font example with DM Sans
  - Removed conflicting recommendation where Inter was suggested as body font while also listed as anti-pattern
  - Ensures consistent guidance to avoid overused fonts (Inter/Roboto/Arial/Space Grotesk) and prefer distinctive alternatives

## [0.3.0] - 2025-12-15

### Added

- Enhanced design system enforcement across workflow
  - Comprehensive design system creation requirements for new applications
  - Mandatory design system validation in code generation pipeline
  - Increased test coverage expectations (50-75 tests with 20-30 visual tests)
  - Design system documentation requirements in architect specifications
  - Validation rules to prevent generic "AI slop" aesthetics across Cascade workflow

### Changed

- Architect agent: Enhanced MODE 1 with comprehensive design system creation requirements
- Coder agent: Integrated mandatory design system validation in feature implementation
- Project initializer agent: Updated test coverage expectations with visual testing focus
- AI workflow documentation: Comprehensive guide for design system enforcement

## [0.2.0] - 2025-12-13

### Added

- Iterative improvement workflow for existing applications
  - Architect agent MODE 2 for creating improvement specifications
  - Project initializer agent MODE 2 for managing improvement tracking
  - Coder agent automatic improvement detection and prioritization
  - Automatic merge of verified improvements into feature_list.json
  - Support for multiple improvement cycles on existing applications

### Enhanced

- Architect agent: Added improvement specification mode
- Project initializer agent: Added improvement list management
- Coder agent: Implemented improvement workflow orchestration with automatic mode detection
- AI workflow documentation: Comprehensive improvement workflow guide

### Fixed

- Mode detection consistency in coder agent bash scripts (improvement_list.json check)

## [0.1.0] - 2025-12-12

### Added
- Initial project structure with Claude AI workflow documentation
- Agent definitions for specialized development tasks:
  - Architect agent for technical specifications
  - Project initializer agent for project setup
  - Coder agent for feature implementation
  - Release engineer agent for version management
  - Grist database specialist agent for database setup
- AI workflow documentation (AI_WORKFLOW.md)
- Version control setup with .gitignore
- VERSION file for version tracking
- CHANGELOG.md for release notes

[0.1.0]: https://github.com/hbiaou/cascade-boilerplate-new-app/releases/tag/v0.1.0
