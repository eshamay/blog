---
layout: post
title: "Building AI Workflow Systems That Deserve Trust: Principles, Components and Processes for Agent-Driven Software Change"
date: 2026-11-22
description: Principles, components and processes for AI workflow systems that carry agent work from a request to a reviewed change.
---

## 0. Preface: what this paper is and how to read it

Many teams are building AI workflow systems at once. An AI workflow system is the machinery that carries an agent's work from a request to a reviewed change, and an agent is a language model that edits code and runs commands. Each system tends to be strong in some parts of the problem and thin in others, and a system that is thin in any part is incomplete. This paper therefore treats the parts as one whole and sets out the principles, process and components that make such systems trustworthy. A system deserves trust when its must-not-fail rules are enforced in code, its reach is contained, its checks can fail on wrong work, its records show what happened, and outcomes on the team's own tasks confirm that it works.

The scope is planned software change. Such work begins as a request and moves through planning, then execution, then review. It ends in a pull request, a proposed change that is reviewed before it merges into shared code. A person accepts or rejects it, or, for the lowest-risk work, a rule the organization set in advance decides. Conversational assistants fall outside, because a conversation ends in an answer rather than a reviewed change. Editor suggestions fall outside because they only propose lines as a person types. Agents that operate production systems are excluded because they act on live services instead of proposing a change.

The guidance explains the failure each rule prevents, and each term is defined where it first appears. Every reader should begin with sections 0 and 1. A newcomer gains the most by reading on in order through section 12, while a builder can take sections 4 to 12 one component at a time. A reader who first wants to know how firm the ground is can go from section 1 straight to section 13. The vocabulary begins with the smallest piece, a single agent.

## 1. From agent loops to workflow systems

The smallest unit within that scope is a single agent, a language model joined to a harness. The harness is the program around the model that gives it instructions and tools, such as a file editor and a shell. It runs those tools and returns the results. The agent works in a loop. It reads the task and code, then edits a file or runs a command. It observes the result and repeats until it decides it is done.

The harness, not the model, is the lever a builder controls. Its instructions say what matters, its tools set what the agent can touch, and its checks decide what counts as finished.

A session is one run of the loop. The model sees only its context window, the finite amount of text it can hold at once. When the window fills, the harness either compacts older text into a summary or truncates it. Whatever the harness drops is no longer available to the model, so a rule stated early in a long session may silently stop applying.

An agent can produce a plausible pull request in minutes, so the hard part has moved to checking changes. Checking needs an oracle, whatever decides that the work is correct. A test suite is the familiar oracle, and a type checker or reviewer also serves. Without one, an agent cannot tell finished work from work that merely looks finished.

Agents run unattended, with nobody watching, and many at once. Reviewers do not multiply that way, and every change still needs a person accountable for it, though for the lowest-risk work a rule set in advance may accept it while people sample its decisions. When output outgrows review capacity, changes queue, grow larger and are read less carefully, so more defects reach shared code. Faster production therefore does not make delivery reliable, because review capacity is often the binding limit.

A skilled contractor still works from drawings, under a permit and with an inspector, because skill does not replace process. A workflow is that process applied to an agent's work. A workflow system keeps it intact when nobody is watching. It consists of the runtime that runs the work, the controls that limit it and the records of what happened.

Two kinds of control carry the most weight. The first is deterministic enforcement, rules that code applies whatever the model decides. Its usual tool is a hook, code the harness runs at a fixed point, such as before a shell command, that can allow, question or refuse the action. A rule kept only in a prompt can be outweighed or lost to compaction, while a hook applies on every call it matches once it is registered, set to block when it fails and tested. The second is the human gate, a point where a person decides before work proceeds. Human gates belong at judgment calls, the decisions that need a person's grasp of purpose or risk. Placed at every step, they use up the attention those decisions need, and people can come to approve without reading. Section 2 states where each control belongs.

**Figure 1.1. From the scope of the paper to the single agent and the workflow system built around it.**

![Figure 1.1. From agent loop to workflow system: planned software change is in scope, a single agent is a model and a harness working in a loop, and a workflow system wraps that agent with a runtime, controls and records]({{ "/assets/img/figure-1-1-agent-loop-to-workflow-system.png" | relative_url }})

## 2. Eight axioms and three corollaries

Section 1 ended by asking where each control belongs. The eight axioms below answer it and set the paper's other premises. An axiom here is a working premise applied without re-argument, and each records a failure costly to learn twice.

**Axiom 1: enforce must-not-fail rules in code, and keep prompts for judgment.** A rule in the prompt is only advice, because the model weighs it against everything else in its context. Compaction can drop it, and injected text can argue the model out of it. Must-not-fail rules therefore belong in code that applies whatever the model decides. A hook refuses a dangerous action before it runs, and a deny rule refuses a whole class of action. Branch protection, a setting on the code host, keeps changes off a branch until required checks pass. Managed policy is organization settings that outrank any repository. Section 7 applies this.

**Axiom 2: start simple.** One agent comes before many, and a fixed pipeline, whose stages run in a set order, comes before a graph of sessions. In a graph a supervisor, which starts sessions and picks the next one, chooses at run time, so the system is harder to predict, limit and resume. Machinery should match the task, since safety comes from firm controls, not orchestration. Sections 3, 4 and 6 apply it.

**Axiom 3: context is a finite budget.** Every loaded instruction competes with the task for the model's attention. An always-loaded instruction file adds cost and, in one study, did not generally raise task success, though it helped where a project follows non-standard practices. An agent should therefore load only what its task needs. Section 6 applies it.

**Axiom 4: the author cannot weaken the checks that grade it.** An agent pressed to finish may edit a test until it passes instead of fixing the work. A test the author can rewrite proves little, so verification needs an oracle the author cannot change. At least one check should also stand apart from the author's context and, where possible, its model family, the models of one maker and lineage, because such models may share blind spots. Sections 8 and 9 apply it.

