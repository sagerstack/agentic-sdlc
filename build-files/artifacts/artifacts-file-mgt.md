# Artifacts File Management Guide

## Purpose & Overview

This document defines the standardized folder structures and file naming conventions for all product development artifacts. It serves as the central reference for agents generating MVPs, Epics, User Stories, and Implementation Plans.

### Key Objectives
- **Standardization**: Consistent folder structures across all product artifacts
- **Discoverability**: Clear naming conventions for easy navigation
- **Scalability**: Hierarchical organization that supports complex product development
- **Agent Guidance**: Clear instructions for automated artifact generation

## ⚠️ CRITICAL REPOSITORY LOCATION REQUIREMENTS

### Repository Boundary Rules
**ALL ARTIFACTS MUST BE CREATED WITHIN THE CURRENT WORKING REPOSITORY ONLY**

- **Current Repository Base**: Use the current working directory as the absolute base path
- **Domain Path**: All artifacts must be created under `{current_working_directory}/domain/`
- **Prohibited**: Creating files outside the current repository boundaries
- **Validation**: Always verify that file paths start with the current working directory

### Path Construction Rules
1. **Determine Current Repository**: Use the current working directory (e.g., `/Users/user/project/`)
2. **Build Domain Path**: Append `domain/` to create the base artifact path
3. **Never Use Absolute Paths**: That don't start with the current working directory
4. **Example Valid Path**: If working in `/Users/user/globaldex/`, then artifacts go in `/Users/user/globaldex/domain/`

### Agent Validation Checklist
Before creating any artifact, agents MUST verify:
- [ ] File path starts with current working directory
- [ ] Path includes `domain/` as the artifact root
- [ ] No files are created outside repository boundaries
- [ ] All folder creation stays within repository structure

## Folder Structure Standards

### Execution Structure (MVP/Epic/Story)

```
domain/execution/mvp{iteration}-{goal}/
├── mvp{iteration}-{goal}.md (main MVP document)
├── {epic-id}-{title}/
│   ├── {epic-id}-{title}.md
│   ├── {story-id}-{title}/
│   │   ├── {story-id}-{title}.md
│   │   └── {story-id}-impl-plan.md
│   └── {story-id}-{title}/
│       ├── {story-id}-{title}.md
│       └── {story-id}-impl-plan.md
└── {epic-id}-{title}/
    ├── {epic-id}-{title}.md
    └── {story-id}-{title}/
        ├── {story-id}-{title}.md
        └── {story-id}-impl-plan.md
```

### Research Structure

```
domain/research/
├── problem-opportunity-brief.md
├── market-research-{product-name}.md
├── competitive-solution-analysis.md
├── critical-research-findings.md
└── {research-topic}/
    ├── {document-name}.md
    └── {document-name}.md
```

**Purpose**: Contains all product research, market analysis, and competitive intelligence documents.

### Resources Structure

```
domain/resources/
└── {resource-category}/
    └── {resource-name}.md
```

**Purpose**: Contains technical references, best practices, and reusable knowledge assets.

### Naming Conventions

#### MVP Level
- **Folder**: `mvp{iteration}-{goal}/`
  - `{iteration}`: Auto-incremented number (1, 2, 3, etc.)
  - `{goal}`: 3-word kebab-case phrase from user's primary objective
- **Document**: `mvp{iteration}-{goal}.md`

#### Epic Level
- **Folder**: `{epic-id}-{title}/`
  - `{epic-id}`: Unique epic identifier (e.g., EP-001, EP-002)
  - `{title}`: Kebab-case epic title
- **Document**: `{epic-id}-{title}.md`

#### Story Level
- **Folder**: `{story-id}-{title}/`
  - `{story-id}`: Unique story identifier (e.g., US-001, US-002)
  - `{title}`: Kebab-case story title
- **Document**: `{story-id}-{title}.md`
- **Implementation Plan**: `{story-id}-impl-plan.md`
- **Critical Analysis**: `{story-id}-critical-analysis.md`

#### Research Level
- **Market Research**: `market-research-{product-name}.md`
- **Problem Brief**: `problem-opportunity-brief.md`
- **Competitive Analysis**: `competitive-solution-analysis.md`
- **Research Findings**: `critical-research-findings.md`
- **Topic Folders**: `{research-topic}/` (kebab-case, e.g., `mvp2-sentiment-analysis/`)
- **Research Documents**: `{document-name}.md` (kebab-case, descriptive names)

#### Resources Level
- **Category Folders**: `{resource-category}/` (kebab-case)
- **Resource Documents**: `{resource-name}.md` (kebab-case)

### Example Structure

```
domain/execution/mvp1-technical-swap-validation/
├── mvp1-technical-swap-validation.md
├── EP-001-user-authentication/
│   ├── EP-001-user-authentication.md
│   ├── US-001-user-registration/
│   │   ├── US-001-user-registration.md
│   │   ├── US-001-impl-plan.md
│   │   └── US-001-critical-analysis.md
│   └── US-002-user-login/
│       ├── US-002-user-login.md
│       ├── US-002-impl-plan.md
│       └── US-002-critical-analysis.md
└── EP-002-token-swap-interface/
    ├── EP-002-token-swap-interface.md
    ├── US-003-swap-form/
    │   ├── US-003-swap-form.md
    │   ├── US-003-impl-plan.md
    │   └── US-003-critical-analysis.md
    └── US-004-transaction-confirmation/
        ├── US-004-transaction-confirmation.md
        ├── US-004-impl-plan.md
        └── US-004-critical-analysis.md

# Note: In a real product with multiple MVPs,
# the next MVP would continue with EP-005, US-005, etc.
# maintaining product-wide uniqueness across all MVPs
```

