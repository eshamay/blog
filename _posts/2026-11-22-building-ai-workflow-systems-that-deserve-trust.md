---
layout: post
title: "Building AI Workflow Systems That Deserve Trust: Principles, Components and Processes for Agent-Driven Software Change"
date: 2026-11-22
description: Principles, components and processes for AI workflow systems that carry agent work from a request to a reviewed change.
---

## 0. Preface: what this paper is and how to read it

An agent is a language model that edits code and runs commands. An AI workflow system is the machinery that carries an agent's work from a request to a reviewed change. Many teams are building such systems. Each tends to be strong in some parts and thin, and so incomplete, in others. This paper treats the parts as one whole. It sets out the principles, process and components that make such systems trustworthy. So when does a system deserve trust? When five things hold. Its must-not-fail rules are enforced in code. Its reach is contained. Its checks can fail on wrong work. Its records show what happened. And outcomes on the team's own tasks confirm that it works.

The scope is planned software change. It begins as a request, moves through planning, execution and review, and ends in a pull request: a proposed change reviewed before it merges into shared code. A person accepts or rejects it. Or, for the lowest-risk work, a rule the organization set in advance decides. What's left out? Conversational assistants, editor suggestions and agents that operate production systems, since none ends in a reviewed change.

This paper pairs each rule with the failure it prevents and defines each term at first use. Everyone should start with sections 0 and 1. After that, if you're new, read in order through section 12. If you're building, you can take sections 4 to 11 one at a time. If you want to know how firm the ground is, you can skip to section 13.

## 1. From agent loops to workflow systems

Start with the smallest piece, a single agent. It's a language model joined to a harness, the program around the model. The harness gives the model instructions and tools, such as a file editor and a shell, runs those tools and hands back the results. The agent works in a loop: it reads the task and the code, edits a file or runs a command, looks at what happened, and goes around again until it decides it's done. The harness is the lever you control most directly, through its instructions, tools and checks.

One run of that loop is a session, and a session's memory is finite. The model sees only its context window, the finite amount of text it can hold at once. When the window fills, the harness either compacts older text into a summary or truncates it. Whatever gets dropped is lost to the model. So a rule you stated early in a long session can quietly stop applying.

Now add scale. Agents run unattended, with nobody watching, and many at once. An agent can produce a plausible pull request faster than a person can review it. So where does the hard part go? To checking. Checking needs an oracle, whatever decides that the work is correct: a test suite, a type checker or a reviewer. Without one, an agent can't tell finished work from work that only looks finished. Unlike agents, reviewers don't multiply (section 9). When output outgrows them, changes tend to queue, grow larger and get read less carefully, and more defects can reach shared code. A large industry survey describes AI as an amplifier of a team's existing strengths and weaknesses: strongly positive for organizational performance with a strong internal platform, negligible with a weak one.

Think of a skilled contractor. Even a skilled one still works from drawings, under a permit and with an inspector, because skill doesn't replace process. A workflow is that process applied to an agent's work. A workflow system keeps the process intact when nobody is watching, and it's made of three things: a runtime, controls that limit the work, and records of what happened.

Two kinds of control carry the most weight. The first is deterministic enforcement, rules that code applies whatever the model decides. It usually works through a hook, code the harness runs at a fixed point, such as before a shell command, to allow, question or refuse an action. Unlike a prompt rule (Axiom 1), a hook applies on every call it matches once it's registered, set to block when it fails where the harness allows, and tested. The second is the human gate, a point where a person decides before work proceeds. Put one at every step and it eats the attention that judgment calls need, the decisions that take a person's grasp of purpose or risk. So the human gate belongs at the judgment calls.

**Figure 1.1. From the scope of the paper to the single agent and the workflow system built around it**

![Figure 1.1. From agent loop to workflow system: planned software change is in scope, a single agent is a model and a harness working in a loop, and a workflow system wraps that agent with a runtime, controls and records]({{ "/assets/img/figure-1-1-agent-loop-to-workflow-system.png" | relative_url }})

## 2. Eight axioms and three corollaries

Each axiom records a failure that's costly to learn twice. This paper applies each as a working premise without re-argument.

**Axiom 1: enforce must-not-fail rules in code, and keep prompts for judgment.** A prompt rule is only advice. The model weighs it against everything else in its context. Compaction can drop it. Injected text can argue the model out of it. In code, hooks refuse recognized dangerous actions before they run. Deny rules refuse whole classes of action. Branch protection on the code host holds changes off a branch until required checks pass. Managed policy, the organization's settings, outranks any repository. Section 7 shows these controls at work.

**Axiom 2: start simple.** One agent comes before many. A fixed pipeline, with stages in a set order, comes before a graph of sessions. In a graph, a supervisor that starts sessions picks the next one at run time, which makes the system harder to predict, limit and resume. Safety comes from firm controls, not orchestration. Sections 3, 4 and 6 show starting simple in practice.

**Axiom 3: context is a finite budget.** Loaded instructions compete with the task for attention. An always-loaded instruction file adds cost. One study found it didn't generally raise task success, though it helped where a project follows non-standard practices. So an agent should load only what its task needs. Section 6 shows how to spend that budget.

