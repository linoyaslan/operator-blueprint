---
name: generate-cr-spec
description: Automatically finds design-session.md for CR design (PATH A), generates cr-[Kind].md specification, and cleans up session file
---

# Generate Custom Resource Specification

**Purpose**: Auto-detect design session file, verify it's a CR definition (PATH A), extract interview data, apply CR spec template, and clean up.

## Execution (Fully Automatic)

### Step 1: Locate Session File

Search for `design-session.md` in the current directory.

**If not found:**
```
❌ Cannot generate CR spec: design-session.md not found
This skill requires a completed design session from the spec-designer agent.
```

### Step 2: Verify Design Type is PATH A (CR Definition)

Read `design-session.md` and check for design type indicator.

Look for markers indicating CR Definition design:
- Section titled "# Design Type:" with value "CR Definition" or "PATH A"
- Presence of "API Structure" section
- Presence of "Spec Fields" section
- Presence of "Status Fields" section

**If this is PATH B (Feature/Workflow):**
```
⚠️  Wrong skill: This session designed a feature, not a CR
Use the generate-feature-spec skill instead.
```

### Step 3: Extract CR Data from Session

Parse the design session file:

**API Structure:**
- Extract: Kind (e.g., "DPFHCPBridge", "Database")
- Extract: Group (e.g., "dpf")
- Extract: Domain (e.g., "hcp.bridge.com")
- Extract: Version (e.g., "v1alpha1")
- Compute: Full API Group = `{group}.{domain}`
- Compute: Full API = `{group}.{domain}/{version}`
- Compute: CRD Name = `{kind-plural}.{group}.{domain}`

**Overview:**
- Extract: Purpose (2-3 sentences)
- Extract: Scope (Namespaced / Cluster-scoped)
- Extract: Relationships (to other CRs)

**Spec Fields** (for each field):
- Extract: Field name
- Extract: Go type
- Extract: Required/Optional
- Extract: Description
- Extract: Validation constraints
- Extract: Default value (if any)
- Extract: Immutable flag
- Extract: Example value

**Status Fields** (for each field):
- Extract: Field name
- Extract: Go type
- Extract: Description
- Extract: Populated by (which feature)
- Extract: Example value

**Status Phases:**
- Extract: Phase names and meanings
- Extract: Phase transitions

**Status Conditions:**
- Extract: Condition types
- Extract: Meanings (True/False/Unknown)
- Extract: Reason codes
- Extract: Which feature sets them

**Validation Rules:**
- Extract: Admission validation
- Extract: Reconciliation validation
- Extract: Cross-field validation

**Examples:**
- Extract: YAML examples provided

### Step 4: Generate cr-[Kind].md

Determine output filename:
- Convert Kind to lowercase: `DPFHCPBridge` → `dpfhcpbridge`
- Filename: `cr-{kind-lowercase}.md`

Apply the CR specification template with extracted data.

**Template:**

Use this template when you designed a CR schema (PATH A):

```markdown
# Custom Resource Definition: [CR Kind Name]

## Metadata
- **CR Kind**: [Kind]
- **Operator**: [Operator Name from PRD]
- **Scope**: Namespaced / Cluster-scoped
- **Created**: [Current date]
- **Status**: Ready for Implementation

## API Structure

This section defines the Kubernetes API structure for this Custom Resource.

- **Kind**: `[Kind]` (e.g., DPFHCPBridge)
- **Group**: `[group]` (e.g., dpf)
- **Domain**: `[domain]` (e.g., hcp.bridge.com)
- **Version**: `[version]` (e.g., v1alpha1)
- **Full API Group**: `[group].[domain]` (e.g., dpf.hcp.bridge.com)
- **Full API**: `[group].[domain]/[version]` (e.g., dpf.hcp.bridge.com/v1alpha1)
- **CRD Name**: `[kind-plural].[group].[domain]` (e.g., dpfhcpbridges.dpf.hcp.bridge.com)

## Overview

### Purpose
[2-3 sentence description of what this CR represents and why it exists]

### Relationships
[Describe how this CR relates to other CRs, if applicable]

## API Specification

### Complete Type Definition

```go
// [Kind]Spec defines the desired state of [Kind]
type [Kind]Spec struct {
    // [Complete struct with all fields, comments, and tags]
}

