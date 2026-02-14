# MorpheRepo Specification (KA:MR1) - Data Access Contract Modeling

## Table of Contents

- [Overview](#overview)
- [Specification Versions](#specification-versions)
- [Specification Hierarchy](#specification-hierarchy)
- [Key Features](#key-features)
- [Motivation](#motivation)
- [Core Concepts](#core-concepts)
  - [Repository](#repository)
  - [Model Reference](#model-reference)
  - [Identifiers](#identifiers)
  - [Filters](#filters)
  - [Operations](#operations)
- [Relationship to Morphe (KA:MO1)](#relationship-to-morphe-kamo1)
- [Specification Features](#specification-features)
- [Contributing](#contributing)
- [License](#license)

## Overview

MorpheRepo (KA:MR1) is a language-agnostic data access contract specification derived from Morphe (KA:MO1) model definitions. It describes the interface between application code and data persistence, defining what operations are available for each model, how models can be identified, and what filtering parameters are supported.

The specification decouples the "what does the data access contract look like" question from "what language or framework am I targeting." A MorpheRepo definition can be compiled into Go interfaces, TypeScript types, Java interfaces, or any other language-specific repository abstraction.

## Specification Versions

Version | Status | Description | Docs
--------|---------|------------|------
KA:MR1 | In Progress | First stable release of the MorpheRepo core specification | This document
KA:MR1:YAML1 | In Progress | YAML format standard for KA:MR1 | [format/YAML.md](format/YAML.md)
KA:MR1:GO1 | In Progress | Go format standard for KA:MR1 | Planned

## Specification Hierarchy

The MorpheRepo specification follows the same hierarchical naming scheme as Morphe:

### Base Specification (`KA:MR1`)
- `KA:` - Organization/context prefix
- `RE1` - Base specification identifier (Repository version 1)

### Format Specification (`KA:MR1:YAML1`)
- Extends base specification with format identifier
- `YAML1` - Format representation identifier (YAML version 1)

### Base Format
**YAML** (`KA:MR1:YAML1`) serves as the canonical base format, consistent with the Morphe ecosystem.

## Key Features

1. **Language-Agnostic Contracts**
   * Define data access interfaces independent of implementation language
   * Compile to Go interfaces, TypeScript types, or any target
   * Consistent contract definitions across your stack

2. **Derived from Morphe**
   * Automatically generated from Morphe model definitions
   * Identifiers and filters derived from model structure
   * No manual synchronization between models and repositories

3. **Hub-and-Spoke Composition**
   * MorpheRepo serves as an intermediate spec between Morphe and language-specific implementations
   * Multiple input sources can produce .repo files (Morphe, OpenAPI, etc.)
   * Multiple output targets consume .repo files (Go, TypeScript, etc.)

4. **Configurable Operations**
   * Per-model control over which CRUD operations are available
   * Operations can be enabled or disabled individually
   * Downstream plugins respect operation configuration

## Motivation

When building applications with Morphe, the data access layer represents a critical contract between your domain models and their persistence implementations. Without a formal specification for this contract:

* Repository interfaces are hand-written and drift from model definitions
* Each language target reimplements the same contract derivation logic
* Changes to model identifiers or relationships require manual updates across all repository definitions

MorpheRepo solves this by providing a single, inspectable intermediate representation of the data access contract. Plugins upstream (e.g., plugin-morphe-morpherepo) generate .repo files from models, and plugins downstream (e.g., plugin-morpherepo-go) compile them into language-specific interfaces.

## Core Concepts

### Repository

A repository represents the data access contract for a single Morphe model. Each repository has a name, a model reference, identifiers, optional filters, and a set of operations.

### Model Reference

The `model` field references the Morphe model name that this repository manages. This name is used by downstream plugins to resolve the corresponding language-specific type (e.g., Go struct, TypeScript type).

### Identifiers

Identifiers define how individual records can be looked up. They are derived from the Morphe model's `identifiers` section.

- **Primary identifier**: Always present. Defines the canonical lookup method (e.g., by UUID).
- **Secondary identifiers**: Optional. Each secondary identifier generates an additional lookup operation (e.g., by code, by email).

Each identifier consists of one or more fields with their Morphe field types.

*Example:*

```yaml
identifiers:
  primary:
    fields:
      - name: ID
        type: UUID
  code:
    fields:
      - name: Code
        type: String
```

### Filters

Filters define optional parameters for list/query operations. They are derived from `ForOne` and `ForOnePoly` relationships in the Morphe model.

Each filter has a name (the Go-style parameter name), a Morphe field type, and a reference to the source relationship.

*Example:*

```yaml
filters:
  - name: organizationID
    type: UUID
    relation: Organization
```

### Operations

Operations define which CRUD methods the repository contract exposes. Each can be individually enabled or disabled.

| Operation | Description | Generated Methods |
|-----------|-------------|-------------------|
| `list` | Query all records with optional filters | `GetAll(filters...) -> []Model` |
| `get` | Retrieve a single record by identifier | `GetByID(id) -> Model`, `GetByCode(code) -> Model`, etc. |
| `create` | Create a new record | `Create(input) -> Model` |
| `update` | Update an existing record by primary identifier | `Update(id, input) -> Model` |
| `delete` | Remove a record by primary identifier | `Delete(id) -> void` |

When `get` is enabled, one getter method is generated per configured identifier (primary + all secondary identifiers).

## Relationship to Morphe (KA:MO1)

MorpheRepo is designed as a downstream specification from Morphe. The derivation rules are:

| Morphe Concept | MorpheRepo Derivation |
|----------------|----------------------|
| Model name | Repository `model` reference |
| `identifiers.primary` | Primary identifier |
| `identifiers.{name}` (secondary) | Secondary identifiers |
| `ForOne` relationships | Filter parameters |
| `ForOnePoly` relationships | Filter parameters (using `through` name if present) |
| `HasOne`, `HasMany` relationships | Not represented (no filters) |

## Specification Features

| Feature | Description | Status |
|---------|-------------|--------|
| **Identifiers** | Primary and secondary identifier definitions | Complete |
| **Filters** | ForOne-derived filter parameters | Complete |
| **Operations** | Per-model CRUD operation configuration | Complete |
| **CompositeIdentifiers** | Multi-field identifier support | Complete |
| **PolymorphicFilters** | Filters from ForOnePoly relationships | Complete |

## Contributing

We welcome contributions to the MorpheRepo specification. Start by creating an issue to discuss proposed changes. See the main [Morphe specification](https://github.com/kalo-build/morphe-spec) for general contribution guidelines.

## License

This project is licensed under the MIT License.