**Axiom 5: the human is the scarcest resource.** A team should treat review as the binding constraint, the step that limits how fast work flows, until its own records of waiting work show otherwise. Per-action approval, which asks a person to confirm each step, can decay into approval fatigue, where people approve without reading. A very high acceptance rate can signal that decay rather than care. Attention belongs on the plan and the finished change. Section 9 develops this.

**Axiom 6: every external effect is idempotent, and resume fails loudly.** A run resumed from saved state after a crash or timeout may repeat a step it already did. An external effect is any change outside the agent's workspace, so a repeated step can send a message twice or create a record twice. An idempotent effect gives the same result however often it is applied. The builder, not the resume machinery, must supply this. When saved state is uncertain, the run should fail loudly, stopping and reporting rather than guessing. Section 10 applies it.

**Axiom 7: judge the system by the team's own tasks and by outcomes.** A team should judge its system with private tasks drawn from its own work. Tasks everyone can see may already be known to the model, while private tasks are far less likely to have leaked, can be refreshed, and measure what matters. People's sense of their own speed is not a measure. Outcome data records what happens after merge, and together with private tasks it is the best available basis for trusting a system. Section 11 applies it.

**Axiom 8: limit the blast radius through isolation and least privilege.** The blast radius is everything a session could damage if it goes wrong. Isolation runs each session apart inside a sandbox, a boundary on what it can touch. Least privilege gives it only the credentials its task needs.

Prompt injection hides instructions in untrusted content, text from outside the operator, the person or team who runs the agent. An agent cannot reliably tell such text from real instructions and may obey it. Approval alone does not contain it, because a tricked agent still acts on everything it can reach. Containment, limiting that reach, is what holds. Section 7 applies it.

### Corollaries
Three corollaries turn the axioms into habits. The first is to **fail closed**, treating an error, a timeout or an unknown state as a refusal. A control that opens when broken fails exactly when it is needed.

The second corollary is to **record whether each control is armed or merely implemented**. An implemented control exists in configuration. An armed control is deployed and enforcing, and it has refused a deliberate test since its configuration last changed. A control can be present and still not work, and an untested one gives false comfort. In many existing systems, guardrails turn out to be prompts or conventions rather than code, and announcements often describe more than the code wires.

The third corollary is to **bind approvals and effects to content rather than names**. A hash is a fingerprint computed from content, and it changes exactly when the content does. A plan hash ties an approval to one version of the plan, so any edit voids it. A name can be reused, collide or stay fixed while content changes. A content key, built from an effect's target and intended action, lets a retry recognize an effect already applied and skip it.

With the premises and habits set, section 3 draws the map on which they are applied.

## 3. The anatomy of a workflow system

This section shows the parts that make the axioms hold. A request, whether an issue, a chat message or a larger body of work, enters a fixed pipeline of seven stages. A control plane sits over the stages, and two durable spines and a supervisor run beneath them, as Figure 3.1 shows.

**Figure 3.1. The anatomy of a workflow system**

![Figure 3.1. The anatomy of a workflow system: a request passes through seven stages under a control plane, with the state spine, feedback spine and supervisor beneath]({{ "/assets/img/figure-3-1-anatomy-of-a-workflow-system.png" | relative_url }})

Triage decides whether a task suits an agent and how much process it deserves. Specification turns intent into a spec, a written statement of what to build and what done means. Planning checks that the spec suffices and puts the plan to its owner, the person accountable for the work. After approval, the plan changes only through recorded new versions.

Execution runs one loop per unit of work, the piece of the plan one session carries to a change. Each unit has an isolated workspace, its own checkout and branch, so units never overwrite each other's files, and conflicts surface at merge instead. Verification runs deterministic checks, then fresh-context reviewers who start without the author's assumptions, and a fixed rule turns their findings into a verdict. The human gate places a person wherever the risk tier, a class computed from what a change touches, calls for one. Learning changes the system so a failure is less likely to repeat. The cheapest checks come first, so a person sees only work that has already passed them.

The control plane is a set of rules that watches every stage instead of belonging to one, because a dangerous action can happen anywhere. It is enforced in code and must be configured to fail closed, since many hooks fail open on a timeout or error by default. Its layers run from prevention to recovery, as section 7 develops, and each is separate: the sandbox and egress control prevent, approvals decide, hooks enforce and monitoring only detects. Guards, code that blocks a dangerous action as it is attempted, belong to it.

The state spine keeps what must survive a crash outside the conversation transcript, which compaction erodes, and away from loose files and labels, so work resumes instead of restarting. The feedback spine feeds events, evaluations, and outcome records back into checks.

The supervisor starts sessions and sequences units, and it holds spending caps across resumes so a restart cannot reset them. By default it runs one agent on one unit at a time. A graph supervisor, which picks the next session at run time, is harder to predict and resume, so it waits for a clear need.

Separation of duties means no party approves its own work, so the agent that does the work never gives the final check, acceptance or merge, and at the high tier a second person approves as well. This gives structure to the axiom that an author cannot weaken its own checks, as section 9 shows.

Each component carries one of three standings. A recommended default is adopted from the start, because the failure it prevents is serious and its cost low. An earn-it component works but is costly, so a team adds it when its own experience shows the need. An open position has unsettled value, so a team tries it as an experiment that stays easy to remove. Together the standings form the confidence ladder of section 12.

Sections 4 to 9 follow the stages, with section 7 on the control plane. Section 10 covers the state spine and the supervisor's limits, and section 11 covers learning and the feedback spine. The walk starts with triage.

## 4. Intent, triage and specification

