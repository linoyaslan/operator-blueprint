---
name: generate-feature-spec
description: Automatically finds design-session.md for feature design (PATH B), generates feature-[name].md specification, and cleans up session file
---

# Generate Feature Specification

**Purpose**: Auto-detect design session file, verify it's a feature/workflow design (PATH B), extract interview data, apply feature spec template, and clean up.

## Execution (Fully Automatic)

### Step 1: Locate Session File

Search for `design-session.md` in the current directory.

**If not found:**
```
❌ Cannot generate feature spec: design-session.md not found
This skill requires a completed design session from the spec-designer agent.
```

### Step 2: Verify Design Type is PATH B (Feature/Workflow)

Read `design-session.md` and check for design type indicator.

Look for markers indicating Feature/Workflow design:
- Section titled "# Design Type:" with value "Feature Implementation" or "PATH B"
- Presence of "Reconciliation Logic" section
- Presence of "Error Handling" section
- Presence of "RBAC Permissions" section
- Presence of "Testing Strategy" section

**If this is PATH A (CR Definition):**
```
⚠️  Wrong skill: This session designed a CR, not a feature
Use the generate-cr-spec skill instead.
```

### Step 3: Extract Feature Data from Session

Parse the design session file:

**Overview:**
- Extract: Feature name
- Extract: Purpose (2-3 sentences)
- Extract: Scope (In Scope / Out of Scope items)
- Extract: Trigger (what starts this feature)
- Extract: Prerequisites

**Target CR:**
- Extract: CR Kind
- Extract: API Group
- Extract: Domain
- Extract: Version
- Compute: Full API Group and Full API

**CR Fields Used:**
- Extract: Spec fields read by this feature
- Extract: Purpose of each field in this feature

**Status Fields Added/Modified:**
- Extract: New status fields (name, type, purpose, when populated, example)
- Extract: Modified existing fields
- Extract: New conditions (type, purpose, reason codes)
- Extract: Modified existing conditions

**Reconciliation Logic:**
- Extract: Desired state source
- Extract: Actual state source
- Extract: Reconciliation flow (step-by-step)
- Extract: Detailed steps (action, idempotency, error handling, requeue)
- Extract: Finalizer handling (name, deletion flow)
- Extract: Requeue strategy

**External Integrations** (for each):
- Extract: System name
- Extract: Connection details (endpoint, protocol, API version, authentication)
- Extract: Client library
- Extract: API operations (purpose, method, endpoint, request/response, errors, retry, timeout)
- Extract: Error handling
- Extract: Example client usage

**Error Handling:**
- Extract: Error categories (validation, transient, permanent)
- Extract: Edge cases (scenario, detection, handling)

**RBAC Permissions:**
- Extract: ClusterRole rules
- Extract: Permission justifications
- Extract: ServiceAccount

**Testing Strategy:**
- Extract: Unit test cases
- Extract: Integration test cases
- Extract: E2E test cases
- Extract: Mock requirements

**Performance & Scale:**
- Extract: Expected scale
- Extract: Rate limiting
- Extract: Timeout configuration
- Extract: Parallelism strategy
- Extract: Caching

**Observability:**
- Extract: Logging (Info/Debug/Error levels)
- Extract: Metrics
- Extract: Events
- Extract: Distributed tracing

**Dependencies:**
- Extract: Go dependencies
- Extract: Dependency matrix
- Extract: Compatibility validation

### Step 4: Generate feature-[name].md

Determine output filename:
- Extract feature name, convert to kebab-case
- Example: "Database Provisioning" → `database-provisioning`
- Filename: `feature-{name-kebab-case}.md`

Apply the Feature specification template with extracted data.

**Template:**

Use this template when you designed a feature/workflow (PATH B):