**Axiom 4: the author cannot weaken the checks that grade it.** Push an agent to finish and it may edit a test until it passes instead of fixing the work. So verification needs an oracle the author can't change. Models of one maker and lineage form a model family, which may share blind spots. At least one check should also stand apart from the author's context and, where possible, its model family. Sections 8 and 9 apply this to checks and review.

**Axiom 5: the human is the scarcest resource.** The binding constraint is the step that limits how fast work flows. Treat review as that step until your own records of waiting work show otherwise. Ask people to approve every action and approval can decay into approval fatigue: people approve without reading. So a very high acceptance rate can signal that decay rather than care. Attention belongs on the plan and the finished change. Section 9 shows where it goes.

**Axiom 6: every external effect is idempotent, and resume fails loudly.** A run resumed from saved state after a crash or timeout may repeat an external effect, any change outside the agent's workspace. It might send a message or create a record twice. An idempotent effect gives the same result however often it's applied. You, the builder, must supply that, not the resume machinery. When saved state is uncertain, the run should fail loudly, stopping and reporting rather than guessing. Section 10 shows what this takes when a run dies halfway through.

**Axiom 7: judge the system by the team's own tasks and by outcomes.** Private tasks from your team's own work, plus outcome data on what happens after merge, are a sounder basis for trust than public scores. Why? The model may already know public tasks. Private ones are less likely to have leaked, can be refreshed and measure what matters. In one randomized trial, experienced developers believed AI made them faster, while it slowed them down. So a sense of speed isn't a measure. Section 11 shows how to measure.

**Axiom 8: limit the blast radius through isolation and least privilege.** The blast radius is everything a session could damage if it goes wrong. Isolation runs each session in a sandbox, a boundary on what it can touch. Least privilege gives it only the credentials its task needs.

Prompt injection hides instructions in untrusted content. That's text from outside the operator, the person or team who runs the agent. An agent can't reliably tell it from real instructions and may obey it. Approval alone doesn't contain it, because a tricked agent still acts on everything it can reach. Containment, limiting that reach, holds. Section 7 shows how to build it.

### Corollaries

First, **fail closed**: treat an error, a timeout or an unknown state as a refusal. A control that opens when broken fails exactly when you need it.

Second, **record whether each control is enforcing or merely configured**. A configured control exists in configuration. An enforcing one is deployed, working, and has refused a deliberate test since its configuration last changed. An untested control gives false comfort. One review of existing systems found that most guardrails were prompts or conventions rather than code, and that announcements often promised more than the code actually did.

Third, **bind approvals and effects to content rather than names**. A hash is a fingerprint of content that changes exactly when the content does. A plan hash ties an approval to one version of the plan, so any edit voids it. A name can be reused, collide or stay fixed as content changes. A content key, built from an effect's target and intended action, lets a retry recognize an applied effect and skip it.

## 3. The anatomy of a workflow system

A request, whether an issue, a chat message or a larger body of work, enters a fixed pipeline of seven stages. A control plane sits over them. Beneath them sit two durable layers, saved state and feedback, and a supervisor (Figure 3.1).

**Figure 3.1. The anatomy of a workflow system**

![Figure 3.1. The anatomy of a workflow system: a request passes through seven stages under a control plane, with saved state, a feedback layer and a supervisor beneath]({{ "/assets/img/figure-3-1-anatomy-of-a-workflow-system.png" | relative_url }})

Triage decides whether a task suits an agent and how much process it deserves. Specification turns intent into a spec, a written statement of what to build and what done means. Planning checks that the spec suffices and puts the plan to its owner, the person accountable for the work. After that, the plan changes only by recorded new version. Execution runs one loop per unit of work, the piece of the plan one session carries to a change. Each loop gets an isolated workspace with its own checkout and branch.

Verification runs deterministic checks, then fresh-context reviewers who start without the author's assumptions. A fixed rule turns their findings into a verdict. The human gate places a person wherever the risk tier, a class computed from what a change touches, calls for one. Learning changes the system so a failure is less likely to repeat. The cheapest checks come first, so a person sees only work that has passed them.

The control plane watches every stage, because a dangerous action can happen anywhere, and code enforces it. What if a hook times out or errors? In at least one widely used harness, it fails open by default. So you must configure the control plane to fail closed. Its separate layers run from prevention to recovery: sandbox and outbound network limits prevent, approvals decide, hooks enforce and monitoring only detects. Section 7 sets them side by side. Guards, code that blocks a dangerous action as it's attempted, belong to the control plane too.

Saved state keeps what must survive a crash. It lives in structured, checkpointed records rather than notes or labels alone, outside the transcript that compaction erodes. So work resumes instead of restarting. The feedback layer feeds events, evaluations and outcome records back into checks. The supervisor sequences units and keeps spend counts across resumes (section 10 shows why). By default it runs one agent on one unit at a time. A graph supervisor waits for a clear need.

Separation of duties means no party approves its own work. The agent that did the work never gives the final check, acceptance or merge. At the high risk tier, a second person also approves. Section 9 sets out what puts a change there.