A request has entered the pipeline, and triage asks two questions about it.

The first question is **whether to delegate**. Work suits an agent when its requirements are settled in advance and its result can be checked. Otherwise it goes to a person, or first becomes a specification, or spec, that makes success checkable.

The second question is **how much process to apply**. Risk should be judged by the property a change touches, such as permissions, not by its line count. Ceremony, the overhead of specifying and reviewing, should stay light on small changes, where it costs more than it returns and invites workarounds.

The light path serves small, low-risk work such as read-only reports and documentation. Its spec is a goal plus a done condition, the observable sign of completion, and the plan stays implicit. Verification is the automated checks plus one reviewer in a fresh context, and acceptance follows a rule set in advance.

The full path serves work that changes data, schemas or behavior others rely on. Its spec is a contract that a machine and a reviewer can both check, written before any plan so the plan answers it. It states success criteria, the checkable statements a finished change must satisfy. It names failure modes, so verification knows what to probe. It declares assumptions, so a reviewer can challenge them. It records blocking questions, whose answers the agent would otherwise invent.

Section 5's sufficiency test then confirms the spec is enough to plan from. The owner approves the plan, which changes only by recorded amendment. Sections 8 and 9 add stronger verification and owner review.

Triage picks the path from the kind of work, before any plan exists. The risk tier comes later, from what the change touches, and light-path work that touches a sensitive area moves to the full path.

When intent is unclear, the agent questions the requester until it can write the spec. Implementation then starts from the spec in a fresh session, free of the conversation's false starts.

A phase, such as planning, that only a prompt enforces can be skipped by the agent it constrains. A blocking check is code that refuses to proceed until the spec passes the sufficiency test. Full-path phases get blocking checks, while a prompt suffices on the light path, where a skipped step costs little.

A spec should say what must be true of the finished change, not how to build it. One that spells out the implementation has already made the design, so passing its checks proves little. One that states the required property leaves a real problem and something to verify.

Specs go stale, and agents rarely revisit early choices, so a spec needs a lifecycle and should stay short enough to review. Each delta, a mid-work change of course, merges into a living spec that always describes what is being built. A drift check flags where the spec or an agent's design notes diverge from the code. Before writing code, the agent searches for code that already does the job.

A change carries notes on how it was produced, since rerunning the same spec can yield different code. Each change links to the tracker item that requested it, which records its reason. The system never blocks work without one, since work arrives informally and a rigid rule breeds invented items.

A full-path spec now goes to planning, which first asks whether it holds enough to plan from.

## 5. Planning and deciding under uncertainty

Planning turns an accepted spec into an agreed plan through three steps. It tests the spec for sufficiency, wins the owner's approval, and then changes only by recorded amendment. A team can also draft the plan with a design panel, an earn-it component in which several model designers each write a plan from a different angle, such as risk first or simplest first, and a separate chooser scores the drafts and picks or merges one. Its benefit over a single planner has not been measured.

The sufficiency test asks whether proceeding would make the agent invent missing information. If so, the question is blocking, and work halts until the owner answers and the spec is updated. If not, it is a preference the agent decides and records. The test is itself a model's verdict, which code enforces and backs with a few fixed checks, such as whether done criteria and scope are stated. How often that verdict is right has not been measured, and a missing or unreadable verdict should count as insufficient. As a backstop, any question touching compliance, security or data retention always blocks, because a wrong guess there costs too much.

At approval the workflow records the plan hash, a fingerprint of its exact text, so approval covers precisely what the owner read. A resumed run whose hash no longer matches returns to planning. A separate check flags any edit not recorded as a new version. Both are the default. None of the public products examined binds plan approval to a hash, so this default is the paper's own position.

A team can go further, once its own runs show the need, and turn the plan's file list into a write guard, a rule in code that blocks writes outside the plan's change surface, the set of files the plan may touch. The guard is tuned not to get in the way. It blocks outright only paths the organization marks as protected. It lets generated files and dependency lock files through, since legitimate work changes them. Any other out-of-plan write becomes an amendment request.

A frozen plan fails because discovery changes the design. Reworking it, even by a supervisor mid-run, leaves no approved, versioned record. An amendment is a new version that states its reason and leaves a visible mark in the change. The owner approves it, and the write guard follows. Limits on amendments, such as never reaching into protected paths, must be enforced by the guard, because a limit written only in prose is advice. Together the hash-bound approval and the write guard form the plan lock. The hash binding is a default, while the write guard must earn its place.

Unattended agents fail in two opposite ways. Some stall on questions nobody can answer, while others push through questions they should have asked. Before any fallback, the agent asks whether a question is a missing requirement or could destroy data, and if so, work stops for the owner. Compliance questions count as missing requirements, and the code guards of section 7 also stop destructive actions. Where stopping would strand a whole unattended run, another design keeps going: it builds the largest safe subset, marks each gap in the code and in the change's list of open items, and never guesses on behavior that could destroy data. Which approach works better has not been compared.

Other questions climb a decision ladder, an open position, one rung at a time, as Figure 5.1 shows.

**Figure 5.1. The decision ladder**

![Figure 5.1. The decision ladder: a question that is not a missing requirement or a risk to data climbs from the projects own records, to an independent panel, to the safest reversible option, and every outcome becomes a decision record]({{ "/assets/img/figure-5-1-the-decision-ladder.png" | relative_url }})

The first rung consults the project's own records, meaning the spec, earlier decisions, the conventions, and the code. A confident answer is recorded and work continues.

The second rung asks a panel of models in fresh sessions, or of people, who answer independently so no voice steers the rest, and any dissent is recorded.

A split panel falls to the third rung, which takes the safest option reversible in code and in effect. Reversible in effect means undoable in the world, since reverting code does not unsend a message or unwrite data. The agent flags it as low confidence for the next review.

