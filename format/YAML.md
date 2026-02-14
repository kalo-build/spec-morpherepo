# YAML Format Specification (`KA:MR1:YAML1`)

## Overview

This document specifies the YAML format for defining MorpheRepo (`KA:MR1`) data access contracts. The `KA:MR1:YAML1` format serves as the canonical base format for the MorpheRepo ecosystem, providing a human-readable syntax for defining repository interfaces.

## Supported Features

The `KA:MR1:YAML1` format supports all core MorpheRepo specification features:

- **Identifiers** - Primary and secondary identifier definitions
- **Filters** - ForOne-derived filter parameters
- **Operations** - Per-model CRUD operation configuration
- **CompositeIdentifiers** - Multi-field identifier support
- **PolymorphicFilters** - Filters from ForOnePoly relationships

## File Extension

- `.repo` - Repository contract definitions

## Document Structure

Each `.repo` file defines the data access contract for a single Morphe model.

### Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Repository name (PascalCase, conventionally `{Model}Repository`) |
| `model` | string | Referenced Morphe model name |
| `identifiers` | object | Identifier definitions (must include `primary`) |
| `operations` | object | CRUD operation flags |

### Optional Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `filters` | list | `[]` | Filter parameter definitions for list operations |

## Identifiers

The `identifiers` section defines how individual records can be looked up. Every repository must define a `primary` identifier. Additional identifiers are named by their semantic purpose.

### Primary Identifier

```yaml
identifiers:
  primary:
    fields:
      - name: ID
        type: UUID
```

### Secondary Identifiers

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
  email:
    fields:
      - name: Email
        type: String
```

### Composite Identifiers

Identifiers can span multiple fields:

```yaml
identifiers:
  primary:
    fields:
      - name: ID
        type: AutoIncrement
  name:
    fields:
      - name: FirstName
        type: String
      - name: LastName
        type: String
```

### Identifier Field Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Field name (PascalCase, matching the Morphe model field) |
| `type` | string | Yes | Morphe field type (UUID, String, AutoIncrement, etc.) |

## Filters

The `filters` section defines optional parameters for the `GetAll` / list operation. Filters are derived from `ForOne` and `ForOnePoly` relationships in the source Morphe model.

```yaml
filters:
  - name: organizationID
    type: UUID
    relation: Organization
```

### Filter Field Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Parameter name (camelCase with ID suffix) |
| `type` | string | Yes | Morphe field type of the referenced model's primary identifier |
| `relation` | string | Yes | Source relationship name from the Morphe model |

### Empty Filters

Models with no `ForOne` or `ForOnePoly` relationships have an empty filters list:

```yaml
filters: []
```

## Operations

The `operations` section defines which CRUD methods the repository contract exposes:

```yaml
operations:
  list: true
  get: true
  create: true
  update: true
  delete: true
```

### Operation Reference

| Operation | Type | Description |
|-----------|------|-------------|
| `list` | boolean | Enable `GetAll` with optional filter parameters |
| `get` | boolean | Enable `GetBy{Identifier}` for each configured identifier |
| `create` | boolean | Enable `Create` with model input |
| `update` | boolean | Enable `Update` by primary identifier with model input |
| `delete` | boolean | Enable `Delete` by primary identifier |

Individual operations can be disabled:

```yaml
operations:
  list: true
  get: true
  create: true
  update: false
  delete: false
```

## Complete Examples

### Simple Model (no relations, with code identifier)

```yaml
# organization.repo
name: OrganizationRepository
model: Organization

identifiers:
  primary:
    fields:
      - name: ID
        type: UUID
  code:
    fields:
      - name: Code
        type: String

filters: []

operations:
  list: true
  get: true
  create: true
  update: true
  delete: true
```

### Model with ForOne Relation

```yaml
# specification.repo
name: SpecificationRepository
model: Specification

identifiers:
  primary:
    fields:
      - name: ID
        type: UUID
  code:
    fields:
      - name: Code
        type: String

filters:
  - name: organizationID
    type: UUID
    relation: Organization

operations:
  list: true
  get: true
  create: true
  update: true
  delete: true
```

### Model with Multiple ForOne Relations

```yaml
# format_version.repo
name: FormatVersionRepository
model: FormatVersion

identifiers:
  primary:
    fields:
      - name: ID
        type: UUID
  code:
    fields:
      - name: Code
        type: String

filters:
  - name: formatID
    type: UUID
    relation: Format
  - name: specificationVersionID
    type: UUID
    relation: SpecificationVersion

operations:
  list: true
  get: true
  create: true
  update: true
  delete: true
```

### Model with Polymorphic Relation

```yaml
# plugin.repo
name: PluginRepository
model: Plugin

identifiers:
  primary:
    fields:
      - name: ID
        type: UUID

filters:
  - name: formatID
    type: UUID
    relation: Format
  - name: ownerID
    type: UUID
    relation: Owner

operations:
  list: true
  get: true
  create: true
  update: true
  delete: true
```

### Model without Secondary Identifiers

```yaml
# plugin_version.repo
name: PluginVersionRepository
model: PluginVersion

identifiers:
  primary:
    fields:
      - name: ID
        type: UUID

filters:
  - name: formatVersionID
    type: UUID
    relation: FormatVersion
  - name: pluginID
    type: UUID
    relation: Plugin

operations:
  list: true
  get: true
  create: true
  update: true
  delete: true
```

## Validation Rules

1. `name` must be non-empty and end with `Repository`
2. `model` must be non-empty and reference a valid Morphe model name
3. `identifiers` must contain a `primary` key
4. Each identifier must have at least one field with `name` and `type`
5. Field `type` must be a valid Morphe atomic field type
6. Filter `name` must be non-empty
7. Filter `relation` must reference a valid relationship name
8. At least one operation must be enabled

## Contributing

See the main [MorpheRepo `KA:MR1` specification](../README.md) for contribution guidelines.

## License

This project is licensed under the MIT License.
