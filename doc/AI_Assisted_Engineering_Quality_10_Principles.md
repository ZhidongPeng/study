# Ten Principles for AI-Assisted Engineering Quality

**A practical summary with software engineering examples**

## Executive Summary

The goal is not to make AI generate more code. It is to build an engineering workflow where AI delivers reliable results with less human effort and lower risk.

The ten principles emphasize accurate context, inspection of the current system, independent verification, reusable workflows, objective evidence, clear handoffs, human accountability, curated memory, and outcome-based measurement.

---

## 1. Provide Relevant Context

**Principle:** Provide sufficient *relevant* context without overwhelming AI with unrelated information.

**Why it matters:** Too little context encourages assumptions; too much introduces noise, conflicting details, and stale information.

**Example — investigating a memory leak**

- **Weak request:** “Analyze the entire repository and fix all memory leaks.”
- **Better request:** “Investigate the memory leak in the connection manager. Focus on connection lifecycle, cleanup paths, relevant tests, and the supplied logs. Identify the root cause before changing code.”

**Practical checklist:**
- Supply relevant source files and interfaces.
- Include applicable design decisions, logs, and tests.
- State expected behavior and constraints.
- Exclude unrelated modules and obsolete documentation.

> **Takeaway:** Relevant context beats maximum context.

## 2. Inspect Before Changing

**Principle:** Understand the current implementation before proposing or making changes.

**Why it matters:** A plausible plan based on a ticket alone may conflict with the code that exists today.

**Example — adding HTTP retries**

- **Weak request:** “Implement exponential backoff for failed requests.”
- **Better request:** “Inspect the existing HTTP client, retry policy, configuration, tests, and recent changes. Determine whether retry support already exists, then propose the smallest compatible change.”

**Workflow:** Inspect code → verify current behavior → find existing abstractions → propose a change → implement and test.

> **Takeaway:** Understand before changing.

## 3. Verify Independently

**Principle:** The agent writing code should not be the only agent deciding whether it is correct.

**Why it matters:** A model may overlook the same assumptions or mistakes in both its implementation and its self-review.

**Example — AI-assisted pull requests**

| Role | Responsibility |
|---|---|
| AI developer | Implement the change and document decisions |
| Independent AI reviewer | Inspect the diff against requirements and repository conventions |
| AI test agent | Design and run validation, including failure paths |
| Human engineer | Assess consequential risks and approve the change |

For genuine independence, reviewers should examine requirements and the actual diff rather than relying solely on the author's summary.

> **Takeaway:** Separate the creator from the verifier.

## 4. Reuse Established Patterns

**Principle:** Prefer established architecture and coding conventions over novel solutions without a clear reason.

**Why it matters:** Standalone code can appear elegant while creating duplicate abstractions, inconsistent error handling, or maintenance costs.

**Example — adding a configuration parameter**

- **Weak approach:** Introduce a new configuration parser and caching layer.
- **Better approach:** Locate the existing configuration manager, defaults, validation, and tests; extend those mechanisms.

**Useful AI instruction:** “Find at least one similar implementation in the repository and explain whether the new change follows it. Justify any departure.”

> **Takeaway:** Consistency is often more valuable than novelty.

## 5. Standardize Repeated Work

**Principle:** Convert recurring tasks and lessons into reusable workflows with clear inputs, checks, and outputs.

**Why it matters:** Repeating prompts wastes time and leads to inconsistent quality. Recurring AI mistakes should become checks or tests.

**Example — PR review skill**

**Inputs:** PR description, diff, relevant architecture, coding standards, and test results.

**Procedure:**
1. Identify intended behavior and changed surfaces.
2. Check correctness, concurrency, resource management, and security.
3. Review tests and edge cases.
4. Report actionable findings with file locations and evidence.

**Output:** Findings by severity, supporting evidence, suggested fixes, and unresolved risks.

If reviews repeatedly miss resource leaks, add a required check for cleanup on success, error, timeout, and cancellation paths.

> **Takeaway:** Turn repeated successes and failures into reusable team knowledge.

## 6. Require Verifiable Evidence

**Principle:** Require verifiable evidence that matches the claimed outcome.

**Why it matters:** “Looks correct” and “production-ready” are assertions, not proof.

**Example — fixing a race condition**

Ask for the root cause, synchronization change, regression test, actual test output, and remaining concurrency risks.

| Claim | Appropriate evidence |
|---|---|
| Code compiles | Actual successful build output |
| Bug is fixed | Reproducing regression test that now passes |
| Feature behaves as requested | Tests mapped to acceptance criteria |
| Performance is acceptable | Before-and-after measurements under comparable conditions |
| Security risks were assessed | Relevant analysis, scan results, and reviewed findings |

No single test proves universal correctness.

> **Takeaway:** Trust inspectable evidence, not AI confidence.

## 7. Clarify Every Handoff

**Principle:** Every agent-to-agent or agent-to-human transition should state what is done, why, what was verified, and what remains.

**Why it matters:** Ambiguous handoffs cause repeated work, lost assumptions, and missed risks.

**Example — developer agent to reviewer agent**