The unattended path fails closed. A question meant for an absent owner is refused or halts the run. Where low-tier work skips a person's plan approval, as section 9 allows, the approval is labeled as not made by a person.

Every unattended or deferred decision becomes a record that travels with the change. A decision record states the question and the answer, so a reviewer can judge it. Its rung shows how the answer was found. Its confidence tells the reviewer where to look hardest, and its dissent keeps disagreement visible.

A deferral is an item postponed rather than settled. Its record names the item, why it waits and its owner. It also names a revisit trigger, the date or event that reopens it. A register of these records is an open position (section 12). It can become a graveyard unless each trigger fires, so a team should adopt one only when deferred items are otherwise lost.

With the plan approved and its decisions on record, execution begins.

## 6. Execution: one loop, one workspace, one context budget

An approved plan splits into units of work, which the supervisor of section 3 dispatches. By default each unit gets one agent in an isolated workspace and branch, so no agent overwrites another's files while it works, and conflicts move to merge time.

Fan-out, running agents in parallel, spends far more tokens, the priced units of text a model reads and writes, and it invites collisions. A child session, a session started by another session and often called a subagent, makes small choices without seeing its siblings, and those choices clash at merge. The supervisor should fan out only when three conditions hold.

The first condition is independence. Planned file sets must not overlap, lock files and migration numbers included, since two units adding migrations will claim the same number. The check runs on the plan and again on the changes. A merge step with no model in it sends conflicts back to a coding agent, because mechanical merging cannot judge intent.

The second condition is cost, so the time saved must justify the extra tokens.

The last is absorption. Concurrency should follow review capacity rather than machine capacity, because unreviewed changes queue and go stale. A fixed session limit, set in section 10, caps the worst case without replacing this judgment.

A unit that builds on an earlier one starts, by default, after that one merges. If the earlier unit changes meanwhile, the later one is marked stale, rebased by replaying its changes onto the new version, and verified again. A stacked change instead branches from the earlier branch, so its pull request shows only its own changes, and the stack merges bottom up. Stacking, whether within one planned change or across several planned together, must earn its place.

A dependent session begins with a structured handoff, because a prose summary drops decisions and constraints the successor must respect. The handoff carries the earlier unit's decision records, amendments and deferrals. It also carries the effect journal, which records each external effect the unit caused, and a path to its full trace.

Each agent spends a context budget, the third axiom, because whatever falls out of its window is lost. Retrieval on demand protects that budget by loading a file through its path only when needed, since loading everything crowds out the task. Progress notes kept outside the window let a new session resume when a long-running agent stops short. A child session for a subtask starts with a clean window and the full budget.

Compaction, which summarizes old context to free space, can drop early rules. Rules that must last belong in files the harness re-injects or, better, in code. A thick always-loaded file restating standard practice wastes the budget, so only a short list of non-standard conventions stays loaded. The rest becomes skills. A skill is a bundle of guidance meant to load when the agent's task touches what it covers, such as a given path, so the agent pays mainly for relevant guidance. Whether a skill loads can rest on the model's own judgment, and the benefit of loading on demand has not been measured.

An agent in its own workspace still needs limits it cannot talk its way past, the control plane of section 7.

## 7. Control and containment

This section, like section 10, steps outside the stage order to the control plane over every stage. A guard enforced in code is a circuit breaker that trips whether or not anyone reads the warning label, and a prompt rule is only the label. Enforcement stops a known-bad action as it is attempted. Containment limits what an agent can reach, so even a tricked agent does little harm. Table 7.1 places both among five layers of defense.

**Table 7.1. Layers of defense**

| Layer | Its job | Typical mechanisms |
|---|---|---|
| Prevent | limit what a session can reach | sandbox, closed network, narrow credentials |
| Decide | approve or refuse a proposed step | plan approval, confirmation |
| Enforce | block known-bad actions as they happen | hooks, permission backstops |
| Detect | notice afterwards | monitoring, audit logs |
| Recover | undo damage | revert, staged rollout, backups |

Containment is the prevent layer, and the guard is the enforce layer.

Code must stop the most destructive actions. The first is a forced push, which rewrites shared history. A second is deleting a protected branch, the team's source of truth. A third is a shell command that wipes a workspace. Guards also refuse overwrites of protected paths, because some hold the policy that governs the agent. They refuse edits outside the approved change surface, which are work nobody agreed to. And they refuse writes that look like secrets, because a committed credential is permanently exposed.

The best guard acts early and says why it refused, so the agent picks a safer route. Behind it stands a permission backstop, such as a database role that cannot drop shared tables. Matching command strings catches accidents, but a determined or injected agent can rephrase the command. A shared stateful environment counts as production, because damage there reaches others.

A guard that fails open allows the action on a timeout or error, giving false comfort under load, so guards fail closed. In one carefully built guard set, most guards let the action through on malformed input or a missing configuration, and only a few failed closed. No hook is assumed to stop an action, since a block-on-failure setting covers only some hook types and events, and the uncovered ones are listed. Failing closed has a cost, so a transient error is retried before it counts as a refusal. Where one cannot fail closed, branch protection or a wrapper with its own timeout enforces the rule. An automatic reviewer, a model that judges whether each action is safe, denies whenever its own review fails. It helps as an extra layer but cannot stand alone, since in one vendor's own test on a small set of real actions that went beyond what the user asked, such a reviewer missed about one in six.

Each guard has a conformance probe, a deliberate attempt at the blocked action after every configuration change. Probes target predictable bypasses. A shell command can change a file and skip a guard that watches only the editor. One file can be reached under several names. A guard never registered never runs, and a missing setting may leave a permissive default. A guard may also fall back to a looser parse when it cannot split a command cleanly, and a scope recorded for an earlier plan may linger because nothing clears it. Several known bypasses need deliberate setup to exploit, so such guards mainly stop a careless or over-eager agent, not a determined one. When a rule must be bypassed, a named person grants a recorded, time-limited exception, so nobody silently switches the guard off.

