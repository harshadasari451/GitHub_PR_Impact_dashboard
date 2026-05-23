# PostHog Engineer Impact Dashboard

An interactive dashboard that identifies the most impactful engineers in the [PostHog/posthog](https://github.com/PostHog/posthog) GitHub repository over the last 90 days.

## Approach

### Data Collection
`fetch_and_score.py` pulls data from the GitHub API — specifically all merged PRs from the last 90 days, plus the review activity on each PR. No third-party libraries required (stdlib only).

### Defining Impact
Rather than ranking by raw commit counts or lines of code, impact is measured across four dimensions:

| Dimension | Weight | Rationale |
|---|---|---|
| PRs Merged | 35% | Core authoring output |
| Reviews Given | 35% | Approvals + change requests on others' PRs — a proxy for team leverage and unblocking others |
| Code Volume | 15% | Lines added + deleted, signals scope of work |
| Consistency | 15% | Active weeks out of 13 — rewards steady shipping over one-time bursts |

Each dimension is min-max normalized across all contributors, then combined into a weighted composite score (0–100).

The decision to weight reviews equally to PRs merged reflects a deliberate choice: an engineer who reviews 20 PRs a week may create more team-wide leverage than one who merges 20 PRs in isolation.

### Dashboard
A fully static `index.html` — no backend, no build step. It reads a pre-generated `data.json` file, keeping load times well under 1 second. Hosted via GitHub Pages.

## Limitations
GitHub's API surfaces activity signals, not outcomes. What's missing: whether a PR fixed a production incident, whether a review caught a critical bug, or whether an engineer mentored others through their PRs. The scores are a reasonable proxy within these constraints, not ground truth.
