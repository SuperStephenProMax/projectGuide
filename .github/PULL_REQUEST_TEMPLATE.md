## Summary
<!-- What & Why. Keep it short but specific. -->

## Related Issue
<!-- Use Fixes to auto-close, or Refs to link. -->
- Fixes #
- Refs #

## Type of Change
- [ ] Bugfix (`bugfix/*`)
- [ ] Feature (`feature/*`)
- [ ] Hotfix (`hotfix/*`)
- [ ] Refactor
- [ ] Docs
- [ ] CI/Build/Chore

## What Changed
-

## How to Test
### Local
```bash
mvn -B -ntp test
```

### Coverage (optional local verification)
```bash
mvn -B -ntp clean \
  org.jacoco:jacoco-maven-plugin:0.8.12:prepare-agent test \
  org.jacoco:jacoco-maven-plugin:0.8.12:report
```

## Evidence
- [ ] Logs attached
- [ ] Screenshot attached (if UI)
- [ ] Repro steps included (bugfix)

## Risk & Rollback Plan

**Risk level**: low / medium / high

**Rollback plan** (be specific, include commands if possible):
- Example: revert squash commit on `main`

## Patch Coverage Gate (Required)
- Target: **>= 70%** on changed lines

## Coverage Override (ONLY if you request/approve override)

> ⚠️ Override can ONLY bypass patch coverage gate. UT must still pass.

**Override Coverage Reason:**

<!-- Required if label `ci-override-coverage` is applied -->

**Override Coverage Expiry (YYYY-MM-DD, <= 14 days):**

**Override Approver (@github-handle):**

## Copilot Code Review
- [ ] Request Copilot review (add "Copilot" as reviewer, or comment `@github-copilot review`)

## Checklist (Developer)
- [ ] Issue linked (Fixes/Refs)
- [ ] UT passed locally
- [ ] Added/updated unit tests for new/changed behavior
- [ ] No secrets / tokens / credentials in code or logs
- [ ] Backward compatibility considered
- [ ] Docs updated if needed