// [Kind]Status defines the observed state of [Kind]
type [Kind]Status struct {
    // [Complete struct with all status fields, comments, and tags]
}
```

## Spec Fields

### Field: `[fieldName]`
- **Type:** `[Go type]`
- **Required:** Yes/No
- **Description:** [What this field represents]
- **Validation:**
  - [Constraint 1]
  - [Constraint 2]
- **Default:** `[value]` (if optional)
- **Immutable:** Yes/No
- **Example:** `[example value]`

[Repeat for each spec field]

### Nested Types

[If there are nested structs, define them here with same detail level]

### Field Dependencies

- [Describe any field dependencies]
- [Describe mutually exclusive fields]

## Status Fields

### Field: `[fieldName]`
- **Type:** `[Go type]`
- **Description:** [What this field represents]
- **Populated By:** [Which reconciliation/feature populates this]
- **Example:** `[example value]`

[Repeat for each status field]

### Status Phases

| Phase | Meaning | Next Phases | Set By |
|-------|---------|-------------|--------|
| [Phase1] | [Description] | [Phase2, Phase3] | [Which feature] |

### Status Conditions

#### Condition: [Type]
- **Type:** `[ConditionType]`
- **Status:** True/False/Unknown
- **Meanings:**
  - **True**: [What this means]
  - **False**: [What this means]
  - **Unknown**: [What this means]
- **Reason Codes:**
  - `[ReasonCode1]`: [When/why]
  - `[ReasonCode2]`: [When/why]
- **Set By:** [Which feature(s) manage this condition]

[Repeat for each condition]

## Validation Rules

### Admission Validation
[What validation happens at admission time (webhook)]

### Reconciliation Validation
[What validation happens during reconciliation]

### Cross-Field Validation
[Any validation that involves multiple fields]

## Examples

### Minimal Example
```yaml
apiVersion: [group]/[version]
kind: [Kind]
metadata:
  name: minimal-example
spec:
  [Minimal required fields]
```

### Complete Example
```yaml
apiVersion: [group]/[version]
kind: [Kind]
metadata:
  name: complete-example
spec:
  [All fields including optional]
status:
  [Example status - read-only, shown for reference]
```

### Common Patterns

#### Pattern 1: [Pattern Name]
```yaml
[Example showing common usage pattern]
```

## Implementation Notes

### Code Structure
- CR definition file: `api/[version]/[kind]_types.go`
- Webhook validation: `api/[version]/[kind]_webhook.go`
- Defaulting logic: `api/[version]/[kind]_defaults.go`

### Key Considerations
- [Important implementation note 1]
- [Important implementation note 2]

## References

- [Link to related documentation]
- [Link to API specs]

---

**Status:** Ready for implementation
**Next Steps:**
1. Generate CRD using controller-gen
2. Implement webhook validation
3. Design features/workflows that use this CR
```

Write this content to `cr-[kind].md`.

### Step 5: Clean Up Session File

Delete `design-session.md` (it was temporary working memory).

### Step 6: Report Success

Output:
```
✅ CR Specification Generated Successfully

Created: cr-[kind].md

Summary:
- CR Kind: [Kind]
- API: [group].[domain]/[version]
- CRD Name: [kind-plural].[group].[domain]
- Spec Fields: [count]
- Status Fields: [count]
- Conditions: [count]
- Validation Rules: [count]

Session file (design-session.md) has been cleaned up.

Next Steps:
- Review the CR specification for accuracy
- Ready for implementer agent to create the API types
```

## Error Handling

**Session file not found:**
```
❌ Error: design-session.md not found
Cannot generate CR spec without a design session.
Run the spec-designer agent first with PATH A (CR Definition).
```

**Wrong design type (Feature, not CR):**
```
⚠️  Error: Wrong design type
This session designed a Feature/Workflow (PATH B), not a CR.
Use generate-feature-spec skill instead.
```

**Missing required sections:**
```
⚠️  Error: Incomplete CR design session
Missing required sections:
- API Structure
- Spec Fields
- Status Fields

Cannot generate spec until design is complete.
```

**Cannot determine Kind:**
```
❌ Error: Cannot determine CR Kind name
API Structure section missing or malformed.
Manual review of design-session.md needed.
```

**Write permission error:**
```
❌ Error: Cannot write cr-[kind].md
Check file permissions in current directory.
```

## Data Extraction Rules

1. **Auto-detect structure**: Parse session file to find sections
2. **Preserve exact types**: Use Go types exactly as specified in session
3. **Maintain examples**: Include all YAML examples from interview
4. **Build complete structs**: Generate full Go type definitions with tags
5. **No hallucination**: Only include data from session file

## When This Skill is Invoked

The spec-designer agent invokes this skill after:
1. PATH A (CR Definition) interview is complete
2. All required sections captured
3. User confirms readiness to generate spec
4. Optional: After validate-session-file confirms completeness