## Auto-Derivation Logic

### MVP Iteration Auto-Increment
1. **Scan Path**: `domain/execution/` for existing `mvp*` folders
2. **Extract Numbers**: From folder names like `mvp1-goal`, `mvp2-goal` → [1, 2]
3. **Find Maximum**: `max([1, 2]) = 2`
4. **Increment**: `2 + 1 = 3`
5. **Create**: `domain/execution/mvp3-{new-goal}/`

### Epic ID Auto-Increment (Product-Wide Uniqueness)
1. **Scan Path**: `domain/execution/` recursively for ALL epic folders
2. **Extract Numbers**: From folder names like `EP-001-title`, `EP-002-title` → [1, 2]
3. **Find Maximum**: Across ALL MVPs to ensure product-wide uniqueness
4. **Increment**: `max + 1`
5. **Create**: `EP-{next-number}-{title}/` (e.g., `EP-003-new-epic/`)

### User Story ID Auto-Increment (Product-Wide Uniqueness)
1. **Scan Path**: `domain/execution/` recursively for ALL user story folders
2. **Extract Numbers**: From folder names like `US-001-title`, `US001-title` → [1, 2, ...]
3. **Find Maximum**: Across ALL MVPs and ALL Epics to ensure product-wide uniqueness
4. **Increment**: `max + 1`
5. **Create**: `US-{next-number}-{title}/` (e.g., `US-014-new-story/`)

### Goal Synthesis Examples
- User Input: "Build a basic crypto token swap interface for technical validation"
  - **Goal**: `technical-swap-validation`
- User Input: "Create user onboarding flow to test market adoption"
  - **Goal**: `market-adoption-test`
- User Input: "Develop payment processing system for revenue validation"
  - **Goal**: `revenue-processing-validation`

## Agent Instructions

### 🔒 MANDATORY REPOSITORY PATH VALIDATION
**BEFORE ANY FILE CREATION, AGENTS MUST:**
1. **Verify Current Working Directory**: Confirm you are operating within the correct repository
2. **Construct Correct Base Path**: `{current_working_directory}/domain/execution/`
3. **Validate Path**: Ensure all file paths start with the current working directory
4. **Never Use External Paths**: Do not create files outside the current repository structure

### For MVP Generation
- **REQUIRED BASE PATH**: `{current_working_directory}/domain/execution/mvp{iteration}-{goal}/`
- Auto-derive iteration number by scanning existing MVP folders
- Synthesize goal from user's primary objective into 3-word kebab-case
- Create main MVP document as `mvp{iteration}-{goal}.md`
- **PATH VALIDATION**: Verify all files are created within current repository

### For Epic Generation
- **REQUIRED BASE PATH**: Within MVP folder in current repository
- Create epic folder: `{epic-id}-{title}/` within MVP folder
- Use unique epic IDs (EP-001, EP-002, etc.) **unique across entire product**
- Generate epic document: `{epic-id}-{title}.md`
- Epic IDs must be sequential across **entire product** (not just within MVP)
- **PATH VALIDATION**: Confirm epic folders are within correct repository structure
- **UNIQUENESS VALIDATION**: Scan all MVPs to ensure ID uniqueness

### For Story Generation
- **REQUIRED BASE PATH**: Within epic folder in current repository
- Create story folder: `{story-id}-{title}/` within epic folder
- Use unique story IDs (US-001, US-002, etc.) **unique across entire product**
- Generate story document: `{story-id}-{title}.md`
- Generate implementation plan: `{story-id}-impl-plan.md`
- Story IDs must be sequential across **entire product** (not just within MVP)
- **PATH VALIDATION**: Ensure story folders are within repository boundaries
- **UNIQUENESS VALIDATION**: Scan all MVPs and Epics to ensure ID uniqueness

### For Implementation Plan Generation
- **REQUIRED BASE PATH**: Within story folder in current repository
- Always create within story folder
- Use naming pattern: `{story-id}-impl-plan.md`
- Link to corresponding story document
- Include technical specifications and development steps
- **PATH VALIDATION**: Confirm implementation plans are within correct repository structure

### For Critical Analysis Generation
- **REQUIRED BASE PATH**: Within story folder in current repository (same as implementation plan)
- Always create within story folder
- Use naming pattern: `{story-id}-critical-analysis.md`
- **Relationship**: 1:1 with implementation plan document
- **Purpose**: Records solution-critic analysis and recommendations
- **Lifecycle**: Updated in-place during solution refinement cycles
- **PATH VALIDATION**: Confirm critical analysis documents are within correct repository structure


## Quality Standards

### Repository Location Validation
- **CRITICAL**: All artifacts created within current repository boundaries
- File paths start with current working directory
- Domain folder structure maintained: `{current_working_directory}/domain/`
- No files created outside repository structure
- Path validation completed before file creation

### Folder Structure Validation
- All folders follow naming conventions
- Hierarchical organization maintained
- No duplicate IDs within scope
- Clear parent-child relationships
- Repository boundary compliance verified

### File Naming Validation
- Consistent ID patterns (EP-###, US-###)
- Kebab-case titles
- Standardized file extensions (.md)
- Implementation plan suffix (-impl-plan.md)
- Correct repository path structure

### Content Organization
- Each artifact type in appropriate folder
- Clear separation of concerns
- Logical grouping by epic themes
- Traceability from MVP to implementation
- Repository structure integrity maintained

---

*This Artifacts File Management guide ensures consistent organization and naming across all product development artifacts, enabling efficient navigation and automated generation.*