---
layout: post
title: "Building AI Workflow Systems That Deserve Trust: Principles, Components and Processes for Agent-Driven Software Change"
date: 2026-11-22
description: Principles, components and processes for AI workflow systems that carry agent work from a request to a reviewed change.
---

## 0. Preface: what this paper is and how to read it

An AI workflow system is the machinery that carries an agent's work from a request to a reviewed change, and an agent is a language model that edits code and runs commands. Many teams are building such systems, and each tends to be strong in some parts and thin, and so incomplete, in others. This paper treats the parts as one whole and sets out the principles, process and components that make such systems trustworthy. A system deserves trust when its must-not-fail rules are enforced in code, its reach is contained, its checks can fail on wrong work, its records show what happened, and outcomes on the team's own tasks confirm that it works.

The scope is planned software change, which begins as a request, moves through planning, execution and review, and ends in a pull request, a proposed change reviewed before it merges into shared code. A person accepts or rejects it, or, for the lowest-risk work, a rule the organization set in advance decides. Conversational assistants, editor suggestions and agents that operate production systems fall outside, since none ends in a reviewed change.

Each rule comes with the failure it prevents, and each term is defined at first use. Everyone should start with sections 0 and 1. Newcomers then read in order through section 12, builders can take sections 4 to 11 one at a time, and readers who want to know how firm the ground is can skip to section 13.

## 1. From agent loops to workflow systems

The smallest unit is a single agent, a language model joined to a harness, the program around the model that gives it instructions and tools, such as a file editor and a shell, runs those tools and returns the results. It works in a loop, reading the task and code, editing a file or running a command and observing the result, until it decides it is done. The harness is the lever a builder controls most directly, through its instructions, tools and checks.

A session is one run of the loop. The model sees only its context window, the finite amount of text it can hold at once. When the window fills, the harness compacts older text into a summary or truncates it, and whatever it drops is lost to the model, so a rule stated early in a long session may silently stop applying.

Agents run unattended, with nobody watching, and many at once, and an agent can produce a plausible pull request faster than a person can review it, so checking becomes the harder part. Checking needs an oracle, whatever decides that the work is correct, such as a test suite, a type checker or a reviewer. Without one, an agent cannot tell finished work from work that merely looks finished. Unlike agents, reviewers do not multiply (section 9). When output outgrows them, changes tend to queue, grow larger and get read less carefully, and more defects can reach shared code. A large industry survey describes AI as an amplifier of a team's existing strengths and weaknesses: strongly positive for organizational performance with a strong internal platform, negligible with a weak one.

A skilled contractor still works from drawings, under a permit and with an inspector, because skill does not replace process. A workflow is that process applied to an agent's work, and a workflow system, made of a runtime, controls that limit the work and records of what happened, keeps it intact unattended.

Two kinds of control carry the most weight. Deterministic enforcement is rules that code applies whatever the model decides, usually through a hook, code the harness runs at a fixed point, such as before a shell command, to allow, question or refuse an action. Unlike a prompt rule (Axiom 1), a hook applies on every call it matches once registered, set to block when it fails where the harness allows, and tested. The human gate, a point where a person decides before work proceeds, belongs at judgment calls, decisions that need a person's grasp of purpose or risk, since at every step it would use up the attention those decisions need.

**Figure 1.1. From the scope of the paper to the single agent and the workflow system built around it**

![Figure 1.1. From agent loop to workflow system: planned software change is in scope, a single agent is a model and a harness working in a loop, and a workflow system wraps that agent with a runtime, controls and records]({{ "/assets/img/figure-1-1-agent-loop-to-workflow-system.png" | relative_url }})

## 2. Eight axioms and three corollaries

Each axiom is a working premise applied without re-argument, recording a failure costly to learn twice.

**Axiom 1: enforce must-not-fail rules in code, and keep prompts for judgment.** A prompt rule is only advice: the model weighs it against everything else in its context, compaction can drop it, and injected text can argue the model out of it. In code, hooks refuse recognized dangerous actions before they run, deny rules refuse whole classes of action, branch protection on the code host holds changes off a branch until required checks pass, and managed policy, organization settings, outranks any repository. Section 7 applies this.

**Axiom 2: start simple.** One agent comes before many, and a fixed pipeline, with stages in a set order, before a graph of sessions, where a supervisor that starts sessions picks the next one at run time, making the system harder to predict, limit and resume. Safety comes from firm controls, not orchestration. Sections 3, 4 and 6 apply this.

**Axiom 3: context is a finite budget.** Loaded instructions compete with the task for attention. An always-loaded instruction file adds cost and, in one study, did not generally raise task success, though it helped where a project follows non-standard practices, so an agent should load only what its task needs. Section 6 applies this.

**Axiom 4: the author cannot weaken the checks that grade it.** An agent pressed to finish may edit a test until it passes instead of fixing the work, so verification needs an oracle the author cannot change. At least one check should also stand apart from the author's context and, where possible, its model family, models of one maker and lineage, which may share blind spots. Sections 8 and 9 apply this.

