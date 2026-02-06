# PRD Template

Use this template when generating the final PRD document. Adapt sections based on project complexity — a menu bar utility doesn't need the same depth as a full productivity suite.

## Template

```markdown
# Product Requirements Document: [App Name]

**Version:** [x.y]
**Last Updated:** [Date]
**Author:** [Name]
**Status:** Draft | Review | Approved

---

## 1. Executive Summary

[2-3 paragraphs: what the product is, who it's for, why it matters, and what success looks like. A busy executive should understand the product after reading only this section.]

## 2. Problem Statement

### The Problem
[Describe the pain point in the user's words. Be specific — "managing prompts is hard" is weak; "knowledge workers who use multiple AI assistants daily lose 15+ minutes per session hunting for, reformatting, and adapting prompts scattered across notes apps, browser bookmarks, and clipboard history" is strong.]

### Current Workarounds
[How do users solve this today? What's painful about each workaround?]

### Market Context
[Brief market sizing or validation. Optional for personal projects.]

## 3. Goals & Success Metrics

### Product Goals
| Goal | Metric | Target | Timeframe |
|------|--------|--------|-----------|
| [Goal 1] | [Measurable metric] | [Target value] | [By when] |

### Non-Goals (v1.0)
[Explicitly state what this version will NOT do. This is as important as what it will do.]
- ❌ [Non-goal 1]: [Why it's excluded]
- ❌ [Non-goal 2]: [Why it's excluded]

## 4. User Personas

### Primary Persona: [Name]
- **Who:** [Demographics, role, tech comfort]
- **Context:** [When/where they use the app]
- **Jobs to be done:**
  1. [Job 1]
  2. [Job 2]
- **Frustrations:** [Current pain points]
- **Success:** [What "done well" looks like for them]

### Secondary Persona: [Name] (if applicable)
[Same structure]

## 5. User Stories

### Epic: [Feature Area]

| ID | Story | Priority | Acceptance Criteria |
|----|-------|----------|-------------------|
| US-001 | As a [persona], I want to [action] so that [benefit] | Must | Given [context], When [action], Then [outcome] |
| US-002 | ... | Should | ... |

[Repeat for each epic/feature area]

## 6. Functional Requirements

### FR-001: [Feature Name]

**Description:** [What it does]

**Behavior:**
- [Specific behavior 1]
- [Specific behavior 2]

**States:**
| State | Appearance | Behavior |
|-------|-----------|----------|
| Empty | [Description] | [What happens] |
| Loading | [Description] | [What happens] |
| Populated | [Description] | [What happens] |
| Error | [Description] | [What happens] |

**Edge Cases:**
- [Edge case 1]: [How it's handled]

[Repeat for each feature]

## 7. Non-Functional Requirements

### Performance
| Metric | Requirement |
|--------|-------------|
| Cold launch | < [X]ms |
| Search response | < [X]ms for [N] items |
| Memory footprint | < [X]MB idle |
| Storage per [unit] | ~[X]KB |

### Accessibility
- Full keyboard navigation for all features
- VoiceOver support for all interactive elements
- Respect system Reduce Motion / Reduce Transparency settings
- Minimum contrast ratios per WCAG 2.1 AA

### Security & Privacy
- [Data encryption approach]
- [Network communication policy]
- [Privacy manifest requirements]

### Reliability
- [Crash-free rate target]
- [Data loss prevention approach]

## 8. UI/UX Specifications

### Information Architecture
[Navigation structure, screen hierarchy]

### Key Screens

#### Screen: [Name]
- **Purpose:** [What the user accomplishes here]
- **Layout:** [Description or wireframe reference]
- **Key interactions:** [Primary actions available]
- **Transitions:** [How user arrives and leaves]

### Keyboard Shortcuts
| Shortcut | Action |
|----------|--------|
| ⌘+[Key] | [Action] |

## 9. Data Model

### Entity: [Name]
| Property | Type | Required | Notes |
|----------|------|----------|-------|
| id | UUID | Yes | Primary key |
| [prop] | [type] | [Yes/No] | [Notes] |

### Relationships
[Entity A] → [Entity B]: [Relationship type, delete rule]

### Sync Strategy
[iCloud/CloudKit/None, conflict resolution approach]

## 10. Error Handling

| Error Scenario | User Impact | Handling |
|----------------|-------------|----------|
| [Scenario] | [What user experiences] | [Recovery approach] |

## 11. Dependencies & Risks

### Dependencies
| Dependency | Type | Risk | Mitigation |
|-----------|------|------|------------|
| [Dep] | [Framework/API/Service] | [What could go wrong] | [Backup plan] |

### Risks
| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| [Risk] | Low/Med/High | Low/Med/High | [Plan] |

## 12. Implementation Phases

### Phase 1: Foundation (Weeks 1-N)
- [ ] [Milestone 1]
- [ ] [Milestone 2]

### Phase 2: Core Features (Weeks N-M)
- [ ] [Milestone 3]

### Phase 3: Polish & Launch (Weeks M-L)
- [ ] [Milestone 4]

## 13. Open Questions

| # | Question | Owner | Deadline | Resolution |
|---|----------|-------|----------|------------|
| 1 | [Question] | [Who decides] | [By when] | [Pending/Resolved: answer] |

## 14. Glossary

| Term | Definition |
|------|-----------|
| [Term] | [Definition as used in this document] |
```

## Section Scaling Guide

Not every project needs every section at full depth:

| Project Size | Skip/Minimize | Expand |
|---|---|---|
| Weekend hack | Personas, Market, Phases | Core features, Data model |
| Solo side project | Market sizing, Formal metrics | User stories, Architecture, Edge cases |
| Startup MVP | None — investors want completeness | Market, Metrics, Risks, Phases |
| Team product | None | All sections at full depth |
