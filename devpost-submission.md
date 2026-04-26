# CodeGuard

## Inspiration

It's your first week as an engineering intern. You've been up since midnight writing your first real feature. Not a tutorial. Not a homework problem. Real code, for a real product, that real users will touch. You test it. It works. You open your first pull request and wait.

The review comes back. Three comments. None of them are about bugs.

*"We don't call the database from route handlers here."*
*"This needs the retry wrapper — check how the payments service does it."*
*"We stopped using this pattern in March. Look at the refactor in PR #281."*

Your code runs fine. It just doesn't belong here. And you had no way to know that. Nobody told you. It wasn't in the README. It wasn't in the style guide. It was in the senior engineer's head, and now it's in your PR comments, and you're rewriting something that worked perfectly at 2 AM on a Tuesday.

This happens to every intern. Every new hire. Every AI coding agent that can write beautiful code but has never seen this repo before. The problem is never syntax. It's context. And context lives in people, not documentation.

Two things clicked for us. Tree-sitter got mature enough to extract function-level semantic diffs, not just line changes. And embedding models got good enough to find "your team tried this six months ago and learned a better way" from a vector store. Neither is useful alone. Together, they let a repository remember how it wants to be changed — and tell the intern before the senior engineer has to.

We built CodeGuard so the next intern's first PR gets a review that sounds like the best mentor on the team. Instantly. Kindly. Before any human has to repeat themselves.

## What it does

You add a `.codeguard/intent.md` file to your repo. Plain English. What the system does, how it's deployed, what constraints matter, which parts of the codebase have different rules. That's the setup. Think of it as the document you wish someone had handed you on your first day.

When the intern opens a PR, CodeGuard reads the diff — not line by line like a linter, but function by function like a thoughtful reviewer. It parses the AST with tree-sitter, pulls relevant guidelines it learned from every PR the team ever merged out of MongoDB, and posts a review. Human-readable markdown the intern can learn from. Agent-readable JSON for automation. Takes seconds.

The output isn't "this may be risky." It's "this function makes an unretried external call in a hot path, and your team's deployment context says this service handles 10k requests per second — check how the payments module handles this." Specific. Grounded. The kind of feedback that teaches you something instead of just telling you you're wrong.

When a PR gets merged, CodeGuard flips to learning mode. It looks at what was approved, decides whether it teaches a reusable lesson, and stores it. Every merge the team makes teaches CodeGuard something new. So when the next intern opens their first PR, the review is even better than the last one. Knowledge compounds.

There's a bootstrap mode too. Point it at your last six months of commits and CodeGuard speed-learns your team's entire history in one run. That intern on Monday morning gets the benefit of six months of institutional knowledge before they write a single line.

## How we built it

We kept coming back to one question: what would make an intern's first week better?

The answer wasn't another linter. Linters catch semicolons. Interns don't struggle with semicolons. They struggle with "why did my PR get three comments about things I couldn't have known?" So we needed a system that understands what a team has learned and shares it at the right moment.

The pipeline has five moving parts.

**AST diff extraction.** Tree-sitter parses both the old and new versions of every changed Python file into full abstract syntax trees. We match functions across versions and build semantic diffs — not "line 47 changed" but "this function gained a parameter and restructured its error handling." When an intern refactors something, CodeGuard understands *what* they changed, not just *where*.

**Intent grounding.** `.codeguard/intent.md` captures what the team actually cares about — deployment context, hard constraints, section-by-section profiles. It's the document someone should have written for every intern who ever joined the team. Now it exists, and CodeGuard reads it on every review.

**Two-tier LLM reasoning.** OpenRouter handles all model calls. A fast model classifies diffs — the intern's typo fix gets a thumbs up in milliseconds. Their new API endpoint with three external calls and no error handling? That hits the deep model for real analysis. Both default to `openrouter/free`. Smart routing keeps it fast and affordable enough to run on every single PR.

**Persistent guideline memory.** MongoDB stores everything — guidelines, history, embeddings. Survives CI teardowns. The lesson the team learned from a production incident in January surfaces in a review of the intern's PR in July. sentence-transformers handle the semantic search. A stored guideline about connection pooling finds the intern's code that introduces raw database connections, even when the words don't overlap at all.

**Dual output rendering.** Every review produces a clean markdown file the intern can read and learn from, and a structured JSON file that automation can act on. One review, two audiences — the human who's learning and the system that's enforcing.

```
codeguard_monitor/
├── cli.py          # CLI entrypoints
├── intent/         # intent parsing & validation
├── ast_diff/       # tree-sitter function-level diffs
├── evaluate/       # PR review pipeline
├── commit/         # guideline extraction & persistence
├── db/             # MongoDB + embedding helpers
├── output/         # human & agent output rendering
└── overseer/       # hard-constraint alert logic
```

CI integration is one YAML file and two secrets. On pull_request, evaluate runs and posts. On push to main, commit runs and learns. The intern doesn't install anything. They just open a PR and the feedback is there.

## Challenges we ran into

**Function matching across versions.** Git gives you line diffs. An intern's reviewer thinks in functions. Parsing both file versions with tree-sitter, matching functions that moved or got renamed, handling splits and restructures — that was the hardest engineering problem. Also the most satisfying to solve. It's the reason CodeGuard's feedback reads like a mentor, not a diff bot.

**Guideline conflicts.** Teams evolve. The pattern the team used six months ago might be the anti-pattern they avoid today. If CodeGuard gave the intern outdated advice, that would be worse than no advice. We built conflict detection using embedding similarity — when new knowledge contradicts old knowledge, CodeGuard resolves it before it causes a confusing review.