- **Completed:** Added retry support using the existing retry-policy abstraction.
- **Decisions:** Retry transient failures only; preserve timeout limits.
- **Evidence:** Build and unit tests passed; timeout scenarios exercised.
- **Risks:** Remote-service integration testing remains pending.
- **Reviewer focus:** Retry amplification, cancellation, and cleanup.

**Reusable handoff template:** Completed / Decisions / Evidence / Risks / Next steps.

> **Takeaway:** A good handoff makes the next participant immediately productive.

## 8. Require Human Approval for Consequential Actions

**Principle:** AI may prepare and recommend consequential actions, but humans retain final approval.

**Why it matters:** Merging, publishing, changing production, and modifying security-sensitive settings can have broad impact.

**Example — production deployment**

AI prepares the deployment, validates configuration, runs checks, summarizes risk, and recommends a decision. A human reviews the evidence and authorizes the deployment.

| Action | Suggested control |
|---|---|
| Generate code or run local tests | AI may act autonomously within approved boundaries |
| Prepare a PR | AI may prepare; repository permissions still apply |
| Merge a PR | Human approval |
| Deploy to production | Human approval |
| Modify security-sensitive configuration | Human approval |

> **Takeaway:** Automate preparation and execution where safe; retain human accountability at consequential gates.

## 9. Curate Durable Knowledge

**Principle:** Store only durable, correct, useful, and verifiable knowledge in long-lived AI memory.

**Why it matters:** Automatically accumulating every conversation and debugging detail produces stale or contradictory instructions.

**Example — repository knowledge**

**Keep:** Architectural rules, stable design decisions, coding standards, known constraints, and validated workflows.

**Do not automatically keep:** Temporary logs, one-off hypotheses, outdated implementation notes, and entire chat transcripts.

| Information | Appropriate home |
|---|---|
| Coding standards | Versioned repository instructions |
| Architectural decisions | Design or decision records |
| Repeated procedures | Reusable agent skills |
| Debugging details for one issue | Task-specific context |
| Historical discussions | Searchable records |

> **Takeaway:** Curate AI memory like a maintained engineering asset.

## 10. Measure Engineering Outcomes

**Principle:** Judge AI by improvements in delivered engineering outcomes, not by prompts, tokens, or generated lines.

**Why it matters:** Faster coding can shift work into testing, review, and rework.

**Illustrative example — end-to-end PR effort**

| Activity | Before AI | With AI |
|---|---:|---:|
| Implementation | 8 hours | 3 hours |
| Testing | 2 hours | 3 hours |
| PR review | 2 hours | 5 hours |
| Bug fixing | 1 hour | 3 hours |
| **Total** | **13 hours** | **14 hours** |

*These are hypothetical values, not study results.*

**Better measures:**
- **Productivity:** End-to-end completion time, review turnaround, human hours per completed task.
- **Quality:** Escaped defects, regressions, build/test failures, and rework.
- **Efficiency:** Cost per successfully completed task and verified human effort saved.

> **Takeaway:** AI activity is not engineering impact.

---

## How the Principles Fit Together

| Discipline | Principles | Objective |
|---|---|---|
| Context and understanding | 1, 2, 4 | Work on the right problem using the current architecture |
| Execution and verification | 3, 6, 8 | Validate independently and preserve approval controls |
| Workflow standardization | 5, 7, 9 | Make good workflows repeatable and maintainable |
| Outcome measurement | 10 | Demonstrate real productivity and quality improvements |

## Example: An AI-Native PR Workflow

1. **Understand:** Gather task-sized context, inspect current code, and find established patterns.
2. **Implement:** Use a standardized coding workflow, add tests, and record decisions.
3. **Verify:** Use independent review and execute relevant tests; attach evidence.
4. **Handoff:** Produce a structured summary of changes, results, and unresolved risks.
5. **Approve:** A human evaluates consequential findings and approves the merge.
6. **Improve:** Turn recurring defects into checks and measure outcomes.

**Loop:** Understand → Implement → Verify → Review → Deliver → Learn.

## Recommended Priorities for an AI PR Guardian

| Priority | Principle | Value |
|---|---|---|
| 1 | **#3 — Verify Independently** | Independent review instead of self-certification |
| 2 | **#6 — Require Verifiable Evidence** | Findings can be checked and acted on |
| 3 | **#5 — Standardize Repeated Work** | Consistent review quality across PRs |
| 4 | **#7 — Clarify Every Handoff** | Less human effort reconstructing changes |
| 5 | **#10 — Measure Engineering Outcomes** | Proof of productivity gains without hidden quality costs |

### Suggested Review-Ready Evidence Package

Each AI-generated PR should ideally include:

- Intended behavior and change summary.
- Relevant architectural context.
- Tests added and **actual** execution results.
- Independent AI review findings, with evidence.
- Known limitations and unresolved risks.
- Areas requiring human judgment.

This allows reviewers to focus attention on meaningful risks rather than reconstructing every decision from scratch. It does **not** remove the need for human code inspection where warranted.

## Final Takeaway

> **AI should not simply accelerate code generation. It should accelerate delivery of reliable software by reducing total human effort, automating repeatable work, and providing verifiable evidence of quality.**

---

*Based on the linked OCM article; practical examples, tables, and AI PR Guardian recommendations were added for general engineering use.*
