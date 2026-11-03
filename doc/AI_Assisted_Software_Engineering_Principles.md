# Nine Principles for AI-Assisted Software Engineering

**A practical summary with software engineering examples**

## Executive Summary

The goal is not to make AI generate more code. It is to build an engineering workflow where AI delivers reliable results with less human effort and lower risk.

The nine principles emphasize accurate context, inspection of the current system, reuse and maintenance of proven knowledge, reusable workflows, independent verification, objective evidence, clear handoffs, human accountability, and outcome-based measurement.

**Repository examples:** The examples below reference [Azure/GuestProxyAgent at commit `9a27720`](https://github.com/Azure/GuestProxyAgent/tree/9a27720438a282876a40e36ca8bd1f25e28a135f), inspected on November 3, 2026. They distinguish implementation from configured workflows and documented expectations. The repository's AI engineering guides are owner-review-pending drafts, not proof of deployed automation, executed validation, or measured productivity gains.

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

**GuestProxyAgent example — select context by change surface**

The [engineering context index](https://github.com/Azure/GuestProxyAgent/blob/9a27720438a282876a40e36ca8bd1f25e28a135f/.ai/README.md) maps topics to when they are relevant, while the [context registry](https://github.com/Azure/GuestProxyAgent/blob/9a27720438a282876a40e36ca8bd1f25e28a135f/.ai/context.json) links each topic to source evidence. For an authorization change, supply the relevant security and compatibility guidance plus the affected authorizer and tests, rather than loading every design plan and deployment script. The registry is navigation data, not an automatic retriever.

> **Takeaway:** Relevant context beats maximum context.

## 2. Inspect Before Changing

**Principle:** Understand the current implementation before proposing or making changes.

**Why it matters:** A plausible plan based on a ticket alone may conflict with the code that exists today.

**Example — adding HTTP retries**

- **Weak request:** “Implement exponential backoff for failed requests.”
- **Better request:** “Inspect the existing HTTP client, retry policy, configuration, tests, and recent changes. Determine whether retry support already exists, then propose the smallest compatible change.”

**Workflow:** Inspect code → verify current behavior → find existing abstractions → propose a change → implement and test.

**GuestProxyAgent example — inspect signing exceptions before refactoring HTTP code**

The shared HTTP client's [signature-skip implementation and tests](https://github.com/Azure/GuestProxyAgent/blob/9a27720438a282876a40e36ca8bd1f25e28a135f/proxy_agent_shared/src/hyper_client.rs) deliberately exempt `PUT /vmAgentLog` and `POST /machine/?comp=telemetrydata`. Its `should_skip_sig_test` also asserts that `GET` requests to those paths are not exempt. Before simplifying signing logic, inspect both the rationale and these method-specific assertions; do not assume every unsigned request is an accidental omission. These are inspected source contracts, not a claim that the tests were run for this document.

> **Takeaway:** Understand before changing.

## 3. Reuse and Maintain Proven Knowledge

**Principle:** Build on established architecture and coding conventions rather than introducing novel solutions without a clear reason. Store only durable, correct, useful, and verifiable knowledge in long-lived AI memory. Justify departures and update guidance when the system changes.

**Scope:** What AI should know and reuse: existing implementations, durable facts, design decisions, and constraints.

**Why it matters:** Ignoring existing patterns creates duplicate abstractions, inconsistent error handling, and maintenance costs. Automatically accumulating every conversation and debugging detail produces stale or contradictory instructions that cause AI to repeat mistakes.

**Example — adding a configuration parameter**

- **Weak approach:** Introduce a new configuration parser and caching layer.
- **Better approach:** Locate the existing configuration manager, defaults, validation, and tests; extend those mechanisms.

**Useful AI instruction:** “Find at least one similar implementation in the repository and explain whether the new change follows it. Justify any departure.”

**Example — maintaining repository knowledge**

After extending the existing configuration manager, preserve any validated, durable configuration rules so future changes reuse the same approach. Keep one-off debugging details out of long-lived AI memory.

**Keep:** Architectural rules, stable design decisions, coding standards, and known constraints.

**Do not automatically keep:** Temporary logs, one-off hypotheses, outdated implementation notes, and entire chat transcripts.

| Information | Appropriate home |
|---|---|
| Coding standards | Versioned repository instructions |
| Architectural decisions | Design or decision records |
| Repeated procedures | Reusable agent skills (see principle 4) |
| Debugging details for one issue | Task-specific context |
| Historical discussions | Searchable records |

**Practical checklist:**
- Reuse existing implementations, conventions, and architectural decisions.
- Preserve only durable, useful, and verifiable lessons in repository instructions or design records.
- Link to reusable procedures maintained under principle 4 rather than duplicating their steps in knowledge records.
- Correct or remove stale guidance; keep temporary logs and hypotheses task-specific.

**GuestProxyAgent example — reuse configuration code and maintain its evidence**

The existing [configuration module](https://github.com/Azure/GuestProxyAgent/blob/9a27720438a282876a40e36ca8bd1f25e28a135f/proxy_agent/src/common/config.rs) centralizes configuration loading, platform-specific filenames, environment-variable resolution, and accessors. Extend that mechanism instead of introducing a second parser. For durable guidance, the [context maintenance contract](https://github.com/Azure/GuestProxyAgent/blob/9a27720438a282876a40e36ca8bd1f25e28a135f/.ai/README.md#maintenance) calls for updating affected guides and source links with behavior changes, recording evidence revisions, and returning changed contracts to draft status for review. This preserves current knowledge rather than merely accumulating notes.

> **Takeaway:** Build on what is proven and keep the supporting knowledge current.

## 4. Make Recurring Work Repeatable

**Principle:** Encode recurring tasks into skills, checklists, scripts, or workflows with defined inputs, steps, checks, and outputs. Turn recurring failures into required checks or regression tests.

**Scope:** How AI should perform recurring tasks consistently. Procedures should use the maintained knowledge from principle 3 rather than duplicate architectural rules or design decisions.

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

**GuestProxyAgent example — encode PR preparation as a reusable procedure**

The checked-in [`/gpa-pr` prompt](https://github.com/Azure/GuestProxyAgent/blob/9a27720438a282876a40e36ca8bd1f25e28a135f/.github/prompts/gpa-pr.prompt.md) defines a repeatable sequence: refresh the comparison branch, establish task scope, fill the canonical proof package, run appropriate validation, request independent review, and return a PR title and description. It stops on comparison-base failures rather than silently using stale evidence. This is a configured local procedure, not evidence that it ran or an authorization to commit, push, or publish.

> **Takeaway:** Encode recurring work into repeatable, verifiable procedures.

## 5. Verify Independently

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

**GuestProxyAgent example — separate implementation, assessment, and test design**

The [agent workflow](https://github.com/Azure/GuestProxyAgent/blob/9a27720438a282876a40e36ca8bd1f25e28a135f/.ai/agent-workflow.md) assigns implementation and deterministic validation to GPA Coding, with independent assessment by a separate read-only GPA Guardian. The [GPA Tests reviewer](https://github.com/Azure/GuestProxyAgent/blob/9a27720438a282876a40e36ca8bd1f25e28a135f/.github/agents/gpa-tests.agent.md) inspects assertions and designs adversarial cases without editing or executing tests; the coding agent owns execution. This separates judgment from authorship without pretending that independent review proves correctness or that unavailable review occurred.

> **Takeaway:** Separate the creator from the verifier.

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

**GuestProxyAgent example — validate heap-growth evidence before claiming a leak**

The [UMDH analysis guide](https://github.com/Azure/GuestProxyAgent/blob/9a27720438a282876a40e36ca8bd1f25e28a135f/doc/UMDHMemoryLeakAnalysis.md) requires matching symbols, verification of stack collection in the running process, and comparable before/after snapshots from the same PID. It rejects a baseline containing only `BackTrace0` and explains that a positive allocation delta alone does not prove a leak. For an AI-proposed fix, request repeated comparable measurements and source-correlated allocation stacks, not just a lower one-off memory reading. UMDH evidence concerns Windows heap allocations, not all process-memory growth.

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

**GuestProxyAgent example — hand over a structured author proof package**

The [proof-package contract](https://github.com/Azure/GuestProxyAgent/blob/9a27720438a282876a40e36ca8bd1f25e28a135f/.ai/proof-package.md) requires eight sections: Intent, Implementation, Behavior change, Risk, Invariants, Validation, Reviewer attention, and Uncertainty. Its validation rules tie acceptance criteria to implementation and meaningful checks, record the assessed snapshot and platform, and preserve failed, missing, or stale evidence. An author-to-Guardian handoff therefore carries both the change's rationale and its unresolved gaps; the Guardian's assessment remains separate evidence rather than an author-filled approval.

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

**GuestProxyAgent example — require explicit review before relaxing a size budget**

The [binary-size budget override procedure](https://github.com/Azure/GuestProxyAgent/blob/9a27720438a282876a40e36ca8bd1f25e28a135f/ci/README.md#override-path) asks for the measured report, a narrowly scoped budget change with matching documentation, and two reviewer approvals specifically acknowledging the change. The [bloat workflow](https://github.com/Azure/GuestProxyAgent/blob/9a27720438a282876a40e36ca8bd1f25e28a135f/.github/workflows/bloat.yml) supplies the configured target/binary ceilings and crate-share exceptions. AI can diagnose growth and prepare the proposed adjustment; it should not silently raise thresholds to make a check green. The approval requirement is documented policy, not proof that branch protection enforces it.

> **Takeaway:** Automate preparation and execution where safe; retain human accountability at consequential gates.

## 9. Measure Engineering Outcomes

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
| Context and understanding | 1, 2, 3 | Work on the right problem using the current architecture |
| Execution and verification | 5, 6, 8 | Validate independently and preserve approval controls |
| Workflow standardization | 4, 7 | Make good workflows repeatable and maintainable |
| Outcome measurement | 9 | Demonstrate real productivity and quality improvements |

## Example: An AI-Native PR Workflow

1. **Understand:** Gather task-sized context, inspect current code, and find established patterns.
2. **Implement:** Use a standardized coding workflow, add tests, and record decisions.
3. **Verify:** Use independent review and execute relevant tests; attach evidence.
4. **Handoff:** Produce a structured summary of changes, results, and unresolved risks.
5. **Approve:** A human evaluates consequential findings and approves the merge.
6. **Improve:** Turn recurring defects into checks and measure outcomes.

**Loop:** Understand → Implement → Verify → Review → Deliver → Learn.

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