**Axiom 5: the human is the scarcest resource.** A team should treat review as the binding constraint, the step that limits how fast work flows, until its own records of waiting work show otherwise. Per-action approval can decay into approval fatigue, where people approve without reading, and a very high acceptance rate can signal that decay rather than care. Attention belongs on the plan and the finished change. Section 9 develops this.

**Axiom 6: every external effect is idempotent, and resume fails loudly.** A run resumed from saved state after a crash or timeout may repeat an external effect, any change outside the agent's workspace, sending a message or creating a record twice. An idempotent effect gives the same result however often it is applied, which the builder, not the resume machinery, must supply. When saved state is uncertain, the run should fail loudly, stopping and reporting rather than guessing. Section 10 applies this.

**Axiom 7: judge the system by the team's own tasks and by outcomes.** Private tasks from the team's own work, together with outcome data on what happens after merge, are a sounder basis for trust than public scores. Unlike public tasks the model may already know, private ones are less likely to have leaked, can be refreshed and measure what matters. In one randomized trial, experienced developers believed AI made them faster while it slowed them down, so a sense of speed is not a measure. Section 11 applies this.

**Axiom 8: limit the blast radius through isolation and least privilege.** The blast radius is everything a session could damage if it goes wrong. Isolation runs each session in a sandbox, a boundary on what it can touch, and least privilege gives it only the credentials its task needs.

Prompt injection hides instructions in untrusted content, text from outside the operator, the person or team who runs the agent, which an agent cannot reliably tell from real instructions and may obey. Approval alone does not contain it, because a tricked agent still acts on everything it can reach. Containment, limiting that reach, holds. Section 7 applies this.

### Corollaries

First, **fail closed**, treating an error, a timeout or an unknown state as a refusal, since a control that opens when broken fails exactly when it is needed.

Second, **record whether each control is enforcing or merely configured**. A configured control exists in configuration, but an enforcing one is deployed, working, and has refused a deliberate test since its configuration last changed. An untested control gives false comfort: in one review of existing systems, most guardrails were prompts or conventions rather than code, and announcements described more than the code wired.

Third, **bind approvals and effects to content rather than names**. A hash is a fingerprint of content that changes exactly when the content does. A plan hash ties an approval to one version of the plan, so any edit voids it, while a name can be reused, collide or stay fixed as content changes. A content key, built from an effect's target and intended action, lets a retry recognize an applied effect and skip it.

## 3. The anatomy of a workflow system

A request, whether an issue, a chat message or a larger body of work, enters a fixed pipeline of seven stages, with a control plane over them and two durable layers, saved state and feedback, and a supervisor beneath (Figure 3.1).

**Figure 3.1. The anatomy of a workflow system**

![Figure 3.1. The anatomy of a workflow system: a request passes through seven stages under a control plane, with saved state, a feedback layer and a supervisor beneath]({{ "/assets/img/figure-3-1-anatomy-of-a-workflow-system.png" | relative_url }})

Triage decides whether a task suits an agent and how much process it deserves. Specification turns intent into a spec, a written statement of what to build and what done means. Planning checks that the spec suffices and puts the plan to its owner, the person accountable for the work, after which it changes only by recorded new version. Execution runs one loop per unit of work, the piece of the plan one session carries to a change, in an isolated workspace with its own checkout and branch. Verification runs deterministic checks, then fresh-context reviewers who start without the author's assumptions, and a fixed rule turns their findings into a verdict. The human gate places a person wherever the risk tier, a class computed from what a change touches, calls for one. Learning changes the system so a failure is less likely to repeat. The cheapest checks come first, so a person sees only work that has passed them.

The control plane watches every stage, because a dangerous action can happen anywhere, and is enforced in code. It must be configured to fail closed, since in at least one widely used harness hooks fail open on a timeout or error by default. Its separate layers run from prevention to recovery (section 7): sandbox and outbound network limits prevent, approvals decide, hooks enforce and monitoring only detects. Guards, code that blocks a dangerous action as it is attempted, belong to it.

Saved state keeps what must survive a crash in structured, checkpointed records rather than notes or labels alone, outside the transcript that compaction erodes, so work resumes instead of restarting. The feedback layer feeds events, evaluations and outcome records back into checks. The supervisor sequences units, keeps spend counts across resumes (section 10) and by default runs one agent on one unit at a time. A graph supervisor waits for a clear need.

Separation of duties means no party approves its own work: the agent that did the work never gives the final check, acceptance or merge, and at the high risk tier of section 9 a second person also approves.

Each component carries one of three statuses. A recommended default is adopted from the start, since the failure it prevents is serious and its cost low. An earn-it component works but is costly, so a team adds it once its experience shows the need. An open position has unsettled value and is tried as an experiment that stays easy to remove. Together the statuses form the confidence ladder of section 12.

## 4. Intent, triage and specification

Triage first asks **whether to delegate**. Work suits an agent best when its requirements are settled and its result can be checked. Otherwise it goes to a person or first becomes a spec that makes success checkable.