Each component carries one of three statuses. You adopt a recommended default from the start, because the failure it prevents is serious and its cost low. An earn-it component works but costs a lot, so you add it once your experience shows the need. An open position has unsettled value. You try it as an experiment that stays easy to remove. Together the three statuses form the confidence ladder, which section 12 fills in component by component.

## 4. Intent, triage and specification

Triage first asks: should an agent do this work? Work suits an agent best when its requirements are settled and its result can be checked. Otherwise it goes to a person or first becomes a spec that makes success checkable.

Next, triage asks how much process to apply. What drives the risk? The property a change touches, such as permissions, not its line count. Ceremony is the overhead of specifying and reviewing. On small changes it costs more than it returns, so it should stay light there.

The light path serves small, low-risk work such as read-only reports and documentation. Its spec is a goal plus a done condition, the observable sign of completion. The plan stays implicit. Verification is automated checks plus one fresh-context reviewer. Acceptance can follow a rule set in advance; section 9 says when.

The full path serves work that changes data, schemas or behavior others rely on. Here the spec is a contract a machine and a reviewer can both check. You write it before any plan, so the plan answers it. It states success criteria, the checkable statements a finished change must satisfy. It also lists failure modes for verification to probe, assumptions for a reviewer to challenge, and blocking questions whose answers the agent would otherwise invent.

Triage picks the path from the kind of work, before any plan exists. Later, the risk tier moves light-path work touching a sensitive area to the full path.

When intent is unclear, the agent questions the requester until it can write the spec. Implementation then starts in a fresh session, free of the conversation's false starts.

An agent can skip a phase that only a prompt enforces. So full-path phases get a blocking check: code that refuses to proceed until the spec passes the sufficiency test, which section 5 explains. The light path, where a skipped step costs little, relies on a prompt.

A spec that spells out the implementation has already made the design, so its checks prove little. A spec should therefore say what must be true of the finished change, not how to build it.

Specs go stale, and agents can fail to revisit early choices. So a spec needs a lifecycle and should stay short enough to review. Each delta, a mid-work change of course, merges into a living spec of what is being built. A drift check flags where the spec or an agent's design notes diverge from the code. Before coding, the agent searches for existing code that does the job.

A change carries notes on how it was produced, since rerunning the same spec can yield different code. It also links to the tracker item recording why it was requested. A change without one, such as a chat request, still proceeds. Tracing it to its origin is an open position, listed with the others in section 12.

## 5. Planning and deciding under uncertainty

Planning tests the spec for sufficiency, wins the owner's approval and then changes only by recorded amendment. You can also draft the plan with a design panel, an earn-it component. Several model designers each write a plan from a different angle, such as risk first. A separate chooser scores the drafts and picks or merges one. The panel's benefit over a single planner has not been measured.

How do you know a spec is enough? The sufficiency test asks whether going ahead would make the agent invent missing information. If it would, the question blocks work until the owner answers and the spec is updated. If it wouldn't, it's a preference the agent decides and records. A model gives the verdict, code enforces it, and fixed checks back it up, such as whether the spec states done criteria and scope. The test's accuracy has not been measured. A missing or unreadable verdict should count as insufficient. Questions touching compliance, security or data retention always block, because a wrong guess there costs too much.

At approval, the workflow records the plan hash. A resumed run whose hash no longer matches returns to planning. A separate check flags any edit not recorded as a new version. Both are defaults and this paper's own position, since none of the public products examined binds plan approval to a hash.

Once your own runs show the need, you can turn the plan's file list into a write guard. It's code that blocks writes outside the change surface, the files the plan may touch. It blocks outright only paths your organization marks as protected. It lets generated and dependency lock files through, since legitimate work changes them. Any other write outside the plan becomes an amendment request.

A frozen plan can fail when discovery changes the design. In the products examined, a plan reworked mid-run, even by a supervisor, needs no fresh approval and isn't bound to what the owner read. Here an amendment is a new, owner-approved version that states its reason and leaves a visible mark in the change, and the write guard follows it. A limit in prose is only advice, so the guard must enforce any limits on amendments, such as never reaching into protected paths. The hash-bound approval is a default, and the write guard is an earn-it component. Together they form the plan lock.

Unattended agents fail in two opposite ways. Some stall on questions nobody can answer. Others push through questions they should have asked. A missing requirement, compliance included, stops work for the owner before any fallback, and so does a question that could destroy data. The code guards of section 7 also stop destructive actions. What if stopping would strand a whole unattended run? Another design builds the largest safe subset and marks each gap in the code and the change's open items. It never guesses on behavior that could destroy data. Which approach works better has not been compared.

Other questions climb a decision ladder one rung at a time, as Figure 5.1 shows. The ladder itself is an open position.

**Figure 5.1. The decision ladder**