**The cold start problem.** CodeGuard learns from merges. But the intern starts on Monday and there are no guidelines yet. The bootstrap pipeline replays historical commits, and making it handle every edge case — root commits, force-pushed branches, non-linear merges — took more work than expected. Worth it. That Monday morning intern gets six months of context on their first PR.

**Keeping it cheap enough for every PR.** If the intern's 20 PRs in their first week each cost real money, teams won't keep CodeGuard on. The two-tier model routing was our answer — most diffs get classified in milliseconds. Only structurally interesting changes get the full deep review. That made the economics work for the use case that matters most: high-volume contributors who are still learning.

## Accomplishments that we're proud of

**It learned a team's voice.** We pointed CodeGuard at a real repo's history, ran bootstrap, opened a test PR with a known anti-pattern. The feedback CodeGuard gave was almost word-for-word what the senior engineer on that team would have said. It had learned their voice. An intern reading that review would learn the same lesson — without the senior engineer having to stop what they're doing to write it.

**Honest about what it knows.** When CodeGuard skips deep review because a diff has no function-level Python changes, it says so. The output explicitly labels "skipped — no semantic changes detected." We didn't want an intern trusting a review that wasn't actually thorough. If CodeGuard doesn't have something meaningful to say, it stays quiet. That felt more important than faking confidence.

**Five-minute setup.** One YAML file. Two secrets. No config sprawl, no onboarding doc, no "ask DevOps to set this up." We wanted the barrier between "the intern starts Monday" and "CodeGuard is reviewing their PRs" to be five minutes. It is.

**The learning loop works.** This is the part we're most proud of. CodeGuard doesn't just review — it learns. Every PR the team merges becomes future review memory. The intern in January gets good reviews. The intern in July gets better ones. The knowledge base grows without anyone maintaining it.

**Merge blocking that teaches.** When a hard constraint from intent.md is violated, CodeGuard doesn't just block with a red banner. It explains: here's the constraint, here's why the team defined it, here's what your code did. The intern learns the rule *and* the reason. That's mentorship, not just enforcement.

## What we learned

**Interns don't fail because of syntax. They fail because of context.** The gap between "this code runs" and "this code belongs here" is entirely contextual. Line-level tools can't close it. Function-level semantic understanding, grounded in the team's actual history, can.

**The best feedback arrives before you ask for it.** When CodeGuard catches a pattern on the intern's PR and explains the better approach with a pointer to where the team does it right, that's a learning moment that happens in flow — not after a painful review cycle. The intern never has to feel like they messed up. They just see "hey, here's how this team handles this" and they learn.

**Teams are more consistent than they realize.** CodeGuard surfaced patterns that even senior engineers didn't know they were enforcing. Turns out the feedback they give interns isn't random — it's deeply consistent. They just never had a tool that made it visible. Now they do.

**Embeddings find what keywords miss.** A guideline about retry logic for external services matches the intern's code that adds `requests.get()` in a loop, even though the word "retry" never appears anywhere. That kind of semantic connection is what makes retrieval-based mentorship actually work.

**Foundation models are the reasoning layer, not the product.** Claude is brilliant. But Claude alone doesn't remember that your team stopped using a pattern in March. CodeGuard is the repository memory, the workflow integration, the system fit — around that intelligence. That's the architecture insight that made everything work.

## Competitive Positioning

We have not found a mainstream tool that combines deployment-intent grounding, commit-time guideline learning, and persistent repo-specific retrieval in one workflow.

| Feature | CodeGuard | CodeRabbit | Greptile | Qodo | Graphite |
|---------|-----------|------------|----------|------|----------|
| Deployment intent file | **Yes** | No | No | No | No |
| Commit-time guideline extraction | **Yes** | No | No | No | No |
| Persistent guideline vector store | **Yes** | Yes | No | Yes | No |
| Function-level AST diff | **Yes** | No | Partial | Yes | No |
| Hard constraint / merge blocking | **Yes** | Partial | No | Yes | No |
| Section profiles / per-directory scope | **Yes** | Partial | No | No | No |
| Agent-readable JSON output | **Yes** | No | No | No | No |
| Bootstrap historical commits | **Yes** | No | No | No | No |

Most review bots tell you whether code looks good in general. CodeGuard tells the intern whether their code fits *this repository*, *this deployment context*, and *this team's learned constraints*.

Claude gives intelligence. CodeGuard gives that intelligence repository memory, deployment context, and the workflow behavior that makes it useful at 2 AM when the intern is coding and no senior engineer is awake.

## What's next for CodeGuard

Multi-language support. The intern writing JavaScript deserves the same mentorship as the one writing Python. Tree-sitter already handles both.

Confidence scoring on guidelines. The intern should see whether a guideline was reinforced by fifty merged PRs or just one. Trust is earned.

Team analytics. Which guidelines fire most. Which patterns are evolving. A dashboard that shows the team's collective wisdom growing over time.

IDE integration. CodeGuard feedback in VS Code before the PR is even opened. The intern learns at the keyboard, not in the review thread.

Auto-fix patches. Beyond flagging — generate the code change that follows the team's pattern. One click and the intern's code matches how the team does it.

Cross-repo learning. What one team learns should help the next team's intern too. Shared guideline libraries across an organization.

The horizon is simple. Every intern, every new engineer, every AI agent contributing to a codebase should write code that fits the system from their first PR. Not because they studied for weeks. Because the repository told them what it needed, gently, before anyone had to say it twice.

## Built With

- python
- tree-sitter
- mongodb
- openrouter
- sentence-transformers
- github-actions
- claude
- devin
- figma
