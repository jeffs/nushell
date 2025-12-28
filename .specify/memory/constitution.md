<!--
SYNC IMPACT REPORT
==================
Version change: 0.0.0 → 1.0.0 (Initial constitution)
Modified principles: N/A (first version)
Added sections:
  - Core Principles (3 principles)
  - Fork Goals
  - Development Workflow
  - Governance
Removed sections: N/A
Templates requiring updates:
  - .specify/templates/plan-template.md ✅ (no changes needed - Constitution Check section is generic)
  - .specify/templates/spec-template.md ✅ (no changes needed - requirements format compatible)
  - .specify/templates/tasks-template.md ✅ (no changes needed - task structure compatible)
  - .specify/templates/agent-file-template.md ✅ (no changes needed - generic structure)
  - .specify/templates/checklist-template.md ✅ (no changes needed - generic structure)
Follow-up TODOs: None
-->

# Nushell Fork Constitution

## Core Principles

### I. Upstream Compatibility

All changes MUST minimize merge conflicts in both directions:

- **Merging FROM upstream**: Changes MUST NOT restructure, rename, or relocate
  files/modules unless absolutely necessary for the feature
- **PRs TO upstream**: Bug fixes and minor enhancements MUST be structured as
  clean, isolated commits suitable for upstream contribution
- **Conflict avoidance**: Prefer additive changes over modifications to existing
  code paths; when modifications are required, keep them minimal and well-documented

**Rationale**: This fork exists to build upon upstream work while contributing
back. Unnecessary conflicts waste effort and reduce the value of both activities.

### II. Non-Breaking Defaults

Changes MUST NOT break other Nushell users' workflows when enabled by default:

- **Key bindings**: MUST NOT remap default key bindings without config opt-in
- **Command behavior**: MUST NOT change existing command semantics unless fixing
  a clear bug
- **Output formats**: MUST NOT alter default output formats that scripts may depend on
- **Configuration**: New features MUST work with default configuration or fail
  gracefully with clear guidance

**Rationale**: Other Nushell users rely on consistent, predictable behavior. Breaking
their workflows would harm the broader community and reduce upstream acceptance.

### III. Config-Gated Changes

Whenever feasible, changes MUST be gated behind build-time or runtime configuration:

- **Runtime config**: Prefer runtime configuration in `config.nu` for behavioral
  changes that users may want to toggle
- **Build-time features**: Use Cargo feature flags for larger optional components
  that add compile time or binary size
- **Environment variables**: Acceptable for development/debugging features but
  SHOULD NOT be the primary configuration mechanism for user-facing features
- **Documentation**: All configuration options MUST be documented with their
  default values and effects

**Rationale**: Config-gating allows this fork to diverge for personal use while
maintaining a clean path for upstream contributions and avoiding breaks for others.

## Fork Goals

This fork serves three specific purposes that guide all development decisions:

1. **Personal Customization**: Customize the shell to the maintainer's preferences
   while keeping changes isolated and reversible
2. **Component Reuse**: Leverage Nushell building blocks for constructing REPLs
   in larger applications, especially non-terminal-based system management tools
3. **Upstream Contribution**: Contribute meaningful changes upstream, prioritizing
   bug fixes and well-isolated enhancements

## Development Workflow

### Before Making Changes

1. **Upstream check**: Verify the change doesn't already exist in upstream or an
   open PR
2. **Conflict assessment**: Evaluate whether the change will cause merge conflicts
3. **Config evaluation**: Determine if the change can be config-gated

### During Implementation

1. **Minimal footprint**: Make the smallest change that achieves the goal
2. **Clear boundaries**: Keep fork-specific code in identifiable locations when possible
3. **Test coverage**: Maintain or improve test coverage for modified code

### Before Committing

1. **Upstream suitability**: If the change is suitable for upstream, structure it
   as a clean PR candidate
2. **Breaking check**: Verify no default behaviors are broken
3. **Config documentation**: Document any new configuration options

## Governance

- This constitution supersedes conflicting guidance in other project documents
- Amendments require updating this document with version increment and rationale
- All PRs and code reviews SHOULD verify compliance with these principles
- Exceptions MUST be documented with clear justification in the relevant PR or commit

**Version**: 1.0.0 | **Ratified**: 2025-12-28 | **Last Amended**: 2025-12-28