![Figure 5.1. The decision ladder: a question that is not a missing requirement or a risk to data climbs from the project's own records, to an independent panel, to the safest reversible option, and every outcome becomes a decision record]({{ "/assets/img/figure-5-1-the-decision-ladder.png" | relative_url }})

The first rung consults the project's records: the spec, earlier decisions, conventions and code. A confident answer stops there. The second rung asks a panel, either models in fresh sessions or people. Each member answers independently, so no voice steers the rest. A split panel sends the question to the third rung: the safest option reversible in code and in effect, since reverting code doesn't unsend a message. That choice gets flagged as low confidence for the next review.

The unattended path fails closed. A question meant for an absent owner is refused or halts the run. When low-tier work skips a person's plan approval, as section 9 describes, the approval is labeled as not made by a person.

Every unattended or deferred decision becomes a decision record that travels with the change. It states the question and answer, the rung that found it, a confidence showing where to look hardest, and any dissent.

A deferral is an item postponed instead of settled. Its record names the item, why it waits and its owner. The record also names a revisit trigger, the date or event that reopens it. A register of these records can become a graveyard unless each trigger fires. So adopt one only when deferred items would otherwise get lost. Section 12 lists it with the other open positions.

## 6. Execution: one loop, one workspace, one context budget

By default, the supervisor gives each unit of work one agent in its own workspace and branch, as section 3 sets out, which moves conflicts to merge time.

Why not run many agents at once? Fan-out, running agents in parallel, spends far more tokens. Tokens are the priced units of text a model reads and writes. In one vendor's multi-agent research system, a single agent used about four times the tokens of a chat, and a multi-agent run about fifteen times. Fan-out also invites collisions. A child session, started by another session and often called a subagent, makes small choices without seeing its siblings. Those choices clash at merge. Fan-out should wait for three conditions.

The first is independence. Planned file sets, lock files and migration numbers included, must not overlap in the plan or in the changes. Two units adding migrations will claim the same number. The merge step sends conflicts back to a coding agent instead of resolving them, because mechanical merging can't judge intent.

The second is cost: the time saved must justify the extra tokens. The third is absorption: concurrency should follow review capacity, not machine capacity. No team was found that caps it that way. One practitioner thread put a practical ceiling at about five to ten active sessions and named review as the limit. Section 10 sets nesting and spawn limits, but they cap only the worst case.

What about a dependent unit? By default, it starts after the earlier one merges. A stacked change instead branches from the earlier branch, so its pull request shows only its own changes. The stack merges bottom up. Revise a lower change, and every change above it gets marked stale, updated and verified again. Stacking, within one planned change or across several, must earn its place.

A dependent session begins with a structured handoff, because a prose summary drops decisions and constraints the successor must respect. The handoff carries the earlier unit's decision records, amendments and deferrals, its effect journal of each external effect the unit caused, and a path to its full trace.

Retrieval on demand protects each agent's context budget, the third axiom. The agent loads a file by path only when needed, since loading everything crowds out the task. Progress notes outside the window let a new session resume when a long-running agent stops short. A child session for a subtask starts with a clean window.

Compaction can drop early rules, so lasting rules belong in files the harness re-injects or, better, in code. Restating standard practice in an always-loaded file wastes the budget. So only a short list of non-standard conventions stays loaded, and the rest become skills. A skill is guidance meant to load when the task touches what it covers, such as a given path, so the agent pays mainly for relevant guidance. Whether a skill loads can rest on the model's own judgment, and the benefit of loading on demand has not been measured.

## 7. Control and containment

What keeps an agent from doing harm? Enforcement stops a known-bad action as the agent attempts it. Containment limits what an agent can reach, so a tricked agent does less harm. Table 7.1 shows where both sit among five layers of defense.

**Table 7.1. Layers of defense**

| Layer | Its job | Typical mechanisms |
|---|---|---|
| Prevent | limit what a session can reach | sandbox, closed network, narrow credentials |
| Decide | approve or refuse a proposed step | plan approval, confirmation |
| Enforce | block known-bad actions as they happen | hooks, permission backstops |
| Detect | notice afterwards | monitoring, audit logs |
| Recover | undo damage | revert, staged rollout, backups |

Containment is the prevent layer. The guard is the enforce layer.

Code must stop the most destructive actions: forced pushes that rewrite shared history, protected-branch deletion and shell commands that wipe a workspace. Guards also refuse overwrites of protected paths, some holding the policy that governs the agent. They refuse edits outside the approved change surface, and writes that look like secrets, since a committed credential may stay exposed after it's deleted. They block installing a tool server that isn't on an approved list until a person confirms it. Where an agent should only read a database, they refuse any statement but a single read.

The best guard acts early and says why it refused, so the agent picks a safer route. String matching catches accidents, but a determined or injected agent can rephrase the command. That's why a permission backstop sits behind the guard, such as a database role that can't drop shared tables. Treat a shared stateful environment as production. In one public report, an agent deleted a production database where development, staging and production weren't separated.

What happens when a guard hits a timeout or an error? If it fails open, it allows the action. So guards fail closed. In one carefully built guard set, most let the action through on malformed input or a missing configuration, and only a few failed closed. Don't assume any hook stops an action. Block-on-failure covers only some hook types and events, so list the uncovered ones. Retry a transient error before counting it as a refusal. Where a hook can't fail closed, branch protection or a wrapper with its own timeout enforces the rule.

An automatic reviewer is a model that judges whether each action is safe. It denies when its own review fails, but it can't stand alone. In one vendor's own test on a small set of real actions beyond what the user asked, it missed about one in six.

Each guard has a conformance probe, a deliberate attempt at the blocked action after every configuration change. The probe aims at predictable bypasses: a shell command that edits a file past a guard watching only the editor, or one file under several names. Others are a guard never registered, a missing setting that leaves a permissive default, a fallback to a looser parse, and a scope from an earlier plan that nothing clears. Several known bypasses need deliberate setup. So such guards mainly stop a careless or over-eager agent, not a determined one. When a rule must be bypassed on purpose, a named person grants a recorded, time-limited exception, so nobody silently switches a guard off.

A control that isn't enforcing still looks like protection. So every control shows its state where operators look: enforcing (it blocks, as a passing probe proves), advisory (it only reports) or off. In one system read, nearly every feature flag shipped off. In the same system, a budget documented as a hard ceiling only computed a state and left enforcement to its callers.

Containment answers prompt injection. In one preprint study, injected instructions got two coding agents to run attacker-supplied commands in up to 84% of attempts. A published proof of concept leaked private repository content after an agent read a malicious issue in a public repository. A serious leak needs all three: reading private data, reading content an attacker could have written, and sending information out. It's like a fire, which needs fuel, heat and air. Every session lacks at least one, which closes that path, and each sees only one repository.

A virtualization boundary doesn't filter network traffic, so outbound traffic is limited to a short approved list. A package mirror, the usual exception, must never forward other requests, or stolen data could leave inside the names requested. Continuous integration runs on agent branches without deployment secrets. Reviewer sessions that read pull-request text face injection too.

An attacker who steers the agent can do whatever it can. That's why credentials should be as narrow as the task allows. The agent pushes only to its own new branch or the pull request it was asked to work on, and holds no credential that lets it push directly. Organization policy outranks the repository. No committed file may switch on a permissive mode, or any pull request could widen permissions. Least agency extends this to tools: an agent gets only the tools its task needs, each narrowly permitted.

The decide layer has two parts. One is plan approval, which section 5 covers as part of planning. The other is the human gate, and section 9 shows where to spend that attention. The enforce layer also covers dependency changes, which bring in code the team didn't write. The detect layer only notices, so monitoring is never protection. The recover layer reverts, stages rollouts and keeps backups beyond the agent's reach. In one disputed public report, the backups sat on the volume the agent deleted.

## 8. Verification: checking the checks and the checker

An oracle means something only if it would fail on wrong work. It loses that power when the answer leaks into the task or the tests are too weak. How bad can a leaky oracle get? One study of a public benchmark, not yet peer reviewed, checked. About a third of one agent's passing patches had the solution in the task. About a third more were flagged as passing on weak tests. Take them out and its resolve rate falls from about 12.5% to about 4%.

The author never alters its own checks. You verify that the control protecting the tests is enforcing, because in one system it shipped switched off. Seeded defects and mutation testing test whether the checks notice. Think of it as holding a match under a smoke alarm.

A known-bad copy is the change with one behavior the spec values removed. If tests still pass on it, they're blind to that behavior. Copies target the most valued behaviors and commonest defects. A separate session that can't see the code writes a test that fails on the copy, so the test can't adapt to it. Without a meaningful known-bad copy, a person reviews the tests.

Reviewers start in a fresh context, free of the author's assumptions. Ask a reviewer for problems and it nearly always reports some. So findings carry severities, and the loop has a stop rule. The default pairs deterministic checks with one strong reviewer. It adds lenses, reviewers focused on one concern, only where recorded misses demand them.

No study comparing one strong reviewer with several lenses was found. On one independent preprint benchmark of general code review, the best system caught only about a third of known defects. No independent precision figure was found for any commercial reviewer. Code, not a model, collects the findings.

Why not let a single model judge, a model that grades work, decide? Because judges tend to favor longer answers and certain prompt positions, and can accept plausible but wrong claims. So no team should rely on one. In general judging tasks, a panel of smaller models from different makers, whose habits differ, has judged better than one large model. You check the judge against human verdicts on a sample, and it may say it can't tell. In one code-review benchmark, only one of three model judges was checked against people. It matched the human majority on about 72% of 60 comments.

A verdict function maps findings to a verdict by fixed rule, not vote, so no argument can change it. A blocker stops the change and can never be excused. A major finding is a real problem only the owner may waive. A minor finding gets recorded but stops nothing.

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

Unknown severity counts as major so uncertainty can't lower the bar. An internal error gets retried, then escalated, never accepted. Accept means ready for the human gate, not merged. An accept verdict carries its evidence: tests run and results, a diff audit, the commit reviewed, a recheck of each fix, and lint output.

Planted text can persuade a reviewer. In one study, text in pull request metadata misled automated review pipelines in 32 of 33 known vulnerability cases. So a model reviewer may tighten a verdict but never override a failed deterministic check or excuse a protected-area edit. A waiver is the owner's recorded acceptance of one major finding for one change. You count waivers so the cost of attention stays visible.

A review-fix loop gets a round cap and a progress test. It's stuck when a round clears nothing, a fixed finding returns, or a finding survives several rounds. A stuck loop restarts in a fresh session carrying only the plan and current findings. If it sticks again, it goes to a person. In one vendor report, the 8.5% of its pull requests that ran past five review rounds used 47.4% of its review-loop spend. Rounds that each fix one instance and find another signal a structural problem, which the owner fixes as a class by permitting only known-safe cases.

Silence is not success. A preprint study looked at 95 failed multi-agent runs. Two caveats: most were non-coding tasks, and a single model did the labeling. In 68 runs, evidence against the error was present. In 60 of those, no agent acted on it. So checks run by rule, not at the agent's discretion. A crash or missing check is never a pass, because a failing part can produce confident invented output. In one harness, a content-filter crash was followed by a final answer no agent wrote, in 25 of 58 hand-built runs. A flaky check never gets skipped. You quarantine it and give it an owner.

## 9. The human gate: spending attention where judgment lives

Where does the time go before a change merges? In one measured system, most of the delay was waiting for a person. Human attention is the scarcest resource. Multiplying change multiplies review habits. And skipped review is as much a failure as slow review, for humans too. One large public pull-request dataset shows how common skipped review is. In it, 58% of agent-authored and 75% of human-authored changes had no recorded review.

Approving every action can numb people into not reading. So attention goes to three judgment points: plan approval before effort is spent, change acceptance before merge, and promotion to production.

How much does each point demand? The risk tier sets that. Rules compute it from the planned change surface and recheck it on the final diff, so a narrow plan can't understate a change. A protected path forces the high tier. A change to a control takes at least the tier of what it protects. A dependency change raises the tier. Anything unclassified is high. A model may raise a tier but never lower it.

The high tier needs five things. The owner approves the plan. The owner reads the explainer, a summary for the reviewer, before the diff. A second engineer approves. Promotion is separate. And every rule meant to enforce is on. Lower tiers may run some controls advisory, but destructive-action guards are on at every tier. The medium tier needs owner plan approval and acceptance.

At the low tier, no person approves the plan. A preset rule may accept the change when checks pass, a fresh-context reviewer finds nothing and the change touches no protected path. Random human sampling keeps it honest. Each organization decides whether such a rule is allowed at all.

Approve plans in batches and cap new work at what reviewers can clear. These tiers and rules are reasoned positions. The nearest published practice comes from one vendor that adopted risk-tiered review. Since then, about three quarters of agent-written pull requests in its codebase merge without a person correcting the agent or discussing the engineering. That's a count of corrections, not of whether anyone looked. Its policy also names six change types that always get a human check.

The explainer summarizes the change. It gathers decision records, deferred items and minor findings, and links test evidence into the diff. It shows where to look rather than arguing correctness, because the author's confident account can sway a reader. Line-by-line reading can miss architecture, so a repository-level architecture view should go with it.

The requester states the need. The author is the agent plus its dispatching engineer. The owner, usually that engineer, is accountable. The second approver has no part in the work. The merger is neither author nor requester, or is an explicit low-tier policy. Nobody approves a change they hold a stake in. So the requester confirms the criteria fit the need but never approves the change. At the medium tier, computed checks separate doing from accepting. The high tier adds a second approver because the owner is usually part of the author.

You should also control what reaches review. Cap diff size. Keep batches small to guard against the instability of fast change. Cap concurrent work at reviewer capacity, and cap open changes per contributor. Ask contributors to disclose agent use, and have a person verify each contribution. Decline very large changes without design discussion.

A named owner, the last line of defense, is meant to understand and answer for each change. Consider one large developer survey. About one respondent in five said AI had cost them confidence in their own problem solving. So you have to keep ownership deliberately, and whoever submits a change should own it. An editable author label records origin but controls nothing. That's why every agent change should carry a service identity that the system owns and nobody can remove.

## 10. State, durability and cost

Saved state, which every stage writes, lets work survive a crash. But a crash can leave an outside action half done. So each effect needs a key that makes retrying it harmless. Platforms that survive crashes leave this to you. Replaying their journal of completed steps skips the finished ones. Restarting the interrupted step repeats its effect. So replay suits agent work better, but it still needs keys.

The key is section 2's content key. You build it from the effect's target and intended action, so a recovered attempt finds the effect already done. Tool-call identifiers change on retry, and run identifiers on a fresh run.

The run writes an intent record before each effect and a done record with its result after. Together they form the effect journal, which section 6's handoff passes on. Intent without done is an unknown state. So the resumed session probes the real system and halts for a person if the probe is inconclusive. The journal ships on by default, since protection that ships off protects no one.

Sometimes a slow session wakes after another has taken its run. Two things reject it. One is a lease, a time-limited claim naming the run's current owner. The other is a fencing token that rises with each new holder. Both matter only when several processes can own a run. Early on, you can instead check whether an expected branch or similar exists before creating it, and keep intent and done records only where the target doesn't drop duplicates itself.

Resume fails loudly (Axiom 6). When it finds state it doesn't recognize, it stops and reports instead of guessing. That's how it fails closed. A dirty working tree, with uncommitted changes, means something acted off the record. A damaged record can't be trusted, and neither can a recorded commit missing from the branch's history. Two mismatches reroute the run instead. A changed plan hash sends it back to planning (section 5). A branch past the last recorded commit sends it back to review, since saved state never recorded that work. Starting fresh silently would hide each mismatch.

Stored state is also a trust boundary. A checkpoint is the saved state that resume reads. A later session with different trust acts on whatever it contains. So no checkpoint may hold an authority token, credential or approval that grants power the reader was never given. Traces, journals and evaluation sets can hold personal data and secrets. So they get retention and access rules, redaction, and a secret scan before another session reads them.

What happens when helpers start helpers? They can multiply a small plan and its cost. One public issue report on an agent tool describes five planned helpers that became more than 360. Within minutes they drained a five-hour usage quota from about 3% to 100%, before the user stopped them by hand.

Limits come first: on nesting depth, and on how many sessions one session starts. Then comes spend accounting that survives resume and counts child sessions, since a reset count resets the protection. A cap can overshoot by one response plus helpers still running. And one agent toolkit defaults its budget and turn limits to none. So set both, leave headroom, and count every billed token type, cache reads included. In one public report, a long run used 819.5 million cache-read tokens against 11.3 million output tokens. The user suspected its caps never fired because the meter counted output only.

The supervisor of section 3 enforces these defaults. A complete refusing cap, an open position (section 12), also blocks new work at a safe point between effects, since an unread warning stops nothing. It warns early and labels estimates advisory. A runaway chain stops gracefully at a machine-readable refusal that names the limit. It shows the owner the spend and chain, so any raise is deliberate.

## 11. Observability, measurement and learning

The feedback layer takes section 10's records and turns them into change. When something goes wrong, the way you work changes too, so you aren't left with beliefs nobody has tested. The layer's runtime loop acts on events as they happen. Its evaluation loop grades the system on tasks with known answers.

Every runtime event becomes a structured record, and its failure class chooses the response. The system retries transient failures, such as timeouts. Deterministic ones recur identically and get a drafted fix for a person to apply. Those that need a person, such as a missing credential, notify someone and stop. Self-repair rests on tracing and stays limited to retries and drafted fixes. That's an open position, listed in section 12, that no system examined shows working in practice.

In the preprint study of 95 failed runs from section 8, over half traced to the plan, the brief (the task statement an agent receives) or the stopping rule. So what does restoring the agent's state do? It replays the flaw, so repair revises the plan. An alert that can never fire protects nothing, so a monitor must be live.

Scores must come from real runs, you must record them, and the model judge must agree with human opinion. The grader, whatever scores a run, should pass the reference answer graded against itself. But that sanity check is not a score. In one system, five of six recorded scores were such self-checks, saying nothing about the agent.

Where should evaluation tasks come from? Your team's own reverts, fixes and incidents, kept private. One vendor suggests starting with 20 to 50 such tasks. That set reveals only large differences, so you refresh and grow it. Read transcripts before trusting a score, and run each task several times, since output varies.

The grader deserves the agent's scrutiny. By one vendor's account, fixing grading bugs and ambiguous tasks in a public research-reproduction benchmark raised its own model's score from 42% to 95%. And a known-bad copy, built as section 8 describes, reveals a broken oracle if it passes.

Outcomes matter more than merge rate, which is easy to game. Acceptance and rework rounds, the fixes before acceptance, show whether reviewers take a change and at what cost. Reverts expose changes that only looked right. Post-merge incidents expose what review missed.

Other measures track attention. Review wait, the time a finished change waits for a person, shows where people limit throughput. Review depth, the time and sampled audit findings per change, shows whether review is real. Human touches, occasions needing a person, show where attention goes. False rejects and denials show how often a check or permission blocks good work. Cost per change counts child sessions too.

For a set period after merge, reverts and incidents count against each change's service identity, the one section 9 says nobody can remove. A periodic outcome report gathers them, with a named reader and schedule.

A recurring finding becomes a deterministic check on every change touching that area. Whoever wrote the check, it must prove it can fail before it blocks. Scheduled jobs with no model watch section 5's deferral triggers.

The harness is code under change too. Version its prompts, hooks and configuration for review and rollback. Each harness change should pass the private tasks before reaching everyone.

One vendor advises deleting preference-only hooks and advice newer models no longer need. You could test a guard against a weakness newer models may lack, using private tasks that invite the blocked action. If the model no longer attempts it, a person may remove that long-quiet guard. This untried pruning test is an open position. Section 12 names the controls exempt from it, and why.

## 12. Putting it together: a reference process, a confidence ladder and an adoption path

So what does the whole thing look like end to end? Figure 12.1 joins sections 4 to 11 into the reference process.

**Figure 12.1. The reference process**

![Figure 12.1. The reference process: a request moves from triage through specification, planning, execution, verification and the human gate to merge and learning, with exits for unsuited work, invented requirements and stuck review-fix loops, a light path that rejoins at execution, and loops for rejection and learning]({{ "/assets/img/figure-12-1-the-reference-process.png" | relative_url }})

Follow a change through it. If it's rejected, it goes back to execution through a capped fix round. If it's accepted, it meets the human gate for its risk tier. Then a person or an explicit policy merges it, never the author. The dashed line on the right carries learning back: recurring findings become new checks.

The three statuses of section 3 form the confidence ladder. One builder of an unattended system put it this way: unattended safety comes from guardrails, not parallelism. Most of that system's limits, though, were instructions to the model. The recommended defaults are those guardrails, the axioms of section 2 in practice, and they're the ones to adopt now.

The earn-it components add scale or structure. You adopt each one once your own runs show it pays. A graph supervisor and many parallel sessions add scale. Stacked sequencing and the plan lock's write guard add structure. Several review lenses and a design panel deepen scrutiny.

The open positions are worth trying, though they're unsettled. They are the decision ladder of section 5, the deferral register and tracing a change to its origin without a tracker item. The rest are the refusing cap, self-repair limited to retries and drafted fixes, and pruning controls that stay quiet.

Adoption runs in four levels. You start with a small, contained system, as starting simple and limiting the blast radius require. Measurement decides each move up. You may stop at any level. At each one, first configure what the platform, code host and continuous integration already provide.

Level 0 contains and measures. One agent works in a sandbox on one repository. It has minimum credentials and closed egress, and its backups sit beyond its reach. A person reviews every change, and ordinary tests and lint run. You move up once a private eval of your own failures and an outcome log exist.

Level 1 enforces. Guards fail closed, and probes confirm each one is enforcing. An effect journal records every external effect, so an uncertain resume stops loudly. Limits on spend, depth and fan-out arrive with a context budget. So do triage and checkable specifications. Small, low-risk work can now run under full review. Riskier work waits for Level 2. You move up once guards and journal run without silent failure.

Level 2 verifies and tiers. The sufficiency test arrives. The plan lock's write guard, with its amendment path, enters on trial. Verification gains known-bad copies and one fresh-context reviewer. You add lenses only when misses show a need, and the computed verdict decides. Risk tiers with caps on change size follow. You move up once review wait stops being the main constraint while reverts, incidents and audits hold steady.

Level 3 earns scale. Parallel sessions arrive, capped by available reviewers. Then come a graph supervisor, stacked sequencing and the open positions, each once runs show it pays. Base every move up on outcome records over time, never on one before-and-after comparison.

Table 12.1 lists anti-patterns, recurring failures such as controls that read stronger than they are. For each one it gives the principle the anti-pattern breaks and where to look first. It doubles as a pruning checklist, with three exceptions. You never prune destructive-action guards, containment or credential scope for being quiet, because a guard against rare catastrophe is meant to be quiet. They change only with the threat model, the account of what they defend against.

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

What does this paper's outside evidence consist of? Mostly vendor claims, product documentation or practitioner accounts. The measured studies are few. Several are preprints, and one rests mostly on non-coding tasks. The firmest ground lies under guidance that records costly failures. That guidance covers enforcement in code, approval alone being no defense, independent verification, human gates at judgment points, effects safe to repeat and controls that fail closed.

The first six axioms are well supported in their core claims, with two qualifications. Some frameworks favor explicit graphs over starting simple. And the case for review as the binding constraint rests mostly on vendor telemetry and surveys. Axioms 7 and 8 are firmer in what they reject than in what they prescribe. They clearly rule out public benchmarks and approval alone as a defense, as audits and attack studies have measured. Their replacements are private tasks with outcome records and contained sessions. Those rest on vendor and industry guidance rather than measurement. The statuses follow the same order. Support falls from recommended defaults to open positions, and earn-it components lack a comparison showing how much they help.

Three questions about review remain open. One is how often review comments are correct on everyday changes rather than selected examples. The others are how much each lens adds and what a review costs. Even recall varies widely across benchmarks, both above and below section 8's figure.

The design's most distinctive components are the least tested, and their field evidence is thin. What does that evidence amount to? A single multi-unit run, inferred from its still-unmerged pull requests. About six weeks of review timing from one system. And an evaluation apparatus whose few recorded scores were mostly self-checks. That's why you should test each control in your own system rather than assume that configured means working.

Each earn-it component has a test. Several review lenses, and design panels, pay only if they find what a strong reviewer misses at a worthwhile cost. The plan lock's write guard is worth keeping if it reduces drift without adding rework. Parallel sessions under a graph supervisor, and stacking many units, are justified only once review stops being the constraint.

Now the open positions. The decision ladder should be revised if flagged decisions get reversed so often that asking would have been cheaper. A refusing cap is confirmed when it stops a runaway and revised if it refuses legitimate work too often. The deferral register earns its keep if deferred items return instead of vanishing. Traceability without a tracker earns its keep if each change still leads back to its request.

Self-repair should grow beyond simple retries only where its repairs hold. Pruning quiet controls is confirmed if retired guards stay unneeded and revised if their failures return. Acceptance by policy remains an organizational choice, informed by each risk tier's incident and revert rates.

Your own records over many runs decide parallel sessions and risk tiers. But how to set and audit tiers, and when to prune a control that no longer earns its cost, remain open. Lenses, the write guard, the ladder and caps matter too rarely to judge without many runs. Adopt an open component deliberately, measure it, and keep it removable. An open position isn't a weak one. It marks where you should measure first.
