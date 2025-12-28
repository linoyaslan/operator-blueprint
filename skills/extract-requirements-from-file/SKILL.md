---
name: extract-requirements-from-file
description: Extracts operator requirements from an existing design document and populates requirements-session.md with found information, identifying what's complete vs. missing
---

# Extract Requirements from Existing File

**Purpose**: Read an existing design document (markdown, PRD, spec, notes) and extract requirements information into the standard requirements-session.md format, clearly marking what was found and what still needs to be gathered through interview.

## Execution (Fully Automatic)

### Step 1: Receive File Path

The agent will provide the file path that the user specified.

**Expected input format:**
- Absolute or relative file path
- Common formats: .md, .txt, .doc (markdown preferred)

### Step 2: Read the File

Read the entire file content.

**If file not found or unreadable:**
```
❌ Error: Cannot read file
File: [path]
Please verify the file path and permissions.
```

### Step 3: Extract Information by Category

Analyze the file content and try to extract information for each of the 5 required categories:

#### Category 1: Operator Name
**Look for:**
- Explicit operator name mentions
- Project name
- Repository name
- Title or heading

**Extraction markers:**
- "Operator Name:", "Operator:", "Project Name:"
- Document title (if it looks like an operator name)
- Phrases like "X operator", "X controller"

#### Category 2: Problem Statement & Purpose
**Look for:**
- Problem description
- "Why" this operator exists
- Manual processes being automated
- Pain points
- Goals or objectives

**Extraction markers:**
- Sections titled: "Problem", "Background", "Motivation", "Purpose", "Goals"
- Phrases like "currently", "manually", "automate", "solve"

#### Category 3: Custom Resources Definition
**Look for:**
- CR/CRD definitions
- Resource kinds
- API groups
- Schema definitions
- Lifecycle descriptions

**Extraction markers:**
- "Custom Resource", "CR", "CRD", "Kind:"
- API definitions (apiVersion, kind)
- YAML examples with apiVersion/kind
- Lifecycle stages (Created, Ready, Updated, Deleted)

**For each CR found, extract:**
- CR Kind name
- What it represents
- Lifecycle stages
- Relationships to other CRs

#### Category 4: Operator Responsibilities
**Look for:**
- What the operator does
- Reconciliation behaviors
- Automation tasks
- Features

**Extraction markers:**
- Sections: "Responsibilities", "Features", "Behavior", "Reconciliation"
- Lists of operator actions
- "The operator will...", "Handles...", "Manages..."

**For each responsibility, extract:**
- Trigger (what starts it)
- Outcome (what happens)

#### Category 5: Success Criteria & Deployment
**Look for:**
- How users know it works
- Success indicators
- Testing criteria
- User workflows
- Deployment method

**Extraction markers:**
- Sections: "Success Criteria", "Testing", "Deployment", "Installation"
- "User should see...", "Indicates success...", "Workflow..."

### Step 4: Build requirements-session.md

Create the session file with extracted information.

**For each category:**

**If information found:**
```markdown
# Category X: [Name]
## Status: EXTRACTED - Needs Verification

## Extracted Information:
[The information found in the file, formatted according to category structure]

## Source:
- File: [original file path]
- Extraction confidence: [High/Medium/Low]
```

**If NOT found:**
```markdown
# Category X: [Name]
## Status: NOT FOUND - Interview Required

## Note:
No information for this category was found in the provided file.
This will need to be gathered through interview.
```

**Confidence levels:**
- **High**: Explicit section with clear information
- **Medium**: Inferred from scattered mentions
- **Low**: Partial or ambiguous information

### Step 5: Generate Extraction Report

Output a summary of what was extracted:

```
═══════════════════════════════════════
Requirements Extraction Report
═══════════════════════════════════════

Source File: [path]
Date: [current date]

Extraction Results
──────────────────
✅ Category 1: Operator Name (High confidence)
   Found: postgres-ha-operator

⚠️  Category 2: Problem Statement (Medium confidence)
   Found: Partial problem description
   Missing: Automation details

❌ Category 3: Custom Resources (Not found)
   Needs: Full interview

✅ Category 4: Operator Responsibilities (High confidence)
   Found: 3 responsibilities

⚠️  Category 5: Success Criteria (Low confidence)
   Found: Brief mentions
   Missing: User workflows, deployment method

Overall Status
──────────────
Categories Found: 2 complete, 2 partial, 1 missing
Next Steps:
1. Agent will read back extracted categories for verification
2. Agent will conduct interview for missing/incomplete categories
3. All categories will be marked VERIFIED after user confirmation

Session File Created: requirements-session.md
```

### Step 6: Return Control to Agent

The skill completes and returns the report. The agent will:
1. Read the extraction report
2. Read back extracted categories to user for verification
3. Interview for missing categories
4. Follow normal DRAFT → VERIFIED flow

## Extraction Intelligence

### Smart Pattern Recognition

**Operator names:**
- Recognize patterns: `*-operator`, `*-controller`, `operator-*`
- Extract from: titles, README headers, repository names

**Custom Resources:**
- Parse YAML blocks for apiVersion/kind
- Recognize Go struct definitions
- Find CRD examples

**Problem statements:**
- Look for pain point keywords: "manual", "difficult", "complex", "error-prone"
- Identify automation keywords: "automate", "simplify", "streamline"

**Responsibilities:**
- Recognize action verbs: "creates", "manages", "monitors", "reconciles"
- Extract from bulleted lists
- Parse "The operator will..." sentences

### Handling Multiple Formats

**Markdown documents:**
- Parse headings and sections
- Extract from lists and tables
- Recognize code blocks

**Existing PRDs:**
- Map to category structure
- Preserve original wording

**Design specs:**
- Extract technical details
- Map to appropriate categories

**Notes/drafts:**
- Best-effort extraction
- Mark low confidence

## Error Handling

**File not found:**
```
❌ Error: File not found
Path: [path]
Please check the file path and try again.
```

**Empty file:**
```
⚠️  Warning: File is empty
No information could be extracted.
Proceeding with full interview.
```

**Unrecognized format:**
```
⚠️  Warning: Unrecognized file format
Attempted best-effort extraction.
Results may have low confidence.
Please review carefully.
```

**No information extracted:**
```
⚠️  No Requirements Found
File: [path]
The file doesn't appear to contain operator requirements.
Proceeding with full interview.
```

## Validation Rules

1. **Never hallucinate**: Only extract information explicitly present in the file
2. **Preserve original wording**: Don't rephrase user's language
3. **Mark confidence**: Be honest about extraction quality
4. **Default to interview**: If unsure, mark as "NOT FOUND" rather than guessing
5. **No assumptions**: Don't infer information not stated in the file

## When This Skill is Invoked

The requirements-engineer agent invokes this skill when:
1. User answers "yes" to "Do you have an existing design document?"
2. User provides a file path
3. Agent needs to bootstrap requirements from existing documentation

## Benefits

1. **Time saving**: Reuse existing documentation
2. **Completeness check**: Identifies gaps in existing docs
3. **Structured output**: Converts any format to standard requirements structure
4. **Verification required**: All extracted info still needs user confirmation
5. **Flexible input**: Works with various document formats and structures
