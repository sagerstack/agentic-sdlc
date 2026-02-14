# Critical Analysis Artifact

## Purpose & Overview

The Critical Analysis serves as a comprehensive review document that identifies production issues, gaps, dangerous assumptions, and challenges in implementation plans before development begins. It acts as a quality gate to prevent costly mistakes and ensure production readiness.

### Key Objectives
- **Risk Identification**: Uncover hidden assumptions and potential failure points
- **Guidance Compliance**: Verify adherence to technical solution guidance from MVP
- **Cost Validation**: Confirm budget constraints are respected
- **Production Readiness**: Identify gaps preventing successful deployment

### Integration Points
- **Input from**: Implementation Plan documents
- **Created by**: solution-critic agent during /solution workflow
- **References**: User Story, Epic, MVP artifacts for context
- **Feeds into**: Implementation Plan refinement cycles

## Critical Analysis Template Structure

```markdown
# Critical Analysis

## Metadata
| Field | Value |
|-------|-------|
| ID | US-###-CRITICAL-ANALYSIS |
| User Story ID | US-### |
| Implementation Plan ID | US-###-IMPL-PLAN |
| Created | YYYY-MM-DD HH:mm:ss |
| Last Updated | YYYY-MM-DD HH:mm:ss |
| Analysis Iteration | [1st Analysis / 2nd Analysis] |
| Confidence Level | [High / Medium / Low] |

## Executive Summary
[2-3 paragraphs highlighting the most critical findings that could prevent successful implementation. Focus on blockers and high-risk items that must be addressed.]

## Technical Solution Guidance Compliance

### Technology Stack Alignment
| Specified in MVP | Used in Plan | Compliance | Impact if Changed |
|------------------|--------------|------------|-------------------|
| [Language/Framework] | [What plan uses] | [✅/⚠️/❌] | [Risk assessment] |
| [Database] | [What plan uses] | [✅/⚠️/❌] | [Risk assessment] |
| [Third-party Service] | [What plan uses] | [✅/⚠️/❌] | [Risk assessment] |

### Architecture Pattern Compliance
| Pattern Type | MVP Guidance | Plan Implementation | Compliance |
|--------------|--------------|-------------------|------------|
| Patterns to Use | [Specified patterns] | [How plan implements] | [✅/⚠️/❌] |
| Patterns to Avoid | [Anti-patterns listed] | [Any violations found] | [✅/⚠️/❌] |

### Budget Compliance
| Cost Category | Budget Constraint | Projected Cost | Within Budget | Risk |
|---------------|------------------|----------------|---------------|------|
| Third-party APIs | [$X/month] | [$Y/month] | [Yes/No] | [Low/Med/High] |
| Infrastructure | [$X/month] | [$Y/month] | [Yes/No] | [Low/Med/High] |
| Total Monthly | [$X] | [$Y] | [Yes/No] | [Low/Med/High] |

## Critical Issues (Blockers)

### Issue 1: [Issue Title]
- **Severity**: CRITICAL
- **Description**: [Detailed explanation of the problem]
- **Evidence**: [Specific examples from the plan]
- **Impact**: [What will fail in production]
- **Recommendation**: [Specific fix required]
- **Addressed in Refinement**: [Yes/No - track if fixed]

### Issue 2: [Issue Title]
[Repeat structure for each critical issue]

## High-Risk Assumptions

### Assumption 1: [Assumption Description]
- **Risk Level**: HIGH
- **Current Assumption**: [What the plan assumes]
- **Reality Check**: [What actually happens in production]
- **Validation Method**: [How to verify this assumption]
- **Fallback Plan**: [What to do if assumption is false]
- **Addressed in Refinement**: [Yes/No]

## Scalability Analysis

### Data Volume Projections
| Metric | Plan Assumption | Realistic Estimate | Gap | Risk |
|--------|----------------|-------------------|-----|------|
| Daily transactions | [Number] | [Number] | [Delta] | [Low/Med/High] |
| Data growth/month | [Size] | [Size] | [Delta] | [Low/Med/High] |
| Concurrent users | [Number] | [Number] | [Delta] | [Low/Med/High] |

### Performance Bottlenecks
| Component | Expected Performance | Likely Reality | Impact |
|-----------|---------------------|----------------|---------|
| [API calls] | [X req/sec] | [Y req/sec based on research] | [System impact] |
| [Database queries] | [X ms latency] | [Y ms under load] | [User experience impact] |

## Third-Party Integration Risks

### Service 1: [Service Name]
- **Integration Method**: [REST/WebSocket/Webhook]
- **Rate Limits**: [Plan assumption vs documented limits]
- **Cost Model**: [Per request/monthly/tiered]
- **Reliability Concerns**: [SLA gaps, downtime history]
- **Fallback Strategy**: [Exists: Yes/No - Quality assessment]

## Missing Production Requirements

### Operational Readiness Gaps
- [ ] **Monitoring**: [What's missing for production]
- [ ] **Alerting**: [Missing alert scenarios]
- [ ] **Runbooks**: [Undefined operational procedures]
- [ ] **Disaster Recovery**: [Missing DR components]

### Security Gaps
- [ ] **Authentication**: [Security concerns]
- [ ] **Authorization**: [Access control gaps]
- [ ] **Data Protection**: [Encryption/privacy issues]
- [ ] **Audit Trail**: [Compliance gaps]

## Recommendations Priority Matrix

### Must Fix Before Development
1. **[Critical Issue]**: [Why it's a blocker and specific fix needed]
2. **[Critical Issue]**: [Why it's a blocker and specific fix needed]

### Should Address During Refinement
1. **[High-Risk Item]**: [Impact and recommended approach]
2. **[High-Risk Item]**: [Impact and recommended approach]

### Can Defer (Document for Future)
1. **[Low-Risk Item]**: [Why it can wait and when to address]

## Research & Validation

### Sources Consulted
- [Production system example with similar scale]
- [Official documentation validating concerns]
- [Industry benchmarks supporting analysis]
- [Community discussions about similar issues]

### Additional Research Needed
- [ ] **[Topic]**: [Why needed and where to research]
- [ ] **[Topic]**: [Why needed and where to research]

## Refinement Tracking

### First Refinement Cycle
- **Issues Addressed**: [List of fixed issues]
- **Issues Remaining**: [Outstanding concerns]
- **New Issues Found**: [Additional problems discovered]

### Second Refinement Cycle
- **Issues Addressed**: [List of fixed issues]
- **Issues Remaining**: [Outstanding concerns]
- **Assessment**: [Overall improvement assessment]

## Changelog

| Date | Author | Change Summary | Analysis Type |
|------|--------|----------------|---------------|
| YYYY-MM-DD HH:mm:ss | Solution Critic | Initial critical analysis | 1st Analysis |
| YYYY-MM-DD HH:mm:ss | Solution Critic | Updated after first refinement | 2nd Analysis |

```