```markdown
# Feature Technical Specification: [Feature Name]

## Metadata
- **Feature**: [Feature Name]
- **Operator**: [Operator Name from PRD]
- **Version**: [Target version for implementation]
- **Created**: [Current date]
- **Status**: Ready for Implementation

## Overview

### Purpose
[2-3 sentence description of what this feature does]

### Scope
**In Scope:**
- [Item 1]
- [Item 2]

**Out of Scope:**
- [Item 1]
- [Item 2]

### Trigger
[What triggers this feature - be specific]

### Prerequisites
- [Prerequisite 1]
- [Prerequisite 2]

## CR Fields Used/Modified by This Feature

### Target CR
- **Kind**: `[CRKind]`
- **Group**: `[group]`
- **Domain**: `[domain]`
- **Version**: `[version]`
- **Full API Group**: `[group].[domain]`
- **Full API**: `[group].[domain]/[version]`

### Spec Fields Read
This feature reads the following spec fields from the CR:

| Field | Type | Purpose in This Feature |
|-------|------|-------------------------|
| `[field1]` | [type] | [How this feature uses it] |
| `[field2]` | [type] | [How this feature uses it] |

### Status Fields Added by This Feature

If this feature adds NEW status fields to the CR:

```go
// Add to [CRKind]Status struct:
type [CRKind]Status struct {
    // ... existing fields ...

    // [FieldName] [description of what this field tracks]
    [FieldName] [type] `json:"fieldName,omitempty"`
}
```

**Field:** `[fieldName]`
- **Type:** `[Go type]`
- **Purpose:** [What this tracks]
- **Populated When:** [When/how this is set]
- **Example:** `[example value]`

[Repeat for each new status field]

### Status Fields Modified by This Feature

This feature updates the following existing status fields:

| Field | Type | How Modified | When |
|-------|------|--------------|------|
| `phase` | string | Set to "[Phase]" | [When] |
| `[field]` | [type] | [How modified] | [When] |

### Conditions Added/Modified by This Feature

#### NEW Condition: `[ConditionType]`
- **Type:** `[ConditionType]`
- **Purpose:** [What this condition indicates]
- **Set to True:** [When/why]
- **Set to False:** [When/why]
- **Reason Codes:**
  - `[ReasonCode1]`: [Description]
  - `[ReasonCode2]`: [Description]
- **Example Message:** "[Example message]"

#### MODIFIED Condition: `[ExistingConditionType]`
- **Modified By This Feature:** [How]
- **New Reason Codes Added:**
  - `[ReasonCode]`: [Description]

## Reconciliation Logic

### State Detection

**Desired State Source:**
[Describe how to determine what should exist]

**Actual State Source:**
[Describe how to determine what currently exists]

### Reconciliation Flow

```
1. [Step 1 name]
   ↓
2. [Step 2 name]
   ↓
3. [Step 3 name]
   ↓
4. [Step 4 name]
```

### Detailed Steps

#### Step 1: [Name]
**Action:** [What happens]
**Idempotency:** [How this step is idempotent]
**Error Handling:** [What to do on error]
**Requeue:** [Whether to requeue and how]

```go
// Pseudocode outline
func Step1(ctx context.Context, cr *CR) error {
    // Implementation outline
}
```

[Repeat for each step]

### Idempotency Guarantees
[Explain how the entire reconciliation is idempotent]

### Finalizer Handling

**Finalizer Name:** `[group]/[feature-name]`

**Deletion Flow:**
1. [Cleanup step 1]
2. [Cleanup step 2]
3. Remove finalizer

```go
func handleDeletion(ctx context.Context, cr *CR) error {
    // Cleanup pseudocode
}
```

### Requeue Strategy

| Condition | Requeue | Delay | Reason |
|-----------|---------|-------|--------|
| Success | No | N/A | Complete |
| Transient error | Yes | Exponential (1s → 60s) | Retry |
| Validation error | No | N/A | User must fix |

## Status Management

### Status Fields

```go
type [Feature]Status struct {
    Phase string `json:"phase,omitempty"`
    Conditions []metav1.Condition `json:"conditions,omitempty"`
    // Additional fields with comments
}
```

### Status Phases

| Phase | Meaning | Next Phase(s) |
|-------|---------|---------------|
[Define each phase]

### Conditions

#### Condition: [Type]
- **Type:** `[ConditionType]`
- **Status:** True/False/Unknown
- **Reason:** [Machine-readable reason]
- **When True:** [Meaning]
- **When False:** [Meaning]

```go
// Example condition setting
```

[Repeat for each condition]

### Events

| Event Type | Reason | Message Template | When |
|------------|--------|------------------|------|
[Define each event]

## External Integrations

### Integration: [System Name]

#### Connection Details
- **Endpoint:** `[URL/discovery method]`
- **Protocol:** [Protocol]
- **API Version:** [Version]
- **Authentication:** [Method and details]

#### Client Library
```go
import "[library-path]"
// Version: [version]
```

#### API Operations

##### Operation 1: [Name]
**Purpose:** [What it does]
**Method:** `[HTTP/RPC method]`
**Endpoint:** `[Path]`
**Request:**
```json
[Example request]
```
**Response:**
```json
[Example response]
```
**Error Codes:**
- `[code]`: [meaning] → [action]

**Retry Strategy:** [Details]
**Timeout:** [Duration]
**Rate Limiting:** [Strategy]

[Repeat for each operation]

#### Error Handling

| Error | Retryable | Action | Status Update |
|-------|-----------|--------|---------------|
[Define error handling]

#### Example Client Usage
```go
// Concrete example code
```

[Repeat for each external integration]

## Error Handling

### Error Categories

#### Validation Errors
**Examples:**
- [Example 1]
- [Example 2]

**Handling:**
[Step-by-step handling]

```go
// Example code
```

[Repeat for Transient Errors, Permanent Errors]

### Edge Cases

#### Edge Case 1: [Description]
**Scenario:** [When]
**Detection:** [How to detect]
**Handling:** [What to do]

```go
// Pseudocode
```

[Repeat for each edge case]

## RBAC Permissions

### ClusterRole Rules

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: [operator-name]-[feature-name]
rules:
[Complete RBAC rules]
```