A disarmed control still looks like protection, so every control shows its state where operators look. An armed control blocks, as a passing probe proves. An advisory control only reports, and one that is off does not run. In one system read, nearly every feature flag shipped off, and a budget its documentation called a hard ceiling only computed a state and left enforcement to its callers.

Containment, which section 2 set apart from approval, answers prompt injection, which is not hypothetical. In one preprint study, injected instructions got two coding agents to run attacker-supplied commands in up to 84% of attempts, and a published proof of concept leaked private repository content after an agent read a malicious issue in a public repository. Three conditions together make a serious leak possible. The agent can read private data. It reads content an attacker could have written. And it can send information out. Removing any one closes that path to a leak, so every session lacks at least one, and each sees only one repository.

A virtualization boundary alone does not filter network traffic, so outbound traffic is limited to a short approved list. The usual exception is a package mirror, a controlled copy of the dependency registry. It must never forward other requests, or stolen data could leave inside the names requested. Continuous integration runs on agent branches without deployment secrets, and reviewer sessions that read pull-request text face injection too.

Credentials should be as narrow as the task allows, because an attacker who steers the agent can do whatever it can. The agent pushes to a single branch, either a new one of its own or the pull request it was asked to work on, and holds no credential that lets it push directly. Organization policy outranks the repository, so no team quietly loosens its rules. No committed file may switch on a permissive mode, or any pull request could widen permissions. Least agency extends this to tools, so an agent gets only the tools its task needs, each with its own narrow permission.

The decide layer approves a step before it runs, through plan approval in section 5 and the human gate in section 9. The enforce layer also covers dependency changes, which bring in code the team did not write. The detect layer only notices, so monitoring is never protection. The recover layer undoes damage through revert and staged rollout, and keeps backups beyond the agent's reach, or an agent that destroys data could destroy them too.

Controls limit what an agent can do, and section 8 asks whether what it did is right.

## 8. Verification: checking the checks and the checker

A finished unit arrives at verification. A thermometer that always reads normal is worse than none, and a test suite that passes broken code is the same instrument. Verification asks whether the checks can detect failure and whether the checker is independent.

An oracle, whatever decides that work is correct, means something only if it would fail on wrong work. It fails when the answer leaks into the task, letting the agent see the expected result, or when the tests are too weak. In one study of a public benchmark, not yet peer reviewed, about a third of one agent's passing patches had the solution stated in the task, and about a third more were flagged as passing on weak tests; removing them cut its resolve rate from about 12.5% to about 4%. The author never alters its own checks, and a control protecting the tests must be verified as armed: in one system, the control meant to stop the coding agent editing tests shipped switched off. The checks are tested with seeded defects and mutation testing, which breaks code slightly to see whether tests notice.

A failing twin is a copy of the change with one behavior the spec cares about removed. Tests that still pass on it are blind to that behavior. Twins target the behaviors the spec values most and the most common defects. A separate session without access to the code writes a test that fails on the twin, so the test cannot adapt to the code. Without a meaningful twin, a person reviews the tests.

Reviewers start in a fresh context so they do not inherit the author's assumptions. A reviewer asked for problems always finds some, so findings carry severities and the loop has a stop rule, or review never ends. No single reviewer catches everything, so the default pairs deterministic checks with one strong reviewer. No study has compared one strong reviewer with several lenses, and on one independent preprint benchmark of general code review the best system caught only about a third of known defects, while no independent precision figure was found for any commercial reviewer. Lenses, reviewers focused on one concern, are added only where recorded misses demand them. Code, not a model, collects every reviewer's findings for the verdict function below.

A team should not rely on one model judge, a model that grades work. Such judges favor longer answers, favor answers by their position in the prompt, and believe confident but wrong claims. In general judging tasks, a panel of smaller models from different makers has judged better than one large model, because their habits differ. The team should check the judge against human verdicts on a sample and let it say it cannot tell rather than guess. In one code-review benchmark, only one of three model judges was checked against people, and it matched the human majority on about 72% of 60 comments.

A verdict function maps findings to a verdict by fixed rule rather than vote, so no argument can change it. A blocker is a defect that must stop the change and can never be excused. A major finding is a real problem only the owner may accept. A minor finding is worth recording but does not stop the change.

**Table 8.1. The verdict function**

| What the findings contain | Verdict |
|---|---|
| Any blocker | Reject |
| An unwaived major finding | Reject |
| Only minor findings | Accept, filing follow-ups under an owner |
| No findings | Accept |
| A finding of unknown severity | Treat as major |
| An internal error in a reviewer or check | Inconclusive |

Unknown severity counts as major because uncertainty must not lower the bar. An internal error is retried, then escalated, and never accepted, because a crash counts as failure. An accept verdict means the change is ready for the human gate of section 9, not that it merges.

The change under review may contain text meant to persuade its reviewer. In one study, text planted in pull request metadata misled automated review pipelines in 32 of 33 known vulnerability cases. A model reviewer may therefore make a verdict stricter but never override a failed deterministic check or excuse an edit to a protected area. A waiver, the owner's recorded acceptance of one major finding for one change, is counted to keep the cost of attention visible.

A review-fix loop, the cycle of findings and fixes, gets a round cap and a progress test. A loop is stuck when a round clears nothing, when a fixed finding returns, or when the same finding survives several rounds. A stuck loop first restarts the fix in a fresh session that carries only the plan and the current findings, and goes to a person if it sticks again, because more rounds waste attention. One vendor reports that the 8.5% of its pull requests that ran past five review rounds used 47.4% of its review-loop spend. When each round fixes one instance and finds another, the problem is structural, so the owner fixes the whole class by permitting only known-safe cases.