Triage then asks **how much process to apply**. Risk follows the property a change touches, such as permissions, not its line count. Ceremony, the overhead of specifying and reviewing, should stay light on small changes, where it costs more than it returns.

The light path serves small, low-risk work such as read-only reports and documentation. Its spec is a goal plus a done condition, the observable sign of completion, and the plan stays implicit. Verification is automated checks plus one fresh-context reviewer, and acceptance can follow a rule set in advance (section 9).

The full path serves work that changes data, schemas or behavior others rely on. Its spec is a contract a machine and a reviewer can both check, written before any plan so the plan answers it. It states success criteria, the checkable statements a finished change must satisfy, plus failure modes for verification to probe, assumptions for a reviewer to challenge, and blocking questions whose answers the agent would otherwise invent.

Triage picks the path from the kind of work before any plan exists, and the later risk tier moves light-path work touching a sensitive area to the full path.

When intent is unclear, the agent questions the requester until it can write the spec, and implementation starts in a fresh session, free of the conversation's false starts.

An agent can skip a phase that only a prompt enforces, so full-path phases get a blocking check, code that refuses to proceed until the spec passes the sufficiency test (section 5). The light path, where a skipped step costs little, relies on a prompt.

A spec should say what must be true of the finished change, not how to build it. One that spells out the implementation has already made the design, so its checks prove little.

Specs go stale and agents can fail to revisit early choices, so a spec needs a lifecycle and should stay short enough to review. Each delta, a mid-work change of course, merges into a living spec of what is being built, and a drift check flags where the spec or an agent's design notes diverge from the code. Before coding, the agent searches for existing code that does the job.

A change carries notes on how it was produced, since rerunning the same spec can yield different code, and links to the tracker item recording why it was requested. A change without one, such as a chat request, still proceeds, and tracing it to its origin is an open position (section 12).

## 5. Planning and deciding under uncertainty

Planning tests the spec for sufficiency, wins the owner's approval and then changes only by recorded amendment. A team can also draft the plan with a design panel, an earn-it component in which several model designers each write a plan from a different angle, such as risk first, and a separate chooser scores the drafts and picks or merges one. Its benefit over a single planner has not been measured.

The sufficiency test asks whether proceeding would make the agent invent missing information. If so, the question blocks work until the owner answers and the spec is updated. If not, it is a preference the agent decides and records. The test is a model's verdict, enforced by code and backed by fixed checks such as whether done criteria and scope are stated. Its accuracy has not been measured, and a missing or unreadable verdict should count as insufficient. Questions touching compliance, security or data retention always block, because a wrong guess there costs too much.

At approval the workflow records the plan hash. A resumed run whose hash no longer matches returns to planning, and a separate check flags any edit not recorded as a new version. Both are defaults and the paper's own position, since none of the public products examined binds plan approval to a hash.

Once its own runs show the need, a team can turn the plan's file list into a write guard, code that blocks writes outside the change surface, the files the plan may touch. It blocks outright only paths the organization marks as protected, lets generated and dependency lock files through, since legitimate work changes them, and turns any other out-of-plan write into an amendment request.

A frozen plan can fail when discovery changes the design. In the products examined, a plan reworked mid-run, even by a supervisor, needs no fresh approval and is not bound to what the owner read. Here an amendment is a new, owner-approved version that states its reason and leaves a visible mark in the change. The write guard follows it, and limits on amendments, such as never reaching into protected paths, must be enforced by the guard, since a limit in prose is only advice. Together the hash-bound approval, a default, and the write guard, an earn-it component, form the plan lock.

Unattended agents fail in two opposite ways: some stall on questions nobody can answer, others push through questions they should have asked. A missing requirement, compliance included, or a question that could destroy data stops work for the owner before any fallback, and section 7's code guards also stop destructive actions. Where stopping would strand a whole unattended run, another design builds the largest safe subset, marks each gap in the code and the change's open items, and never guesses on behavior that could destroy data. Which approach works better has not been compared.

Other questions climb a decision ladder, an open position, one rung at a time, as Figure 5.1 shows.

**Figure 5.1. The decision ladder**