## No Issues Scenario Template

When implementation plan comprehensively addresses all requirements with no gaps, use this minimal template to document successful validation without listing every fulfilled criterion:

```markdown
# Critical Analysis

## Metadata
| Field | Value |
|-------|-------|
| ID | US-###-CRITICAL-ANALYSIS |
| User Story ID | US-### |
| Implementation Plan ID | US-###-IMPL-PLAN |
| Created | YYYY-MM-DD HH:mm:ss |
| Last Updated | YYYY-MM-DD HH:mm:ss |
| Analysis Iteration | 1st Analysis |
| Confidence Level | High |

## Executive Summary

Implementation plan comprehensively addresses all user story requirements, acceptance criteria, and functional requirements. No critical issues, high-risk assumptions, or missing production requirements identified. Plan is ready for development.

## Technical Solution Guidance Compliance

Full compliance with user story technical guidance:

| Specified in User Story | Used in Plan | Compliance | Notes |
|------------------------|--------------|------------|-------|
| [Technology Stack] | [As specified] | ✅ | Matches requirements |
| [Architecture Pattern] | [As specified] | ✅ | Correctly applied |
| [Budget Constraints] | [Within limits] | ✅ | Budget respected |

No deviations from specified technical guidance.

## Recommendations Priority Matrix

### Should Address During Development
*Optional enhancements for improved quality (non-blocking):*

1. **[Optional Enhancement]**: [Brief description if any minor improvements identified]

*If no optional enhancements: "No recommendations. Implementation plan is ready for development."*

## Research & Validation

### Sources Consulted
- User story artifact: US-###-{title}.md
- Epic artifact: EP-###-{title}.md
- MVP artifact: mvp{N}-{goal}.md
- Implementation plan: US-###-impl-plan.md

### Validation Approach
Comprehensive requirements validation against:
- All acceptance criteria (functional, edge case, error handling, performance, security)
- All functional and technical requirements
- Technical solution guidance specifications
- Integration requirements and dependencies

## Refinement Tracking

### Refinement Cycle
No refinement required - implementation plan comprehensively addresses all requirements on first iteration.

## Changelog

| Date | Author | Change Summary | Analysis Type |
|------|--------|----------------|---------------|
| YYYY-MM-DD HH:mm:ss | Solution Critic | Initial critical analysis - comprehensive requirements validation with no gaps identified | 1st Analysis |
```

