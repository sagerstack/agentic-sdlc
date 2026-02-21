# MVP Iteration Artifact

## Purpose & Overview

The MVP Iteration artifact defines specific MVP iteration strategies, success criteria, and epic themes for Business Analyst breakdown. Each MVP iteration serves as a focused validation experiment that advances product development through direct market feedback. The MVP artifact takes user-defined scope and validates against market research insights.

### Key Objectives
- **MVP Strategy**: Define specific MVP iteration objectives and validation goals
- **Epic Themes**: Provide structured epic themes for Business Analyst decomposition into user stories
- **Success Framework**: Establish clear success criteria and measurement approach
- **Learning Plan**: Define what the MVP will validate and how learning will be captured
- **Market Validation**: Test market research assumptions through focused prototype development

### Integration Points
- **Input from**: Market Research Report and user-defined scope requirements
- **Created by**: `/create-mvp` command with product-manager sub-agent
- **Feeds into**: `/create-epic` command for epic generation, then `/create-story` for user story breakdown
- **Structure Creation**: Creates MVP folder structure for epic and story organization

## MVP Iteration Template Structure

```markdown
# MVP{auto-iteration}: {goal}

## Metadata
| Field | Value |
|-------|-------|
| ID | MVP-{auto-iteration} |
| Title | {goal} |
| Product Name | [single-word-product-name] |
| Market Research Reference | MR-### |
| Created | YYYY-MM-DD HH:mm:ss |
| Status | Draft / Final |
| Status History | [Date: Status - Reason for change] |
| Last Updated | YYYY-MM-DD HH:mm:ss |
| Auto-Derived Iteration | {auto-iteration} (automatically determined) |

## Goal (User Input)
**Primary Objective**: [User defines the main goal and purpose of this MVP]
**Target Outcome**: [User specifies what success looks like]
**Validation Purpose**: [What assumptions or hypotheses this MVP will test]

## In Scope (User Input)
**Core Functionality**:
- [User defines specific features to include]
- [User specifies technical requirements]
- [User sets functional boundaries]

**Technical Requirements**:
- [User defines architecture/technology constraints]
- [User specifies performance requirements]
- [User sets integration requirements]

**User Experience Requirements**:
- [User defines UX scope and expectations]
- [User specifies interface requirements]
- [User sets accessibility/usability standards]

## Out of Scope (User Input)
**Explicitly Excluded**:
- [User clearly defines what won't be built]
- [User specifies deferred features]
- [User excludes advanced functionality]

**Future Considerations**:
- [Features considered for future MVPs]
- [Technical debt acceptable in this MVP]
- [Known limitations and workarounds]

## Epics

| Epic ID | Epic Name | User Value | Priority | Complexity | Dependencies |
|---------|-----------|------------|----------|------------|--------------|
| EP-### | [Epic 1] | [Value description] | P1 | High/Med/Low | [Dependencies] |
| EP-### | [Epic 2] | [Value description] | P2 | High/Med/Low | [Dependencies] |
| EP-### | [Epic 3] | [Value description] | P3 | High/Med/Low | [Dependencies] |

## Success Criteria

### Primary Success Metrics
- [Key measurable outcomes that define MVP success]
- [User behavior metrics to track]
- [Technical performance benchmarks]
- [Business validation criteria]

### Secondary Success Metrics
- [Additional metrics that indicate positive progress]
- [Quality indicators and user satisfaction measures]
- [Learning and validation outcomes]

### Failure Criteria
- [Conditions that would indicate MVP failure]
- [Minimum thresholds for continuation]
- [Red flags that require pivot or stop]

## Market Research Validation Framework

### Market Gap Analysis
- **Primary Market Gap**: [Which specific gap from market research this MVP addresses]
- **User Pain Point**: [Specific user problem being solved from market research]
- **Competitive Advantage**: [How MVP differentiates based on competitive analysis]
- **Market Timing**: [Why now is the right time based on market conditions]

### User Behavior Validation
- **Expected User Behavior**: [Patterns predicted by market research]
- **Usage Metrics**: [How users should interact with MVP based on research]
- **Adoption Timeline**: [Expected user adoption curve from market insights]
- **Behavior Benchmarks**: [Success thresholds based on market research]

### Market Opportunity Validation
- **Market Size Validation**: [Metrics to confirm addressable market assumptions]
- **Pricing Validation**: [User willingness to pay based on market research]
- **Segment Validation**: [Confirm target user segments from market research]
- **Geographic Validation**: [Market expansion opportunities to test]

### Competitive Performance Framework
- **Performance Benchmarks**: [How MVP should perform vs current solutions]
- **Feature Differentiation**: [Unique value props to validate from competitive analysis]
- **User Switching Validation**: [Testing user migration from competitors]
- **Market Response Monitoring**: [Tracking competitive responses]

### Risk Assessment
- **Market Risks**: [Risks identified in market research and mitigation]
- **Adoption Barriers**: [User adoption challenges and testing approach]
- **Competitive Response**: [Expected competitor reactions and monitoring]
- **Technical Risks**: [Market-related technical challenges to validate]

### Validation Hypotheses
**Market Research Predictions**:
- [What market research suggests should happen]
- [Key assumptions about user behavior]
- [Expected market response and adoption]

**MVP Test Results Framework**:
- [How to measure actual outcomes vs predictions]
- [Success criteria for market validation]
- [Learning objectives and data collection]

**Learning Objectives**:
- [Key market insights MVP should provide]
- [Assumptions to validate or invalidate]
- [Market intelligence to gather for next iteration]

## Open Questions for MVP

**Note**: These questions focus on functional requirements, user experience, market validation, and business model assumptions. Technical implementation details are addressed at the user story level with solution-architect guidance.

### User Experience Questions

| Question | User Response | Impact on Epic Generation |
|----------|---------------|---------------------------|
| What are the key UX assumptions that need validation? | [INSERT_USER_RESPONSE_HERE] | Creates UX validation epics |
| Are there interface design patterns to follow? | [INSERT_USER_RESPONSE_HERE] | Guides UI/UX epic design |
| What are the critical user workflows? | [INSERT_USER_RESPONSE_HERE] | Defines user journey epics |

### Market Questions

| Question | User Response | Impact on Epic Generation |
|----------|---------------|---------------------------|
| What market assumptions need validation through this MVP? | [INSERT_USER_RESPONSE_HERE] | Creates market validation epics |
| What user behavior patterns should be measured? | [INSERT_USER_RESPONSE_HERE] | Defines analytics epic requirements |
| How will competitive response be monitored? | [INSERT_USER_RESPONSE_HERE] | May create competitive monitoring epic |

### Business Model Questions

| Question | User Response | Impact on Epic Generation |
|----------|---------------|---------------------------|
| What pricing models will be tested? | [INSERT_USER_RESPONSE_HERE] | Creates monetization epic if applicable |
| What value propositions need validation? | [INSERT_USER_RESPONSE_HERE] | Guides feature prioritization in epics |
| What are the go-to-market considerations? | [INSERT_USER_RESPONSE_HERE] | May influence marketing/analytics epics |

### Resource Questions

| Question | User Response | Impact on Epic Generation |
|----------|---------------|---------------------------|
| What is the monthly budget for this MVP? | [INSERT_USER_RESPONSE_HERE] | Constrains epic scope and resource allocation |

## Next Steps
- [Immediate actions required to start MVP development]
- [Key decisions needed before proceeding]
- [Resource allocation and team assignments]

## Changelog
**Note**: Date format must include timestamp (YYYY-MM-DD HH:mm:ss) to match metadata Created field format

| Date | Author | Summary | Sections Affected | Reason |
|------|--------|---------|------------------|--------|
| YYYY-MM-DD HH:mm:ss | [Author] | [Change summary] | [Affected sections] | [Reason for change] |
```