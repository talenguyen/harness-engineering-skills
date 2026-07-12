# Field Returns — escaped defects, upstream fixes, and metrics

Station 5 log. One entry per defect that escaped past review or into production; name the
station that should have caught it and the upstream check you added.

## Metrics
Reviewed at 30 / 60 / 90 days. Hook-bypass target is zero.

| Date | Plan-rejection rate | Escaped defects (S4 / S5) | Hook bypasses |
|------|---------------------|---------------------------|---------------|
|      |                     |                           |               |

## Entries

### <date> — <short defect title>
- Impact: <what broke, who saw it>
- Responsible station: <1 Work Order | 2 The Part | 3 Andon Cord | 4 Inspection>
- Why it escaped: <one sentence>
- Upstream check added: <the test / rule / criterion, with file path>
- Immediate fix: <link / commit>

## Debt (skipped gates, logged so they stay visible)
- <gate skipped, when, why>