**Use this template when**:
- All acceptance criteria fully addressed
- All functional and technical requirements included
- No critical issues or high-risk assumptions identified
- No missing production requirements

**Do NOT use this template when**:
- Any acceptance criteria missing or incomplete
- Critical issues identified
- High-risk assumptions present
- Missing production requirements

In those cases, use the full template with all sections detailing specific gaps and issues.

## Section Specifications

### Metadata
**Purpose**: Track analysis iterations and relationships to other artifacts
**Content Requirements**:
- Unique ID following pattern US-###-CRITICAL-ANALYSIS
- Links to User Story and Implementation Plan
- Analysis iteration tracking (1st, 2nd)
- Confidence level based on available information

### Executive Summary
**Purpose**: Provide high-level overview of critical findings
**Content Requirements**:
- Focus on blockers and high-risk items
- Summarize impact on project success
- Highlight what must be fixed before development

### Technical Solution Guidance Compliance
**Purpose**: Ensure adherence to user-specified technical decisions
**Content Requirements**:
- Validate technology stack matches specifications
- Check architecture patterns compliance
- Verify budget constraints are met
- Flag any deviations with impact assessment

### Critical Issues
**Purpose**: Document blockers that prevent successful implementation
**Content Requirements**:
- Only include CRITICAL severity items
- Provide specific evidence from plan
- Include actionable recommendations
- Track resolution status

### High-Risk Assumptions
**Purpose**: Identify dangerous assumptions that could cause failure
**Content Requirements**:
- Focus on assumptions likely to be false
- Include reality checks based on research
- Provide validation methods
- Suggest fallback approaches

### Refinement Tracking
**Purpose**: Monitor improvement across analysis iterations
**Content Requirements**:
- Track which issues were addressed
- Identify remaining concerns
- Note any new issues discovered
- Provide overall assessment

## Compliance Rules

### Required Fields
- All metadata fields with proper formatting
- Executive summary with critical findings
- Technical solution guidance compliance check
- At least one critical issue or high-risk assumption
- Research references and validation
- Changelog entries for each iteration

### Content Quality Standards
- Evidence-based analysis with specific examples
- Actionable recommendations for each issue
- Clear severity levels (Critical/High/Medium/Low)
- Research-backed concerns with citations
- Tracking of refinement effectiveness

### File Management
- Location: Same folder as implementation plan
- Naming: `{story-id}-critical-analysis.md`
- Updates: Edit in-place, never create new versions
- Relationship: 1:1 with implementation plan

---

*This Critical Analysis artifact ensures implementation plans are thoroughly vetted for production readiness before development begins, preventing costly mistakes and rework.*