Silence is not success. An agent may notice its own failure and carry on. In one study of 95 failed multi-agent runs, mostly on non-coding tasks and labeled by a single model, evidence against the error was present in 68, and in 60 of those no agent acted on it. So checks run by rule, not at the agent's discretion. A crash or missing check is never a pass, because a failing part can produce confident invented output: in one harness, a content-filter crash was followed by a final answer no agent wrote in 25 of 58 hand-built runs. A flaky check, which passes or fails without the code changing, is quarantined under an owner, never skipped. The verdict then hands the change to the one resource that does not scale, a person.

## 9. The human gate: spending attention where judgment lives

After the verdict, the change meets the human gate. Human attention is the scarcest resource, and in one system whose timing was measured, most delay before a merge was waiting for a person. A system that multiplies change also multiplies whatever review habits it finds. The failure to design against is skipped review as much as slow review, and it is not specific to agents: in one large public dataset of pull requests, 58% of agent-authored and 75% of human-authored changes had no recorded review.

Approving every action can numb people until they stop reading. Attention belongs at three judgment points. Plan approval redirects work before effort is spent. Change acceptance lets a finished change merge. Promotion releases it to production.

The risk tier sets what each point demands and how strictly controls run. Rules compute it from the planned change surface and recheck it on the final diff, so a narrow plan cannot understate a change. A protected path forces the high tier. A change to a control takes at least the tier of what that control protects. A dependency change raises the tier. Anything unclassified is high. A model may raise a tier but never lower it.

At the high tier, the owner approves the plan and reads the explainer, a summary for the reviewer, before the diff. A second engineer approves, and promotion is a separate step. Lower tiers may run some controls advisory, warning without blocking, but here every enforcing rule is armed. Guards on destructive actions are armed at every tier.

At the medium tier, the owner approves the plan and accepts the change.

At the low tier, no person approves the plan. A rule set in advance may accept the change when checks pass, a fresh-context reviewer finds nothing and no protected path is touched. Human review of random samples keeps that rule honest. Each organization decides whether a rule may accept changes at all.

Since review is the limiting resource, plans should be approved in batches and new work capped at a day's review capacity.

The explainer opens with a summary of the change. It then gathers decision records, deferred items and minor findings. It closes with test evidence linked into the diff. It shows the reviewer where to look rather than arguing the change is correct, because the author's confident account can sway a reader. Line-by-line reading can miss architecture, so a repository-level architecture view should accompany it.

The requester states the need. The author is the agent plus the engineer who dispatched it. The owner, usually that engineer, is accountable. The second approver has no part in the work. The merger is neither author nor requester, or is an explicit low-tier policy.

Nobody approves a change in which they hold a stake, so the agent never approves or merges its own work. The requester confirms that the criteria capture the need but never approves the change. At the medium tier, computed checks separate the agent's doing from the owner's accepting. At the high tier the owner counts as author, so a second approver is needed.

Control what reaches review, not only review speed. A diff size cap keeps changes readable, and small batches resist the instability of fast change. Reviewer capacity caps concurrent work. Every contribution should disclose agent use and be verified by a person. Very large changes without design discussion should be declined, and open changes per contributor capped.

Ownership is the last line of defense: a named owner is meant to understand each change and answer for it. In one large developer survey, about one respondent in five said AI had cost them confidence in their own problem solving, so ownership must be kept deliberately, and whoever submits a change should own it. An editable label naming who made a change records origin but controls nothing. Every agent change should carry a service identity that belongs to the system and cannot be removed, so reviewers and audits can trust who did what.

All of this assumes the judged work survives crashes and restarts intact, which section 10 covers.

## 10. State, durability and cost

A durable record beneath the stages lets work survive a crash. Every stage writes to this state spine. A crash can leave an outside action half done for a retry to repeat. Each effect needs a key that makes a retry harmless, as a payment key prevents double billing.

Platforms that save progress across crashes do not make an outside effect safe to repeat. That job stays with the builder. Such a platform keeps a journal of completed steps. Replaying the journal skips the finished steps. Restarting the interrupted step instead repeats its effect. Replay suits agent work better but still needs keys.

The key should be section 2's content key, built from the effect's target and intended action. Tool-call identifiers fail as keys, because a retried call gets a new one, and a run identifier holds only while the same run is resumed, since a fresh run gets a new one. A content key lets a recovered attempt find the effect already done.

Each effect gets an intent record before it happens and a done record, carrying its result, afterward. These records form the effect journal that section 6's handoff passes on. Intent without done marks an unknown state, so the resumed session probes the real system and halts for a person if the probe cannot settle it. The effect journal is on by default, because protection shipped off protects no one.

A slow session may also wake after another has taken its run. A lease is a time-limited claim naming the current owner. A fencing token rises with each new lease holder, so actions carrying an older token are rejected. Both matter only when several processes can own a run. Early on, a team can instead check before creating something whether it exists, such as an expected branch. It keeps intent and done records only where the target does not drop duplicates itself.

Resume should fail loudly, as section 2's sixth axiom requires. On unrecognized state it stops and reports instead of guessing, which is how it fails closed. A dirty working tree, holding uncommitted changes, means something acted off the record. A changed plan hash means the approved plan is gone. A branch past the last recorded commit holds work the spine never saw. Starting fresh silently would hide each mismatch.

Stored state is also a trust boundary. A later session with different trust will act on whatever a checkpoint, the saved state resume reads, contains. A checkpoint must therefore never hold an authority token, a credential or approval that grants power the reader was never given. Traces, journals and evaluation sets can hold personal data and secrets. Retention rules limit how long each is kept. Access rules limit who reads it. Redaction strips what is sensitive. A secret scan runs before another session reads them.

