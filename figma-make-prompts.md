# CodeGuard — Figma Make Prompt Sequence

## Strategy (based on winning patterns)
- Start with SHORT prompts, iterate from there
- Import the dashboard SVG as your starting design file
- Keep a log of each prompt for your Devpost writeup
- Test after every 3-5 prompts

---

## Phase 1: Foundation (Prompts 1-5)

### Prompt 1 — Starting prompt
```
Build a dark-themed dashboard for a code review tool called CodeGuard. Sidebar on the left with navigation. Main area shows stat cards at the top and a PR review panel below.
```

### Prompt 2 — Structure the sidebar
```
Add sidebar navigation items: Dashboard, PR Reviews, Guidelines, Architecture, Signal Analysis, Intern View. Add two collection counters at the bottom: "Positive Patterns: 247" in green and "Negative Patterns: 89" in red.
```

### Prompt 3 — Add stat cards
```
Add 4 stat cards in a row at the top: "PRs Reviewed Today: 12 (+33%)", "Violations Caught: 7 (3 critical)", "New Guidelines Learned: 4 (2 pos / 2 neg)", "Knowledge Coverage: 73% (+8%)". Use dark cards with colored accents.
```

### Prompt 4 — PR Review panel
```
Add a main panel showing an active PR review. Show PR #347 "fix: move payment logic from route to service layer". Display 2 violation cards below it with red/yellow borders showing the guideline text and source commit info. Each violation has a dismiss button.
```

### Prompt 5 — Code diff preview
```
Add a small code diff preview at the bottom of the PR panel showing: the old line "except:" in red and the new line "except PaymentError as e:" in green, with a file path "routes/payment.py:45" above it.
```

---

## Phase 2: Right Panel (Prompts 6-10)

### Prompt 6 — Similar functions panel
```
Add a right sidebar panel titled "Similar Functions Found" showing 3 matched functions with cosine similarity scores: services/billing.py::process_refund (0.89), services/order.py::validate_order (0.82), routes/checkout.py::handle_checkout (0.71). Use monospace font for function names.
```

### Prompt 7 — Recently learned panel
```
Below the similar functions, add a "Recently Learned" panel showing 3 recent guidelines with colored dots: green for positive ("Use service layer for business logic"), red for negative ("Avoid print() in production"), green for positive ("Handle errors with specific exception types"). Show confidence and time.
```

### Prompt 8 — Activity feed
```
Add a full-width activity feed at the bottom with 4 items: LEARNED (green), FLAGGED (yellow), INDEXED (purple), DISMISSED (red). Each shows a one-line description and timestamp. Make it look like a real-time event log.
```

### Prompt 9 — Mode indicator
```
Add a mode indicator in the sidebar showing "Auto Mode Active" with a green dot. This indicates CodeGuard is using LLM + heuristic mode, not heuristic-only.
```

### Prompt 10 — User profile
```
Add a user profile card at the bottom of the sidebar showing initials "BM", name "Bhavika M.", role "Project Context Lead". Use a subtle purple accent.
```

---

## Phase 3: Interactivity (Prompts 11-20)

### Prompt 11 — Make dismiss buttons work
```
When I click a dismiss button on a violation, fade it out with a brief animation and show a toast notification saying "Violation dismissed — this pattern will be suppressed for similar functions."
```

### Prompt 12 — PR selector
```
Add a dropdown at the top of the PR panel that lets me switch between PR #347, #346, and #345. Each should show different violations.
```

### Prompt 13 — Sidebar navigation
```
Make the sidebar navigation clickable. When I click "PR Reviews", show a list view of recent PRs. When I click "Guidelines", show a searchable list of stored guidelines with positive/negative filters.
```

### Prompt 14 — Guideline detail view
```
When I click on a guideline in the "Recently Learned" panel, expand it to show the full guideline: pattern_summary, reasoning, diff_snippet, applies_to, commit_hash, confidence, and cold_start status.
```

### Prompt 15 — Search functionality
```
Make the search bar functional. When I type, filter guidelines by pattern_summary text. Show results in a dropdown below the search bar.
```

### Prompt 16 — Toggle between modes
```
Make the mode indicator clickable. When I click it, toggle between "Auto Mode" (green, LLM + heuristics) and "Heuristic Mode" (blue, rules only). Show a brief explanation of each mode.
```

### Prompt 17 — Animated stat counters
```
Add subtle count-up animations to the stat cards when the page loads. Numbers should animate from 0 to their final value over 1 second.
```

### Prompt 18 — Intern View
```
When I click "Intern View" in the sidebar, show a simplified view that hides technical details and shows friendly explanations of each violation, like "This code might make it harder for your teammates to debug problems later."
```

### Prompt 19 — Architecture tree view
```
When I click "Architecture" in the sidebar, show the CodeGuard system tree with nodes for Project Context, Commit Pipeline, Evaluate Pipeline, and Integration. Make nodes clickable to show their functions.
```

### Prompt 20 — Live simulation
```
Add a "Simulate PR Review" button that walks through the evaluate pipeline step by step: "Extracting AST diff..." → "Finding similar functions..." → "Checking guidelines..." → "2 violations found!" with animated progress indicators.
```

---

## Phase 4: Polish (Prompts 21-30)

### Prompt 21
```
Add smooth transitions between all sidebar views. Use slide-in animations.
```

### Prompt 22
```
Add a notification badge on "PR Reviews" showing the count of active reviews.
```

### Prompt 23
```
Add tooltips on the stat cards explaining what each metric means for the project.
```

### Prompt 24
```
Make the activity feed auto-scroll and add new simulated events every 10 seconds.
```

### Prompt 25
```
Add a "How CodeGuard Helps" onboarding overlay for first-time users that explains the two pipelines.
```

### Prompt 26
```
Add dark/light mode toggle in the top bar.
```

### Prompt 27
```
Add a mini chart in the "Knowledge Coverage" stat card showing the last 7 days trend.
```

### Prompt 28
```
Add keyboard shortcuts: Escape to close panels, / to focus search, n to cycle through violations.
```

### Prompt 29
```
Add responsive breakpoints so the dashboard works on tablet screens.
```

### Prompt 30
```
Final polish: check all animations are smooth, all text is readable, all interactive elements have hover states. Add a subtle gradient animation to the CodeGuard logo.
```

---

## Tips from Winners
1. **Revert often** — use the safety net when experiments fail (Sebastian)
2. **Test after each prompt** — click everything, check every state (Cara)
3. **Ask Make to organize code** — separate into components/files (Susanna)
4. **Use real data** — the mock PR data makes it feel alive (Matt)
5. **Keep prompt versions** — copy each prompt to a doc (Daniella & Max)

## Your Prompt Count Target
Winners used between 80-800 prompts. For a polished dashboard, aim for ~30-50 prompts. Quality > quantity.