### Permission Justification

| Resource | Verbs | Justification |
|----------|-------|---------------|
[Explain each permission]

### Required ServiceAccount
```yaml
[ServiceAccount YAML]
```

## Testing Strategy

### Unit Tests

#### Test Suite: [Component]
**File:** `[path]`

**Test Cases:**
1. **Test_[Name]**
   - **Given:** [Setup]
   - **When:** [Action]
   - **Then:** [Expected]
   - **Mocks:** [What to mock]

[Repeat for each test case]

### Integration Tests
[Similar structure]

### E2E Tests
[Similar structure]

### Mock Requirements
```go
// Mock implementations needed
```

## Performance & Scale

### Expected Scale
- **Number of CRs:** [Number]
- **Reconciliation frequency:** [Frequency]
- **Expected duration:** [Duration]
- **Concurrent reconciliations:** [Number]

### Rate Limiting
```go
// Configuration example
```

### Timeout Configuration

| Operation | Timeout | Justification |
|-----------|---------|---------------|
[Define timeouts]

### Parallelism
**Strategy:** [Sequential/Parallel/Mixed]
**Rationale:** [Why]
**Implementation:**
```go
// Example code
```

### Caching
[If applicable]

## Observability

### Logging

#### Log Levels
**Info Level:**
- [Log entry 1]
- [Log entry 2]

**Debug Level:**
- [Log entry 1]

**Error Level:**
- [Log entry 1]

```go
// Example logging code
```

### Metrics

```go
// Metric definitions
```

**Metrics to track:**
- [Metric 1]: [Purpose]
- [Metric 2]: [Purpose]

### Events
```go
// Event emission examples
```

### Distributed Tracing
[If applicable]

## Dependencies

### Go Dependencies

```go
// go.mod additions
require (
    [dep1] [version] // Purpose
)
```

### Dependency Matrix

| Dependency | Version | Min K8s | License | Purpose |
|------------|---------|---------|---------|---------|
[Complete matrix]

### Compatibility Validation
[Specific compatibility notes]

## Implementation Notes

### Code Structure
```
[Directory structure]
```

### Key Functions
```go
// Function signatures
```

### Best Practices
- [Practice 1]
- [Practice 2]
[Comprehensive list]

## Open Questions

[Any unresolved technical questions]

## References

- [Link 1]
- [Link 2]

---

**Status:** Ready for implementation by Developer agent

**Next Steps:**
1. Developer agent implements following this spec
2. Ensure all unit tests pass
3. Run integration tests
4. Update operator documentation
```

Write this content to `feature-[name].md`.

### Step 5: Clean Up Session File

Delete `design-session.md` (it was temporary working memory).

### Step 6: Report Success

Output:
```
✅ Feature Specification Generated Successfully

Created: feature-[name].md

Summary:
- Feature: [name]
- Target CR: [CR Kind]
- External Integrations: [count]
- Reconciliation Steps: [count]
- Test Cases: [count]
- RBAC Permissions: [resource count]

Session file (design-session.md) has been cleaned up.

Next Steps:
- Review the feature specification for accuracy
- Ready for implementer agent to write the code
```

## Error Handling

**Session file not found:**
```
❌ Error: design-session.md not found
Cannot generate feature spec without a design session.
Run the spec-designer agent first with PATH B (Feature/Workflow).
```

**Wrong design type (CR, not Feature):**
```
⚠️  Error: Wrong design type
This session designed a CR (PATH A), not a Feature.
Use generate-cr-spec skill instead.
```

**Missing required sections:**
```
⚠️  Error: Incomplete feature design session
Missing required sections:
- Reconciliation Logic
- RBAC Permissions
- Testing Strategy

Cannot generate spec until design is complete.
```

**Cannot determine feature name:**
```
❌ Error: Cannot determine feature name
Overview section missing or malformed.
Manual review of design-session.md needed.
```

**Write permission error:**
```
❌ Error: Cannot write feature-[name].md
Check file permissions in current directory.
```

## Data Extraction Rules

1. **Auto-detect structure**: Parse session file to find sections
2. **Preserve code examples**: Include all code snippets from interview
3. **Maintain technical details**: Keep exact API versions, timeouts, etc.
4. **Build complete specs**: Fill all template sections with interview data
5. **No hallucination**: Only include data from session file

## When This Skill is Invoked

The spec-designer agent invokes this skill after:
1. PATH B (Feature/Workflow) interview is complete
2. All required sections captured
3. User confirms readiness to generate spec
4. Optional: After validate-session-file confirms completeness