Cost belongs with state. Helpers starting helpers can multiply a small plan enormously and exhaust a budget within minutes. In one public issue report against an agent tool, five planned helpers became more than 360, over seventy times the plan, and drained a five-hour usage quota from about 3% to 100% within minutes before the user stopped them by hand. Limits on how deeply sessions nest and how many one session starts come first. Spend accounting that survives resume and counts the sessions a session starts comes second, since a reset count resets the protection. A cap can overshoot by one response plus any helpers still running, and in one agent toolkit budget and turn limits default to none, so set both, leave headroom, and count every billed token type, cache reads included. In one public report a long run used far more cache-read than output tokens, and the user suspected its caps never fired because the meter counted output only. The supervisor of section 3 enforces these defaults. A complete refusing cap, an open position in section 13, goes further. It blocks new work at a safe point between effects, because an unread warning stops nothing. It warns early and labels estimates as advisory, so the owner raises the limit deliberately. That runaway chain of helpers then meets a machine-readable refusal naming the limit. The system stops gracefully rather than crashing, and the owner sees the spend and the chain behind it.

These records of what happened are also what learning builds on.

## 11. Observability, measurement and learning

The feedback spine turns the records of section 10 into change, so each bad outcome alters practice. Without it, a system holds only untested beliefs. Its runtime loop acts on events as they happen. Its evaluation loop grades the system against tasks whose answers are known.

Each runtime event is a structured record whose failure class chooses the response. A transient failure, such as a timeout, is retried. A deterministic failure, which recurs identically, gets a drafted fix for a person to apply. A failure that needs a person, such as a missing credential, notifies someone and stops. Self-repair, resting on tracing, stays limited to retries and drafted fixes.

Faults often start in the plan or the brief, the task statement an agent receives. In one preprint study of 95 failed multi-agent runs, mostly outside coding and labeled by a model, the fault lay in the plan, the brief or the stopping rule in over half. Restoring the agent's state then replays the flaw, so repair revises the plan.

A monitor must be armed, because an alert that can never fire protects nothing.

Having an evaluation is not having a trustworthy one. Scores must come from real runs and be recorded, and the judge, the model rating work, must agree with human opinion. The grader, whatever scores a run, should pass the reference answer when grading it against itself, but that sanity check is not a score. In one system, five of six recorded scores were such self-checks, so the record said nothing about the agent.

Evaluation tasks should be private and drawn from the team's own reverts, fixes and incidents. The set starts small, which reveals only large differences, and is refreshed. People read transcripts before trusting a score. Each task runs several times, since output varies. The grader deserves the agent's scrutiny. A failing twin from section 8 that passes reveals a broken oracle, the check that decides correctness.

Outcomes matter more than merge rate, which is easy to game. For quality, acceptance shows whether reviewers take a change, and rework rounds, the fixes before acceptance, show its cost. Reverts expose changes that looked right and were not. Post-merge incidents expose what review missed.

For attention, review wait, the time a finished change waits for a person, shows where people limit throughput. Review depth, the time and sampled audit findings per change, shows whether review is real. Counting human touches, occasions that needed a person, shows where attention goes. False rejects and denials show how often good work is blocked, by a check or a permission. Cost per change counts child sessions too.

To attribute outcomes reliably, each change carries a service identity, an author label the agent cannot alter. Reverts and incidents count against it for a set period after merge. A periodic outcome report gathers them, and the team names who reads it and how often.

A recurring finding becomes a deterministic check on every change touching that area. Written by a person or a drafted fix, it must prove it can fail before it blocks. Outcomes are recorded, and scheduled jobs with no model watch the deferral triggers of section 5.

Controls may also need to leave. One vendor advises deleting a hook that encodes only a preference and dropping advice newer models no longer need. A guard against a weakness newer models may lack could be tested with private tasks that invite the blocked action, and if the model no longer attempts it, a person may remove that long-quiet guard. This pruning test is untried, an open position in section 13.

Guards on destructive actions, containment and credential scope are exempt, because a guard against rare catastrophe is meant to be quiet. They change only with the threat model, the account of what they defend against.

Section 12 assembles every component into one process and an order of adoption.

## 12. Putting it together: a reference process, a confidence ladder and an adoption path

Sections 4 to 11 each built one component, and Figure 12.1 joins them into the reference process that carries work from request to learning.

**Figure 12.1. The reference process**

![Figure 12.1. The reference process: a request moves from triage through specification, planning, execution, verification and the human gate to merge and learning, with exits for unsuited work, invented requirements and stuck loops, and loops for rejection and learning]({{ "/assets/img/figure-12-1-the-reference-process.png" | relative_url }})

Earlier sections explain each stage. A rejected change returns through a capped fix round to execution. An accepted change meets the human gate for its risk tier, and a person or an explicit policy, never the author, merges it. The dashed line on the right carries learning back to the checks, because recurring findings become new checks.

These components fall into the three standings of section 3. Unattended safety comes from guardrails rather than from parallelism, and a guardrail kept only in a prompt is weaker than one in code. The recommended defaults are those guardrails, the axioms of section 2 put into practice, and a team should adopt them now.

The earn-it components add scale or structure, and a team adopts each once its own runs show it pays. A graph supervisor and many parallel sessions add scale. Stacked sequencing and the write guard of the plan lock add structure. Several review lenses and a design panel, a group that drafts competing plans for a chooser to pick from or merge, deepen scrutiny.

The open positions are worth trying, but their value is unsettled. They begin with the decision ladder of section 5 and the deferral register. A way to trace a change to its origin when no tracker item exists is another. The refusing cap and self-repair limited to retries and drafted fixes complete them.

