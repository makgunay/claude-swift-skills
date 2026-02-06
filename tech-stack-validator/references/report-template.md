# Tech Stack Validation Report Template

Use this template when producing the final validation report. Adapt depth to project complexity.

## Template

```markdown
# Tech Stack Validation Report: [App Name]

**Date:** [Date]
**PRD Version:** [Reference]
**Architecture Version:** [Reference]
**Validator:** Claude (assisted by [user name])

---

## Verdict: [✅ GO | ⚠️ GO WITH CHANGES | ❌ STOP]

**Summary:** [2-3 sentences explaining the verdict. What's the bottom line?]

---

## 1. Requirements Summary

### Platform
- **Target:** [macOS X+ / iOS X+ / both]
- **Distribution:** [App Store / Direct / Both]
- **Minimum OS:** [Version] — gates available APIs
- **Device targets:** [Mac, iPhone, iPad, etc.]

### Key Technical Requirements
| Requirement | Source (PRD section) | Tech Implication |
|-------------|---------------------|-----------------|
| [Requirement] | [Section #] | [What this forces/constrains] |

---

## 2. Stack Assessment

### Proposed Stack
| Layer | Choice | Version/Min OS | Verdict |
|-------|--------|---------------|---------|
| Language | [e.g., Swift 6.2] | [Xcode 26] | ✅ / ⚠️ / ❌ |
| UI Framework | [e.g., SwiftUI] | [macOS 10.15+] | ✅ / ⚠️ / ❌ |
| UI Design | [e.g., Liquid Glass] | [macOS 26+] | ✅ / ⚠️ / ❌ |
| Persistence | [e.g., SwiftData] | [macOS 14+] | ✅ / ⚠️ / ❌ |
| Sync | [e.g., CloudKit] | [macOS 10.10+] | ✅ / ⚠️ / ❌ |
| Concurrency | [e.g., Swift Concurrency] | [macOS 10.15+] | ✅ / ⚠️ / ❌ |
| Testing | [e.g., Swift Testing] | [Xcode 16+] | ✅ / ⚠️ / ❌ |
| Distribution | [e.g., App Store + DMG] | — | ✅ / ⚠️ / ❌ |

### Per-Choice Analysis

#### [Technology Choice 1]
- **Requirement it serves:** [What PRD requirement drives this choice]
- **Availability:** ✅ Available on [target OS]
- **Capability:** [Can it deliver the specific requirement? At scale?]
- **Compatibility:** [Works with other stack choices? Sandbox OK?]
- **Maturity:** [Stable / Evolving / Experimental]
- **Risk:** [Low / Medium / High] — [explanation]
- **Verdict:** ✅ / ⚠️ [specific concern] / ❌ [blocker]

[Repeat for each major technology choice]

---

## 3. Critical Issues ❌

Issues that **must be resolved** before implementation begins.

### Issue 1: [Title]
- **What:** [Description of the mismatch]
- **Why it matters:** [Impact if not addressed]
- **PRD requirement:** [Which requirement is affected]
- **Recommended fix:** [Specific action]
- **Alternative:** [Backup option if fix isn't feasible]

[Repeat for each critical issue]

---

## 4. Warnings ⚠️

Issues that **should be addressed** but won't immediately block progress.

### Warning 1: [Title]
- **What:** [Description]
- **Risk:** [What could go wrong]
- **Recommended action:** [Specific suggestion]
- **Timeline:** [When this needs to be resolved by]

[Repeat for each warning]

---

## 5. Compatibility Matrix

### Requirements → Technology Fit

| PRD Requirement | Chosen Tech | Fit | Notes |
|----------------|-------------|-----|-------|
| [Req 1] | [Tech] | ✅/⚠️/❌ | [Details] |
| [Req 2] | [Tech] | ✅/⚠️/❌ | [Details] |

### Cross-Technology Compatibility

| Tech A | Tech B | Compatible? | Notes |
|--------|--------|------------|-------|
| [Tech] | [Tech] | ✅/⚠️/❌ | [Known issues or limitations] |

### Distribution Compatibility

| Feature | App Store (Sandbox) | Direct | Notes |
|---------|-------------------|--------|-------|
| [Feature] | ✅/⚠️/❌ | ✅/⚠️/❌ | [Entitlement needed?] |

---

## 6. Performance Feasibility

| PRD Budget | Chosen Tech | Feasible? | Evidence/Estimate |
|-----------|-------------|-----------|-------------------|
| Launch < [X]ms | [Stack] | ✅/⚠️ | [Benchmark or estimate] |
| Search < [X]ms @ [N] items | [Persistence] | ✅/⚠️ | [Based on known characteristics] |
| Memory < [X]MB | [Stack] | ✅/⚠️ | [Estimate] |

---

## 7. Recommendations

### Recommended Changes
| # | Change | Reason | Impact |
|---|--------|--------|--------|
| 1 | [Swap X for Y] | [Why] | [What it fixes] |
| 2 | [Add Z] | [Why] | [What it enables] |

### Architecture Decision Records

#### ADR-001: [Decision prompted by validation]
- **Context:** [What the validation revealed]
- **Decision:** [What should change]
- **Alternatives:** [What was considered]
- **Consequences:** [Trade-offs of this decision]

---

## 8. Pre-Implementation Checklist

Before writing code, confirm:

- [ ] All critical issues resolved
- [ ] Minimum OS version confirmed across all chosen frameworks
- [ ] Distribution method confirmed (sandbox entitlements identified if App Store)
- [ ] Permissions identified and fallback behavior designed
- [ ] Third-party dependencies evaluated (maintenance, license, size)
- [ ] Performance budgets achievable with chosen stack (benchmarked or estimated)
- [ ] Data model supports all PRD features
- [ ] Sync/conflict strategy defined
- [ ] Migration path exists for v1.0 → v1.1 data model changes
- [ ] Testing approach compatible with chosen frameworks

---

## Appendix: OS Version Impact Analysis

If the minimum OS were raised or lowered, what would change?

| Min OS Version | Gains | Loses |
|---------------|-------|-------|
| [Current - 1] | [Wider audience] | [Loses: framework X, API Y] |
| [Current] | [Current plan] | — |
| [Current + 1] | [Gains: framework Z] | [Smaller audience] |
```

## Report Depth Guide

| Project Type | Depth |
|---|---|
| Personal weekend project | Stack table + critical issues only (skip matrix, ADRs) |
| Solo side project shipping to users | Full report, lighter on ADRs |
| Startup MVP | Full report, include performance feasibility and ADRs |
| Team project | Full report at maximum depth |