![Figure 5.1. The decision ladder: a question that is not a missing requirement or a risk to data climbs from the project's own records, to an independent panel, to the safest reversible option, and every outcome becomes a decision record]({{ "/assets/img/figure-5-1-the-decision-ladder.png" | relative_url }})

The first rung consults the project's records: the spec, earlier decisions, conventions and code. A confident answer stops there. The second asks a panel of models in fresh sessions, or of people, who answer independently so no voice steers the rest. A split panel falls to the third rung, the safest option reversible in code and in effect, since reverting code does not unsend a message, and is flagged as low confidence for the next review.

The unattended path fails closed: a question meant for an absent owner is refused or halts the run. When low-tier work skips a person's plan approval (section 9), the approval is labeled as not made by a person.

Every unattended or deferred decision becomes a decision record that travels with the change, stating the question and answer, the rung that found it, a confidence showing where to look hardest, and any dissent.

A deferral, an item postponed rather than settled, has a record naming the item, why it waits, its owner and a revisit trigger, the date or event that reopens it. A register of these records is an open position (section 12) that can become a graveyard unless each trigger fires, so a team should adopt one only when deferred items are otherwise lost.

## 6. Execution: one loop, one workspace, one context budget

By default the supervisor gives each unit of work one agent in its own workspace and branch (section 3), moving conflicts to merge time.

Fan-out, running agents in parallel, spends far more tokens, the priced units of text a model reads and writes, and invites collisions. In one vendor's multi-agent research system, a single agent used about four times the tokens of a chat and a multi-agent run about fifteen times. A child session, started by another session and often called a subagent, makes small choices without seeing its siblings, which clash at merge. Fan-out should wait for three conditions.

The first is independence. Planned file sets, lock files and migration numbers included, must not overlap in the plan or in the changes, since two units adding migrations will claim the same number. The merge step sends conflicts back to a coding agent rather than resolving them, because mechanical merging cannot judge intent. The second is cost: the time saved must justify the extra tokens. The third is absorption: concurrency should follow review capacity, not machine capacity. No team was found that caps it that way, and in one practitioner thread about five to ten active sessions was a practical ceiling, with review named as the limit. Section 10's nesting and spawn limits cap only the worst case.

A dependent unit starts, by default, after the earlier one merges. A stacked change instead branches from the earlier branch, so its pull request shows only its own changes, and the stack merges bottom up. When a lower change is revised, every change above it is marked stale, updated and verified again. Stacking, within one planned change or across several, must earn its place.

A dependent session begins with a structured handoff, because a prose summary drops decisions and constraints the successor must respect. It carries the earlier unit's decision records, amendments and deferrals, its effect journal of each external effect the unit caused, and a path to its full trace.

Retrieval on demand protects each agent's context budget, the third axiom, by loading a file by path only when needed, since loading everything crowds out the task. Progress notes outside the window let a new session resume when a long-running agent stops short, and a child session for a subtask starts with a clean window.

Because compaction can drop early rules, lasting rules belong in files the harness re-injects or, better, in code. Restating standard practice in an always-loaded file wastes the budget, so only a short list of non-standard conventions stays loaded and the rest becomes skills, guidance meant to load when the task touches what it covers, such as a given path, so the agent pays mainly for relevant guidance. Whether a skill loads can rest on the model's own judgment, and the benefit of loading on demand has not been measured.

## 7. Control and containment

Enforcement stops a known-bad action as it is attempted; containment limits what an agent can reach, so a tricked agent does less harm. Table 7.1 places both among five layers of defense.

**Table 7.1. Layers of defense**

| Layer | Its job | Typical mechanisms |
|---|---|---|
| Prevent | limit what a session can reach | sandbox, closed network, narrow credentials |
| Decide | approve or refuse a proposed step | plan approval, confirmation |
| Enforce | block known-bad actions as they happen | hooks, permission backstops |
| Detect | notice afterwards | monitoring, audit logs |
| Recover | undo damage | revert, staged rollout, backups |

Containment is the prevent layer, and the guard is the enforce layer.

Code must stop the most destructive actions: forced pushes that rewrite shared history, protected-branch deletion and shell commands that wipe a workspace. Guards also refuse overwrites of protected paths, some holding the policy that governs the agent; edits outside the approved change surface; and writes that look like secrets, since a committed credential may stay exposed after deletion. They block installing a tool server not on an approved list until a person confirms it and, where an agent should only read a database, refuse any statement but a single read.

The best guard acts early and says why it refused, so the agent picks a safer route. Behind it stands a permission backstop, such as a database role that cannot drop shared tables, because string matching catches accidents but a determined or injected agent can rephrase the command. A shared stateful environment counts as production: in one public report, an agent deleted a production database where development, staging and production were not separated.

A guard that fails open allows the action on a timeout or error, so guards fail closed. In one carefully built guard set, most guards let the action through on malformed input or a missing configuration, and only a few failed closed. No hook is assumed to stop an action, since block-on-failure covers only some hook types and events; the uncovered ones are listed. A transient error is retried before it counts as a refusal, and where one cannot fail closed, branch protection or a wrapper with its own timeout enforces the rule. An automatic reviewer, a model judging whether each action is safe, denies when its own review fails but cannot stand alone: in one vendor's own test on a small set of real actions beyond what the user asked, it missed about one in six.

Each guard has a conformance probe, a deliberate attempt at the blocked action after every configuration change, aimed at predictable bypasses: a shell command that edits a file past a guard watching only the editor, one file under several names, a guard never registered, a missing setting that leaves a permissive default, a fallback to a looser parse, and a scope from an earlier plan that nothing clears. Several known bypasses need deliberate setup, so such guards mainly stop a careless or over-eager agent, not a determined one. A bypass needs a recorded, time-limited exception from a named person, so nobody silently switches a guard off.

A control that is not enforcing still looks like protection, so every control shows its state where operators look: enforcing (it blocks, as a passing probe proves), advisory (it only reports) or off. In one system read, nearly every feature flag shipped off, and a budget documented as a hard ceiling only computed a state and left enforcement to its callers.

Containment answers prompt injection. In one preprint study, injected instructions got two coding agents to run attacker-supplied commands in up to 84% of attempts, and a published proof of concept leaked private repository content after an agent read a malicious issue in a public repository. A serious leak needs all three of: reading private data, reading content an attacker could have written, and sending information out. Every session lacks at least one, which closes that path, and each sees only one repository.

A virtualization boundary does not filter network traffic, so outbound traffic is limited to a short approved list. A package mirror, the usual exception, must never forward other requests, or stolen data could leave inside the names requested. Continuous integration runs on agent branches without deployment secrets, and reviewer sessions that read pull-request text face injection too.

Credentials should be as narrow as the task allows, because an attacker who steers the agent can do whatever it can. The agent pushes only to its own new branch or the pull request it was asked to work on, and holds no credential that lets it push directly. Organization policy outranks the repository, and no committed file may switch on a permissive mode, or any pull request could widen permissions. Least agency extends this to tools: an agent gets only the tools its task needs, each narrowly permitted.

The decide layer is plan approval (section 5) and the human gate (section 9). The enforce layer also covers dependency changes, which bring in code the team did not write. The detect layer only notices, so monitoring is never protection. The recover layer reverts, stages rollouts and keeps backups beyond the agent's reach: in one disputed public report, the backups sat on the volume the agent deleted.

## 8. Verification: checking the checks and the checker

An oracle means something only if it would fail on wrong work. It fails when the answer leaks into the task or the tests are too weak. In one study of a public benchmark, not yet peer reviewed, about a third of one agent's passing patches had the solution in the task and about a third more were flagged as passing on weak tests; removing them cut its resolve rate from about 12.5% to about 4%. The author never alters its own checks, and the control protecting the tests is verified as enforcing: in one system it shipped switched off. Seeded defects and mutation testing test whether the checks notice.

A known-bad copy is the change with one behavior the spec values removed; tests still passing on it are blind to that behavior. Copies target the most valued behaviors and commonest defects. A separate session that cannot see the code writes a test that fails on the copy, so the test cannot adapt to it. Without a meaningful known-bad copy, a person reviews the tests.

Reviewers start in a fresh context, free of the author's assumptions. A reviewer asked for problems nearly always reports some, so findings carry severities and the loop has a stop rule. The default pairs deterministic checks with one strong reviewer, adding lenses, reviewers focused on one concern, only where recorded misses demand them. No study comparing one strong reviewer with several lenses was found; on one independent preprint benchmark of general code review the best system caught only about a third of known defects, and no independent precision figure was found for any commercial reviewer. Code, not a model, collects the findings.

No team should rely on one model judge, a model that grades work. Judges tend to favor longer answers and certain prompt positions and can accept plausible but wrong claims. In general judging tasks, a panel of smaller models from different makers, whose habits differ, has judged better than one large model. The judge is checked against human verdicts on a sample and may say it cannot tell. In one code-review benchmark, only one of three model judges was checked against people; it matched the human majority on about 72% of 60 comments.

A verdict function maps findings to a verdict by fixed rule, not vote, so no argument can change it. A blocker stops the change and can never be excused; a major finding is a real problem only the owner may waive; a minor finding is recorded but stops nothing.

**Table 8.1. The verdict function**

| What the findings contain | Verdict |
|---|---|
| Any blocker | Reject |
| An unwaived major finding | Reject |
| A waived major finding | Accept, filing follow-ups under an owner |
| Only minor findings | Accept, filing follow-ups under an owner |
| No findings | Accept |
| A finding of unknown severity | Treat as major |
| A malformed finding | Treat as major, never waivable |
| An internal error in a reviewer or check | Inconclusive |

Unknown severity counts as major so uncertainty cannot lower the bar, and an internal error is retried, then escalated, never accepted. Accept means ready for the human gate, not merged, and carries its evidence: tests run and results, a diff audit, the commit reviewed, a recheck of each fix, and lint output.

Planted text can persuade a reviewer: in one study, text in pull request metadata misled automated review pipelines in 32 of 33 known vulnerability cases. So a model reviewer may tighten a verdict but never override a failed deterministic check or excuse a protected-area edit. Waivers, each the owner's recorded acceptance of one major finding for one change, are counted so the cost of attention stays visible.

A review-fix loop gets a round cap and a progress test. It is stuck when a round clears nothing, a fixed finding returns, or a finding survives several rounds. A stuck loop restarts in a fresh session carrying only the plan and current findings, and goes to a person if it sticks again. In one vendor report, the 8.5% of its pull requests that ran past five review rounds used 47.4% of its review-loop spend. Rounds that each fix one instance and find another signal a structural problem, which the owner fixes as a class by permitting only known-safe cases.

Silence is not success. In one preprint study of 95 failed multi-agent runs, mostly on non-coding tasks and labeled by a single model, evidence against the error was present in 68, and in 60 of those no agent acted on it. So checks run by rule, not at the agent's discretion. A crash or missing check is never a pass, because a failing part can produce confident invented output: in one harness, a content-filter crash was followed by a final answer no agent wrote in 25 of 58 hand-built runs. A flaky check is quarantined under an owner, never skipped.

## 9. The human gate: spending attention where judgment lives

Human attention is the scarcest resource: in one measured system, most delay before a merge was waiting for a person. Multiplying change multiplies review habits, and skipped review is as much a failure as slow review, for humans too: in one large public pull-request dataset, 58% of agent-authored and 75% of human-authored changes had no recorded review.

Approving every action can numb people into not reading, so attention goes to three judgment points: plan approval before effort is spent, change acceptance before merge, and promotion to production.

Rules compute the risk tier, which sets what each point demands, from the planned change surface and recheck it on the final diff, so a narrow plan cannot understate a change. A protected path forces the high tier, a change to a control takes at least the tier of what it protects, a dependency change raises it, and anything unclassified is high. A model may raise a tier but never lower it.

The high tier needs owner plan approval, an owner reading of the explainer, a summary for the reviewer, before the diff, a second engineer's approval, separate promotion and every rule meant to enforce on; lower tiers may run some controls advisory, but destructive-action guards are on at every tier. The medium tier needs owner plan approval and acceptance. At the low tier no person approves the plan; a preset rule may accept the change when checks pass, a fresh-context reviewer finds nothing and no protected path is touched, random human sampling keeps it honest, and each organization decides whether such a rule is allowed at all.

Plans should be approved in batches and new work capped at what reviewers can clear. These tiers and rules are reasoned positions. The nearest published practice: since one vendor adopted risk-tiered review, about three quarters of agent-written pull requests in its codebase merge without a person correcting the agent or discussing the engineering (a count of corrections, not of whether anyone looked), while six change types its policy names always get a human check.

The explainer summarizes the change, gathers decision records, deferred items and minor findings, and links test evidence into the diff. It shows where to look rather than arguing correctness, since the author's confident account can sway a reader. A repository-level architecture view should accompany it, since line-by-line reading can miss architecture.

The requester states the need; the author is the agent plus its dispatching engineer; the owner, usually that engineer, is accountable; the second approver has no part in the work; the merger is neither author nor requester, or is an explicit low-tier policy. Nobody approves a change they hold a stake in, so the requester confirms the criteria fit the need but never approves the change. Computed checks separate doing from accepting at the medium tier; the high tier adds a second approver because the owner is usually part of the author.

Control what reaches review: cap diff size, keep batches small against the instability of fast change, and cap concurrent work at reviewer capacity and open changes per contributor. Contributions should disclose agent use and be verified by a person; very large changes without design discussion should be declined.

A named owner, the last line of defense, is meant to understand and answer for each change. In one large developer survey, about one respondent in five said AI had cost them confidence in their own problem solving, so ownership must be kept deliberately, and whoever submits a change should own it. An editable author label records origin but controls nothing, so every agent change should carry a service identity that the system owns and nobody can remove.

## 10. State, durability and cost

Saved state, written by every stage, lets work survive a crash. A crash can leave an outside action half done, so each effect needs a key that makes its retry harmless. Platforms that survive crashes leave this to the builder. Replaying their journal of completed steps skips finished ones, where restarting the interrupted step repeats its effect, so replay suits agent work better but still needs keys.

The key is section 2's content key, built from the effect's target and intended action, so a recovered attempt finds the effect done. Tool-call identifiers change on retry, and run identifiers on a fresh run.

Each effect gets an intent record before it and a done record with its result after, forming the effect journal that section 6's handoff passes on. Intent without done is an unknown state, so the resumed session probes the real system and halts for a person if the probe is inconclusive. The journal ships on by default, since protection shipped off protects no one.

A lease, a time-limited claim naming a run's current owner, and a fencing token that rises with each new holder reject a slow session that wakes after another has taken its run. Both matter only when several processes can own a run. Early on, a team can instead check whether an expected branch or similar exists before creating it, keeping intent and done records only where the target does not drop duplicates itself.

Resume fails loudly (Axiom 6), stopping and reporting on unrecognized state instead of guessing, which is how it fails closed. A dirty working tree, with uncommitted changes, means something acted off the record. A damaged record, or a recorded commit missing from the branch's history, cannot be trusted. Two mismatches reroute instead: a changed plan hash returns the run to planning (section 5), and a branch past the last recorded commit returns to review, since saved state never recorded that work. Starting fresh silently would hide each mismatch.

Stored state is also a trust boundary. A later session with different trust acts on whatever a checkpoint, the saved state resume reads, contains, so no checkpoint may hold an authority token, a credential or approval granting power the reader was never given. Traces, journals and evaluation sets can hold personal data and secrets, so they get retention and access rules, redaction, and a secret scan before another session reads them.

Helpers starting helpers can multiply a small plan and its cost. In one public issue report on an agent tool, five planned helpers became more than 360 and drained a five-hour usage quota from about 3% to 100% within minutes before the user stopped them by hand. Limits on nesting depth and on how many sessions one session starts come first, then spend accounting that survives resume and counts child sessions, since a reset count resets the protection. A cap can overshoot by one response plus helpers still running, and one agent toolkit defaults budget and turn limits to none, so set both, leave headroom, and count every billed token type, cache reads included. In one public report a long run used 819.5 million cache-read tokens against 11.3 million output tokens, and the user suspected its caps never fired because the meter counted output only. The supervisor of section 3 enforces these defaults. A complete refusing cap, an open position (section 12), also blocks new work at a safe point between effects, since an unread warning stops nothing. It warns early and labels estimates advisory. A runaway chain stops gracefully at a machine-readable refusal naming the limit, showing the owner the spend and chain, so any raise is deliberate.

## 11. Observability, measurement and learning

The feedback layer turns section 10's records into change, so each bad outcome alters practice instead of leaving untested beliefs. Its runtime loop acts on events as they happen, and its evaluation loop grades the system on tasks with known answers.

Each runtime event is a structured record whose failure class chooses the response. Transient failures, such as timeouts, are retried. Deterministic ones, recurring identically, get a drafted fix for a person to apply. Those needing a person, such as a missing credential, notify someone and stop. Self-repair, resting on tracing, stays limited to retries and drafted fixes, an open position (section 12) that no system examined shows working in practice.

In the same preprint study of 95 failed runs (section 8), over half traced to the plan, the brief (the task statement an agent receives) or the stopping rule. Restoring the agent's state replays the flaw, so repair revises the plan. A monitor must be live, since an alert that can never fire protects nothing.

Scores must come from real runs and be recorded, and the model judge must agree with human opinion. The grader, whatever scores a run, should pass the reference answer graded against itself, but that sanity check is not a score. In one system, five of six recorded scores were such self-checks, saying nothing about the agent.

Evaluation tasks should be private and drawn from the team's own reverts, fixes and incidents. One vendor suggests starting with 20 to 50 such tasks, a set that reveals only large differences, so it is refreshed and grown. People read transcripts before trusting a score, and each task runs several times, since output varies. The grader deserves the agent's scrutiny: in one vendor's account, fixing grading bugs and ambiguous tasks in a public research-reproduction benchmark raised its own model's score from 42% to 95%. A known-bad copy (section 8) that passes reveals a broken oracle.

Outcomes matter more than merge rate, which is easy to game. Acceptance and rework rounds, the fixes before acceptance, show whether reviewers take a change and at what cost. Reverts expose changes that only looked right, and post-merge incidents expose what review missed. For attention, review wait, the time a finished change waits for a person, shows where people limit throughput, and review depth, the time and sampled audit findings per change, shows whether review is real. Human touches, occasions needing a person, show where attention goes, and false rejects and denials how often a check or permission blocks good work. Cost per change counts child sessions too.

Reverts and incidents count against each change's service identity (section 9) for a set period after merge, gathered in a periodic outcome report with a named reader and schedule.

A recurring finding becomes a deterministic check on every change touching that area, and whoever wrote it, it must prove it can fail before it blocks. Scheduled jobs with no model watch the deferral triggers of section 5.

The harness is itself code under change: its prompts, hooks and configuration should be versioned for review and rollback, and a change should pass the private tasks before reaching everyone.

One vendor advises deleting preference-only hooks and advice newer models no longer need. A guard against a weakness newer models may lack could be tested with private tasks that invite the blocked action, and a person may remove that long-quiet guard if the model no longer attempts it. This untried pruning test is an open position, and section 12 names the controls exempt from it.

## 12. Putting it together: a reference process, a confidence ladder and an adoption path

Figure 12.1 joins sections 4 to 11 into the reference process.

**Figure 12.1. The reference process**

![Figure 12.1. The reference process: a request moves from triage through specification, planning, execution, verification and the human gate to merge and learning, with exits for unsuited work, invented requirements and stuck review-fix loops, a light path that rejoins at execution, and loops for rejection and learning]({{ "/assets/img/figure-12-1-the-reference-process.png" | relative_url }})

A rejected change returns to execution through a capped fix round. An accepted one meets the human gate for its risk tier, and a person or an explicit policy, never the author, merges it. The dashed line on the right carries learning back, as recurring findings become new checks.

The three statuses of section 3 form the confidence ladder. Unattended safety comes from guardrails, not parallelism, as one builder of an unattended system said, though most of that system's limits were instructions to the model. The recommended defaults are those guardrails, the axioms of section 2 in practice, and a team should adopt them now.

The earn-it components add scale or structure, each adopted once a team's own runs show it pays. A graph supervisor and many parallel sessions add scale, stacked sequencing and the plan lock's write guard add structure, and several review lenses and a design panel deepen scrutiny.

The open positions, worth trying though unsettled, are the decision ladder of section 5, the deferral register, tracing a change to its origin without a tracker item, the refusing cap, self-repair limited to retries and drafted fixes, and pruning controls that stay quiet.

Adoption proceeds in four levels, a small, contained system first, as starting simple and limiting the blast radius require, and measurement decides each move up. A team may stop at any level, and at each should first configure what the platform, code host and continuous integration already provide.

Level 0 contains and measures. One agent works in a sandbox on one repository, with minimum credentials and closed egress, and backups beyond its reach. A person reviews every change, and ordinary tests and lint run. The team moves up once a private eval of its own failures and an outcome log exist.

Level 1 enforces. Guards fail closed, and probes confirm each is enforcing. An effect journal records every external effect, so an uncertain resume stops loudly. Limits on spend, depth and fan-out arrive with a context budget, as do triage and checkable specifications. Small, low-risk work can now run under full review; riskier work waits for Level 2. The team moves up once guards and journal run without silent failure.

Level 2 verifies and tiers. The sufficiency test arrives, and the plan lock's write guard, with its amendment path, enters on trial. Verification gains known-bad copies and one fresh-context reviewer, adding lenses only when misses show a need, and the computed verdict decides. Risk tiers with caps on change size follow. The team moves up once review wait stops being the main constraint while reverts, incidents and audits hold steady.

Level 3 earns scale. Parallel sessions arrive, capped by available reviewers, then a graph supervisor, stacked sequencing and the open positions, each once runs show it pays. Every move up should rest on outcome records over time, never on one before-and-after comparison.

Table 12.1 lists anti-patterns, recurring failures such as controls that read stronger than they are, with the principle each breaks and where to look first. It doubles as a pruning checklist, except for destructive-action guards, containment and credential scope, never pruned for being quiet, because a guard against rare catastrophe is meant to be quiet. They change only with the threat model, the account of what they defend against.

**Table 12.1. Anti-patterns and how to notice them**

| Anti-pattern | Principle it breaks | Where to look |
|---|---|---|
| A control that looks protective but is not enforcing | Enforcing versus configured | Conformance probes |
| A destructive-work freeze only in a prompt | Enforce in code | The list of guards |
| Approval fatigue from per-action prompts | Attention is scarce | Approval rate |
| The author editing tests it did not write | Independent checks | Test changes, known-bad copies |
| A crash or warning-only cap treated as safe | Fail closed | Internal error count, spend past each cap |
| Summary handoffs, thick always-loaded instructions | Context is a budget | Context cost per session |
| Fan-out without independent units | Start simple | Conflict rate, review queue |
| A resume that starts fresh or replays effects | Loud, idempotent resume | Intents with no done record |
| Merge rate as the quality signal | Measure outcomes | Reverts and incidents by tier |
| An advisory estimate labeled as a hard cap | Enforcing versus configured | What happens when the limit is reached |
| Review skipped as load grows | Attention is scarce | Changes merged without review |
| Full ceremony on a small change | Start simple | Process effort against change size |

## 13. Limits of the evidence and open questions

Most of this paper's outside evidence is vendor claims, product documentation or practitioner accounts. The measured studies are few, several are preprints, and one rests mostly on non-coding tasks. The firmest ground lies under guidance that records costly failures: enforcement in code, approval alone being no defense, independent verification, human gates at judgment points, effects safe to repeat and controls that fail closed.

The first six axioms are well supported in their core claims, with two qualifications: some frameworks favor explicit graphs over starting simple, and the case for review as the binding constraint rests mostly on vendor telemetry and surveys. Axioms 7 and 8 are firmer in what they reject than in what they prescribe. They clearly rule out public benchmarks and approval alone as a defense, as audits and attack studies have measured, but the replacements, private tasks with outcome records and contained sessions, rest on vendor and industry guidance rather than measurement. The statuses follow the same order: support falls from recommended defaults to open positions, and earn-it components lack a comparison showing how much they help.

Three questions about review remain open: how often review comments are correct on everyday changes rather than selected examples, how much each lens adds, and what a review costs. Even recall varies widely across benchmarks, above and below section 8's figure. The design's most distinctive components are the least tested, on thin field evidence: a single multi-unit run inferred from its still-unmerged pull requests, about six weeks of review timing from one system, and an evaluation apparatus whose few recorded scores were mostly self-checks. A team should therefore test each control in its own system rather than assume that configured means working.

Each earn-it component has a test. Several review lenses, and design panels, pay only if they find what a strong reviewer misses at a worthwhile cost. The plan lock's write guard is worth keeping if it reduces drift without adding rework. Parallel sessions under a graph supervisor, and stacking many units, are justified only once review stops being the constraint.

Among the open positions, the decision ladder should be revised if flagged decisions are reversed so often that asking would have been cheaper. A refusing cap is confirmed when it stops a runaway and revised if it refuses legitimate work too often. The deferral register earns its keep if deferred items return rather than vanish, and traceability without a tracker if each change still leads back to its request. Self-repair should grow beyond simple retries only where its repairs hold. Pruning quiet controls is confirmed if retired guards stay unneeded and revised if their failures return. Acceptance by policy remains an organizational choice, informed by each risk tier's incident and revert rates.

A team's own records over many runs decide parallel sessions and risk tiers, though how to set and audit tiers, and when to prune a control that no longer earns its cost, remain open. Lenses, the write guard, the ladder and caps matter too rarely to judge without many runs. An open component should be adopted deliberately, measured, and kept removable. An open position is not a weak one but marks where a builder should measure first.