Adoption proceeds in four levels. Starting simple and limiting the blast radius put a small, contained system first, and measurement decides each move up. A team may stop at any level. At each, it should first configure what the platform, code host and continuous integration already provide.

Level 0 contains and measures. One agent works in a sandbox on one repository, with minimum credentials and closed egress, and backups sit beyond its reach. A person reviews every change, and ordinary tests and lint run. The team moves up once a private eval of its own failures and an outcome log exist.

Level 1 enforces. Guards fail closed, and probes confirm that each is armed. An effect journal records every external effect, so an uncertain resume stops loudly. Limits on spend, depth and fan-out arrive with a context budget. Triage and checkable specifications begin here. Small, low-risk work can now run under full review, while riskier work waits for Level 2. The team moves up once guards and journal run without silent failure.

Level 2 verifies and tiers. The sufficiency test arrives, and the plan lock's write guard, with its amendment path, enters on trial. Verification gains failing twins and one fresh-context reviewer, adding lenses only when misses show a need, and the computed verdict decides. Risk tiers with caps on change size follow. The team moves up once review wait is no longer the main constraint, with reverts, incidents and audits held steady as it fell.

Level 3 earns scale. Parallel sessions arrive, capped by the reviewers available. A graph supervisor, stacked sequencing and the open positions follow, each once runs show it pays. Every move up should rest on outcome records gathered over time, never on one before-and-after comparison.

Most recurring failures, or anti-patterns, are controls that read as stronger than they are. Table 12.1 lists them, with the principle each breaks and where to look first. It doubles as a pruning checklist, though destructive-action guards, containment and credential scope are never pruned for being quiet.

**Table 12.1. Anti-patterns and how to notice them**

| Anti-pattern | Principle it breaks | Where to look |
|---|---|---|
| A disarmed control that looks protective | Armed versus implemented | Conformance probes |
| A destructive-work freeze only in a prompt | Enforce in code | The list of guards |
| Approval fatigue from per-action prompts | Attention is scarce | Approval rate |
| The author editing tests it did not write | Independent checks | Test changes, failing twins |
| A crash or warning-only cap treated as safe | Fail closed | Internal error count |
| Summary handoffs, thick always-loaded instructions | Context is a budget | Context cost per session |
| Fan-out without independent units | Start simple | Conflict rate, review queue |
| A resume that starts fresh or replays effects | Loud, idempotent resume | Intents with no done record |
| Merge rate as the quality signal | Measure outcomes | Reverts and incidents by tier |
| An advisory estimate labeled as a hard cap | Armed versus implemented | What happens when the limit is reached |
| Review skipped as load grows | Attention is scarce | Changes merged without review |
| Full ceremony on a small change | Start simple | Process effort against change size |

How firmly each of these recommendations is known varies, and section 13 says where.

## 13. Limits of the evidence and open questions

Not every recommendation in this paper is equally settled, and this section separates the firm from the open. Most outside evidence is vendor claims, product documentation or practitioner accounts. The measured studies are few, several are preprints, and one rests mostly on non-coding tasks. The firmest ground lies under guidance that records costly failures. Enforcing rules in code stands there, and so does the rule that approval alone is no defense. Independent verification and human gates at judgment points stand there too. So do effects that are safe to repeat and controls that fail closed.

The first six axioms of section 2, from enforcement in code through idempotent effects, are well supported in their core claims, with two qualifications. Some frameworks favor explicit graphs over starting simple, and the case for treating review as the binding constraint rests mostly on vendor telemetry and surveys. The last two, on judging a system by private tasks and on limiting the blast radius, are firmer in what they reject than in what they prescribe. They clearly rule out public benchmarks and approval alone as a defense, which audits and attack studies have measured. The replacements, private tasks with outcome records and contained sessions, rest on vendor and industry guidance rather than measurement, so they are less settled. The standings of section 3 follow the same order. The recommended defaults are settled, while the components that must earn their place lack a comparison showing how much they help. The open positions are the least settled.

Three questions about review remain open. The first is how often review comments are correct on everyday changes rather than selected examples. The second is how much each review lens adds. The third is what a review costs. The components that most distinguish this design are the least tested. Their field evidence is thin: a single multi-unit run inferred from its still-unmerged pull requests, about six weeks of review timing from one system, and an evaluation apparatus with no recorded scores. A team should therefore confirm by testing that each control works in its own system, rather than assume it works because it is configured.

Each component that must earn its place has a test. Several review lenses, each looking for one kind of defect, pay only if the extra lenses find what a strong reviewer misses at a worthwhile cost. Design panels answer to the same test. The write guard of section 5's plan lock is worth keeping if it reduces drift without adding rework. Parallel sessions under a graph supervisor, and stacking of many units, are justified only once review is no longer the constraint.

Among the open positions, the decision ladder should be revised if flagged decisions are reversed so often that asking would have been cheaper. A refusing cap, which blocks new work at its spending limit instead of warning, is confirmed when it stops a runaway and revised if it refuses legitimate work too often. The deferral register earns its keep if deferred items return rather than vanish. Traceability without a tracker earns its keep if each change still leads back to its request. Self-repair should grow beyond simple retries only where its repairs hold. Pruning quiet controls is confirmed if retired guards stay unneeded and revised if their failures return. Acceptance by policy remains an organizational choice, informed by incident and revert rates in each risk tier.

A team's own records over many runs decide parallel sessions and risk tiers, and parallel sessions also wait on review capacity. How to set and audit risk tiers, and when to prune a control that no longer earns its cost, remain open. Lenses, the write guard, the ladder and caps matter too rarely to judge without many runs. An open component should be adopted deliberately, measured, and kept removable. An open position is not a weak one. It marks where a builder should measure first.

