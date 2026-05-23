# PostHog Engineer Impact Dashboard

An interactive dashboard that identifies the most impactful engineers in the [PostHog/posthog](https://github.com/PostHog/posthog) GitHub repository over the last 90 days.

## Approach

### Data Collection

`fetch_and_score.py` pulls from three bulk GitHub API endpoints — no per-PR calls:

| Endpoint | What it gives us |
|---|---|
| `GET /repos/{repo}/pulls` | All merged PRs in the window |
| `GET /repos/{repo}/pulls/comments` | Every inline review comment across all PRs |
| `GET /repos/{repo}/issues/events` | PR lifecycle events (reviews, closures) |

~20–30 total API calls regardless of PR count. Stays well within GitHub's 5000/hr rate limit even for a high-volume repo like PostHog.

### Defining Impact

At a fast-moving, open-source company like PostHog, impact means two things: shipping reliably yourself, and raising the team's floor by unblocking others. An engineer who merges 10 PRs and reviews 40 more — especially stale ones nobody else touches — creates more total output than one who merges 30 PRs in isolation. This is the core premise behind the scoring model.

Five dimensions, each min-max normalized to 0–100 across all contributors, then combined into a weighted composite score:

| Dimension | Weight | How it's computed | Why |
|---|---|---|---|
| **Collaboration Centrality** | 35% | Unique authors you reviewed (×1.5) + unique people who reviewed you | Hub reviewers create more team leverage than leaf nodes |
| **Review Influence** | 30% | % of your review comments where more activity followed on the PR | Measures whether your feedback actually changed things, not just that you showed up |
| **Bottleneck Resolution** | 20% | `log(median PR age at first comment) × log(PRs reviewed)` | Rewards picking up old, stale PRs that nobody else is touching |
| **Code Quality** | 10% | PRs authored × (1 − review_rounds / 3) | Fewer revision cycles = cleaner first submissions |
| **Consistency** | 5% | Unique active days in the 90-day window | Steady contribution over the full period, not a single burst |

### Dashboard

Fully static `index.html` — no backend, no build step. Reads a pre-generated `data.json`, keeping load times under 1 second. Hosted via GitHub Pages.

Features a radar chart comparing the five dimensions across the top 5 engineers, a full leaderboard, and a per-engineer score breakdown.

## Limitations

These are proxies, not ground truth. A few known approximations:

- **Review influence** uses "comment not being the last on the PR" as a signal that feedback triggered further action. A late-joining reviewer would score poorly even if their comment was the most important one.
- **Bottleneck resolution** can't distinguish "I rescued a stale PR" from "I was slow to review" — both look the same in the data.
- **Code quality** approximates revision rounds via unique reviewer count, since bulk endpoints don't expose `CHANGES_REQUESTED` events without per-PR calls.

GitHub data surfaces activity signals, not outcomes. What's invisible: whether a PR fixed a production incident, whether a review caught a critical bug, or whether an engineer unblocked a teammate through Slack rather than GitHub comments. The scores are a reasonable signal within these constraints.
