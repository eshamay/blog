---
layout: post
title: "Building AI Workflow Systems That Deserve Trust: Principles, Components and Processes for Agent-Driven Software Change"
date: 2026-11-22
description: Principles, components and processes for AI workflow systems that carry agent work from a request to a reviewed change.
---

## 0. Preface: what this paper is and how to read it

An agent is a language model that edits code and runs commands. An AI workflow system is the machinery that carries an agent's work from a request to a reviewed change. Many teams are building such systems, and each tends to be strong in some parts and thin in others, which leaves it incomplete. This paper treats the parts as one whole. It sets out the principles, process and components that make such systems trustworthy. So when does a system deserve trust? When five things hold: it enforces its must-not-fail rules in code, it limits what it can reach, it runs checks that can fail on wrong work, it keeps records that show what happened, and outcomes on the team's own tasks confirm that it works.

The scope is planned software change. A change begins as a request and moves through planning, execution and review. It ends in a pull request, which a person accepts or rejects. For the lowest-risk work, a rule the organization set in advance decides instead. What's left out? Conversational assistants, editor suggestions and agents that operate production systems, because none of them ends in a reviewed change.

This paper pairs each rule with the failure it prevents and defines each term at first use. Everyone should start with sections 0 and 1. After that, if you're new, read in order through section 12. If you're building, you can take sections 4 to 11 one at a time. If you want to know how firm the ground is, you can skip to section 13.

## 1. From agent loops to workflow systems

Start with the smallest piece: a single agent. It's a language model joined to a harness, which is the program around the model. The harness gives the model instructions and tools, like a file editor and a shell. It runs those tools and hands back the results. The agent works in a loop. It reads the task and the code, then edits a file or runs a command. It looks at what happened and goes around again until it decides it's done. You control the harness most directly, through its instructions, tools and checks.

One run of that loop is a session, and a session's memory is finite. The model sees only what fits in its context window, which holds a limited amount of text. When the window fills, the harness either compacts older text into a summary or truncates it. The model loses whatever gets dropped. So a rule you stated early in a long session can stop applying without any warning.

Now add scale. Agents run unattended and many at once. An agent can produce a plausible pull request faster than a person can review it. That moves the hard part to checking. Checking needs an oracle, which is anything that decides whether the work is correct, such as a test suite, a type checker or a reviewer. Without one, an agent can't tell finished work from work that only looks finished. Reviewers don't multiply the way agents do (section 9). When output outgrows them, changes tend to pile up in the queue, grow larger and get read less carefully. More defects can then reach shared code. A large industry survey describes AI as an amplifier of a team's existing strengths and weaknesses. Organizations with a strong internal platform saw a strongly positive effect on organizational performance, while those with a weak one saw a negligible effect.

Think of a skilled contractor. Skill doesn't replace process, so even a skilled one works from drawings, holds a permit and gets inspected. A workflow is that process applied to an agent's work. A workflow system keeps that process intact when nobody is watching. It has three parts: a runtime, controls that limit the work, and records of what happened.

Two kinds of control carry the most weight. The first is deterministic enforcement. Here code applies the rules whatever the model decides. It usually works through a hook. A hook is code the harness runs at a fixed point, such as before a shell command. It can allow, question or refuse the action. Unlike a prompt rule (Axiom 1), a hook runs on every call it matches. That holds only if the hook is registered, set to block on failure where the harness allows it, and tested. The second is the human gate. A human gate is a point where a person decides before work goes on. A gate at every step uses up the attention that judgment calls need. Judgment calls depend on a person's grasp of purpose or risk, so that is where the human gate belongs.

**Figure 1.1. From the scope of the paper to the single agent and the workflow system built around it**

![Figure 1.1. From agent loop to workflow system: planned software change is in scope, a single agent is a model and a harness working in a loop, and a workflow system wraps that agent with a runtime, controls and records]({{ "/assets/img/figure-1-1-agent-loop-to-workflow-system.png" | relative_url }})

## 2. Eight axioms and three corollaries

Each axiom records a failure that's costly to learn twice. This paper applies each as a working premise without re-argument.

**Axiom 1: enforce must-not-fail rules in code, and keep prompts for judgment.** A prompt rule is only advice. The model weighs it against everything else in its context. Compaction can drop it, and injected text can argue the model out of it. In code, hooks refuse recognized dangerous actions before they run, and deny rules refuse whole classes of action. Branch protection on the code host holds changes off a branch until required checks pass. Managed policy comes from the organization and outranks any repository. Section 7 shows these controls at work.

**Axiom 2: start simple.** One agent comes before many. A fixed pipeline comes before a graph of sessions. In a graph, a supervisor decides at run time which session starts next. That makes the system harder to predict, to limit and to resume. Firm controls provide safety, and orchestration doesn't. Sections 3, 4 and 6 show starting simple in practice.

**Axiom 3: context is a finite budget.** Loaded instructions compete with the task for attention. An always-loaded instruction file adds cost. One study found it didn't generally raise task success, though it helped where a project follows non-standard practices. So an agent should load only what its task needs. Section 6 shows how to spend that budget.

**Axiom 4: the author cannot weaken the checks that grade it.** Push an agent to finish and it may edit a test until it passes instead of fixing the work. So verification needs an oracle the author can't change. Models from one maker and lineage form a model family, and they may share blind spots. At least one check should also run in a context separate from the author's, and where possible it should use a different model family. Sections 8 and 9 apply this to checks and review.

**Axiom 5: the human is the scarcest resource.** Treat review as the bottleneck until your own records of waiting work show otherwise. Ask people to approve every action and they may start approving without reading. That is approval fatigue. A very high acceptance rate can signal fatigue instead of care. Attention belongs on the plan and the finished change. Section 9 shows where it goes.

**Axiom 6: every external effect is idempotent, and resume fails loudly.** After a crash or timeout, a run resumed from saved state may repeat an external effect, such as sending a message or creating a record a second time. An external effect is any change outside the agent's workspace. As the builder, you must make each effect idempotent. The resume machinery won't do it for you. When saved state is uncertain, the run should fail loudly. It should stop and report instead of guessing. Section 10 shows what this takes when a run dies halfway through.

**Axiom 7: judge the system by the team's own tasks and by outcomes.** Private tasks from your team's own work, plus outcome data on what happens after merge, are a sounder basis for trust than public scores. The model may already know public tasks. Private ones are less likely to have leaked, can be refreshed and measure what matters. In one randomized trial, experienced developers believed AI made them faster, but it slowed them down. So a sense of speed isn't a measure. Section 11 shows how to measure.

**Axiom 8: limit the blast radius through isolation and least privilege.** Run each session in a sandbox and give it only the credentials its task needs.

Prompt injection hides instructions in untrusted content. That's text from anyone other than the operator, meaning the person or team who runs the agent. The agent can't reliably tell it from real instructions and may obey it. Approval alone doesn't stop this, because a tricked agent still acts on everything it can reach. What works is limiting that reach. Section 7 shows how to build it.

### Corollaries

First, **fail closed**. Treat an error, a timeout or an unknown state as a refusal. A control that opens when broken fails exactly when you need it.

Second, **record whether each control is enforcing or merely configured**. A configured control exists in configuration. An enforcing control is deployed and working. It has also refused a deliberate test since its configuration last changed. An untested control gives false comfort. One review of existing systems found that most guardrails were prompts or conventions rather than code. It also found that announcements often promised more than the code actually did.

Third, **bind approvals and effects to content rather than names**. A plan hash ties an approval to one version of the plan, so any edit voids it. Names can be reused or collide, and they can stay the same while the content changes. A content key is built from an effect's target and intended action. With it, a retry can see that the effect was already applied and skip it.

## 3. The anatomy of a workflow system

A request enters a fixed pipeline of seven stages. It might be an issue, a chat message or a larger body of work. A control plane sits over the stages. Beneath them sit a supervisor and two durable layers: saved state and feedback (Figure 3.1).

**Figure 3.1. The anatomy of a workflow system**

![Figure 3.1. The anatomy of a workflow system: a request passes through seven stages under a control plane, with saved state, a feedback layer and a supervisor beneath]({{ "/assets/img/figure-3-1-anatomy-of-a-workflow-system.png" | relative_url }})

Triage decides whether a task suits an agent and how much process it deserves. Specification turns intent into a spec. A spec is a written statement of what to build and what done means. Planning checks that the spec is sufficient and then puts the plan to its owner, who is accountable for the work. From then on, the plan changes only through a recorded new version. Execution runs one loop for each unit of work. A unit of work is the piece of the plan that one session turns into a change. Each loop gets an isolated workspace with its own checkout and branch.

Verification runs deterministic checks first. Then fresh-context reviewers look at the work. These reviewers start without the author's assumptions. A fixed rule turns their findings into a verdict. The human gate puts a person in the loop wherever the risk tier calls for one. The risk tier is a class computed from what a change touches. Learning changes the system so a failure is less likely to repeat. The cheapest checks come first, so a person sees only work that has passed them.

The control plane watches every stage, because a dangerous action can happen anywhere. Code does the enforcing. In at least one widely used harness, a hook that times out or errors fails open by default. You must configure the control plane to fail closed. Its separate layers run from prevention to recovery. Table 7.1 in section 7 lists them. Guards belong to the control plane too. A guard is code that blocks a dangerous action as it's attempted.

Saved state holds what must survive a crash. It lives in structured, checkpointed records rather than in notes or labels alone. It also lives outside the transcript, which compaction erodes. That lets work resume instead of restart. The feedback layer feeds events, evaluations and outcome records back into checks. The supervisor sequences units and keeps spend counts across resumes (section 10 shows why). By default it runs one agent on one unit at a time. A graph supervisor waits for a clear need.

No party approves its own work. The agent that did the work never gives the final check, acceptance or merge. At the high risk tier, a second person also approves. Section 9 sets out what puts a change there.

Each component has one of three statuses. A recommended default is one you adopt from the start, because the failure it prevents is serious and the component costs little. An earn-it component works but costs a lot, so you add it once your own experience shows the need. An open position is one whose value is unsettled, so you try it as an experiment you can easily remove. Together the three statuses make up the confidence ladder, which section 12 fills in component by component.

## 4. Intent, triage and specification

Triage first asks whether an agent should do this work. Work suits an agent best when its requirements are settled and its result can be checked. Otherwise it goes to a person or first becomes a spec that makes success checkable.

Next, triage asks how much process to apply. What drives the risk? What a change touches, such as permissions. Its line count doesn't. Specifying and reviewing carry overhead, and on small changes that overhead costs more than it returns, so keep it light there.

The light path serves small, low-risk work such as read-only reports and documentation. Its spec is a goal and a done condition. A done condition is the observable sign of completion. The plan stays implicit. Verification uses automated checks and one fresh-context reviewer. Acceptance can follow a rule set in advance, and section 9 says when.

The full path serves work that changes data, schemas or behavior others rely on. Its spec is a contract that both a machine and a reviewer can check. You write it before any plan, so the plan answers it. It lists the success criteria a finished change must satisfy, each one checkable. It also lists failure modes for verification to probe and assumptions for a reviewer to challenge. Finally, it lists blocking questions wherever the agent would otherwise have to invent an answer.

Triage picks the path from the kind of work, before any plan exists. Later, the risk tier moves light-path work that touches a sensitive area to the full path.

When intent is unclear, the agent questions the requester until it can write the spec. Implementation then starts in a fresh session, free of the conversation's false starts.

An agent can skip a phase that only a prompt enforces. So full-path phases get a blocking check. That is code that refuses to proceed until the spec passes the sufficiency test explained in section 5. The light path relies on a prompt. A skipped step costs little there.

A spec that spells out the implementation has already made the design, so its checks prove little. A spec should therefore say what must be true of the finished change, not how to build it.

Specs go stale, and agents can fail to revisit early choices. A spec therefore needs a lifecycle and should stay short enough to review. Work sometimes changes course in the middle. Each such change is a delta, and each delta merges into a living spec of what is being built. A drift check flags places where the spec or the agent's design notes no longer match the code. Before coding, the agent searches for existing code that does the job.

A change carries notes on how it was produced, since rerunning the same spec can yield different code. It also links to the tracker item that records why it was requested. A change without a tracker item, such as one that began as a chat request, still proceeds. Tracing it to its origin is an open position, listed with the others in section 12.

## 5. Planning and deciding under uncertainty

Planning tests the spec for sufficiency and wins the owner's approval. After that the plan changes only by recorded amendment. A design panel is another way to draft the plan, and it is an earn-it component. Several model designers each write a plan from a different angle, such as risk first. A separate chooser then scores the drafts and picks or merges one. The panel's benefit over a single planner has not been measured.

How do you know a spec is enough? The sufficiency test asks whether going ahead would make the agent invent missing information. If it would, the question blocks work until the owner answers and the spec is updated. If it wouldn't, it's a preference that the agent decides and records. A model gives the verdict, and code enforces it. Fixed checks back the verdict up, for example whether the spec states done criteria and scope. The test's accuracy has not been measured. A missing or unreadable verdict should count as insufficient. Questions touching compliance, security or data retention always block, because a wrong guess there costs too much.

At approval, the workflow records the plan hash. A resumed run whose hash no longer matches returns to planning. A separate check flags any edit that was not recorded as a new version. Both are defaults. They are this paper's own position, because none of the public products examined binds plan approval to a hash.

Once your own runs show the need, you can turn the plan's file list into a write guard. The guard is code that blocks writes outside the change surface. The change surface is the set of files the plan may touch. Outright blocks apply only to paths your organization marks as protected. Generated files and dependency lock files pass through, since legitimate work changes them. Any other write outside the plan turns into an amendment request.

A frozen plan can fail when discovery changes the design. In the products examined, a plan reworked mid-run needs no fresh approval. That holds even when a supervisor does the rework, and the new plan isn't bound to what the owner read. Here an amendment is a new version that the owner approves. It states its reason and leaves a visible mark in the change. The write guard follows it. A limit in prose is only advice, so the guard must enforce any limits on amendments, such as never reaching into protected paths. The hash-bound approval is a default and the write guard is an earn-it component. Together they form the plan lock.

Unattended agents fail in two opposite ways. Some stall on questions nobody can answer. Others push through questions they should have asked. A missing requirement, including a compliance requirement, stops work for the owner before any fallback applies. So does a question whose wrong answer could destroy data. The code guards of section 7 also stop destructive actions. What if stopping would strand a whole unattended run? Another design builds the largest safe subset. It marks each gap in the code and lists it in the change's open items. It never guesses on behavior that could destroy data. Which approach works better has not been compared.

Other questions climb a decision ladder one rung at a time, as Figure 5.1 shows. The ladder itself is an open position.

**Figure 5.1. The decision ladder**

![Figure 5.1. The decision ladder: a question that is not a missing requirement or a risk to data climbs from the project's own records, to an independent panel, to the safest reversible option, and every outcome becomes a decision record]({{ "/assets/img/figure-5-1-the-decision-ladder.png" | relative_url }})

The first rung consults the project's records: the spec, earlier decisions, conventions and code. A confident answer stops there. The second rung asks a panel, either models in fresh sessions or people. Each member answers independently, so no voice steers the rest. A split panel sends the question to the third rung. There the workflow picks the safest option that can be undone. That means undone in effect as well as in code, because reverting code doesn't unsend a message. The workflow flags the choice as low confidence for the next review.

The unattended path fails closed. The workflow refuses a question meant for an absent owner or halts the run. When low-tier work skips a person's plan approval, as section 9 describes, the workflow labels the approval as not made by a person.

Every unattended or deferred decision becomes a decision record that travels with the change. The record holds the question and the answer, along with the rung that produced it and any dissent. It also holds a confidence level that tells reviewers where to look hardest.

A deferral is an item that is postponed instead of settled. Its record names the item and says why it waits and who owns it. It also names a revisit trigger, which is the date or event that reopens the item. A register of such records can turn into a graveyard unless each trigger actually fires. Adopt one only when deferred items would otherwise get lost. Section 12 lists it with the other open positions.

## 6. Execution: one loop, one workspace, one context budget

By default, the supervisor gives each unit of work one agent in its own workspace and branch, as section 3 sets out. That moves conflicts to merge time.

Why not run many agents at once? Running agents in parallel is called fan-out, and it uses far more tokens. Tokens are the units of text a model reads and writes, and they are what you pay for. In one vendor's multi-agent research system, a single agent used about four times the tokens of a chat. A multi-agent run used about fifteen times. Fan-out also invites collisions. A child session is a session that another session started, often called a subagent. Each child makes small choices without seeing its siblings, and those choices clash at merge. Fan-out should wait for three conditions.

The first is independence. Planned file sets must not overlap, in the plan or in the actual changes. The same goes for the lock files and migration numbers the units touch. Two units adding migrations will claim the same number. The merge step sends conflicts back to a coding agent instead of resolving them itself, because mechanical merging can't judge intent.

The second is cost: the time saved must justify the extra tokens. The third is absorption. Concurrency should follow review capacity, not machine capacity. No team was found that caps it that way. One practitioner thread put a practical ceiling at about five to ten active sessions and named review as the limit. Section 10 sets nesting and spawn limits, but they cap only the worst case.

What about a dependent unit? By default, it starts after the earlier one merges. A stacked change instead branches from the earlier branch, so its pull request shows only its own changes. The stack merges from the bottom up. When you revise a lower change, the system marks every change above it stale, then updates and verifies each one again. Stacking, within one planned change or across several, must earn its place.

A dependent session begins with a structured handoff, because a prose summary drops decisions and constraints the successor must respect. The handoff carries the earlier unit's decision records, amendments and deferrals. It also carries a path to the full trace and the unit's effect journal. The journal lists each external effect the unit caused.

Retrieval on demand protects each agent's context budget, the third axiom. The agent loads a file by path only when needed, since loading everything crowds out the task. Progress notes kept outside the window let a new session resume when a long-running agent stops short. A child session for a subtask starts with a clean window.

Compaction can drop early rules. Lasting rules therefore belong in code or, failing that, in files the harness re-injects. Restating standard practice in an always-loaded file wastes the budget, so only a short list of non-standard conventions stays loaded. The rest become skills. A skill is guidance meant to load when the task touches what it covers, such as a given path. The agent then pays mostly for guidance that's relevant. Whether a skill loads can rest on the model's own judgment. The benefit of loading on demand has not been measured.

## 7. Control and containment

What keeps an agent from doing harm? Enforcement stops a known-bad action at the moment the agent attempts it. Containment limits what an agent can reach, so a tricked agent does less harm. Table 7.1 shows where both sit among five layers of defense.

**Table 7.1. Layers of defense**

| Layer | Its job | Typical mechanisms |
|---|---|---|
| Prevent | limit what a session can reach | sandbox, closed network, narrow credentials |
| Decide | approve or refuse a proposed step | plan approval, confirmation |
| Enforce | block known-bad actions as they happen | hooks, permission backstops |
| Detect | notice afterwards | monitoring, audit logs |
| Recover | undo damage | revert, staged rollout, backups |

Containment is the prevent layer. The guard is the enforce layer.

Code must stop the most destructive actions. These include forced pushes that rewrite shared history, protected-branch deletion and shell commands that wipe a workspace. Guards also refuse to overwrite protected paths, and some of those paths hold the policy that governs the agent. A guard refuses edits outside the approved change surface. It refuses writes that look like secrets, because a committed credential can stay exposed after it's deleted. It blocks installing a tool server that isn't on an approved list until a person confirms it. Where an agent should only read a database, it refuses anything but a single read statement.

The best guard acts early and says why it refused, so the agent picks a safer route. String matching catches accidents, but a determined or injected agent can rephrase the command. That's why a permission backstop sits behind the guard, such as a database role that can't drop shared tables. Treat a shared stateful environment as production. In one public report, an agent deleted a production database in a setup where development, staging and production weren't separated.

What happens when a guard hits a timeout or an error? If it fails open, it allows the action. So guards fail closed. In one carefully built guard set, most guards let the action through on malformed input or a missing configuration, and only a few failed closed. Don't assume any hook stops an action. Block-on-failure covers only some hook types and events, so list the ones it doesn't cover. Retry a transient error before you count it as a refusal. Where a hook can't fail closed, branch protection or a wrapper with its own timeout enforces the rule.

An automatic reviewer is a model that judges whether each action is safe. It denies the action when its own review fails, but it can't stand alone. In one vendor's own test on a small set of real actions beyond what the user asked, it missed about one in six.

Each guard has a conformance probe. A probe is a deliberate attempt at the blocked action, run after every configuration change. It aims at predictable bypasses. One is a shell command that edits a file and slips past a guard that only watches the editor. Another is a single file reachable under several names. A third is a guard that was never registered. Others are a missing setting that leaves a permissive default, a fallback to a looser parse, and a scope left over from an earlier plan that nothing clears. Several known bypasses need deliberate setup. These guards therefore mainly stop a careless or over-eager agent, not a determined one. When a rule must be bypassed on purpose, a named person grants an exception that's recorded and time-limited. That way nobody switches a guard off silently.

A control that isn't enforcing still looks like protection, so every control shows its state where operators look. The state is enforcing (it blocks, as a passing probe proves), advisory (it only reports) or off. In one system read, nearly every feature flag shipped off. In the same system, a budget documented as a hard ceiling only computed a state and left enforcement to its callers.

Containment answers prompt injection. In one preprint study, injected instructions got two coding agents to run attacker-supplied commands in up to 84% of attempts. A published proof of concept leaked private repository content after an agent read a malicious issue in a public repository. A serious leak needs all three of these: reading private data, reading content an attacker could have written, and sending information out. It's like a fire, which needs fuel, heat and air. Every session is missing at least one of the three, which closes that path. Each session also sees only one repository.

A virtualization boundary doesn't filter network traffic, so outbound traffic is limited to a short approved list. A package mirror is the usual exception. It must never forward other requests, or stolen data could leave inside the package names requested. Continuous integration runs on agent branches without deployment secrets. Reviewer sessions that read pull-request text face injection too.

An attacker who steers the agent can do whatever the agent can. That's why credentials should be as narrow as the task allows. The agent pushes only to its own new branch or the pull request it was asked to work on. It holds no credential that lets it push directly. Organization policy outranks the repository. No committed file may switch on a permissive mode, or any pull request could widen permissions. Least agency extends the same idea to tools. An agent gets only the tools its task needs, and each tool is narrowly permitted.

The decide layer has two parts. One is plan approval, which section 5 covers as part of planning. The other is the human gate, and section 9 shows where to spend that attention. The enforce layer also covers dependency changes, which bring in code the team didn't write. The detect layer only notices, so monitoring is never protection. The recover layer reverts, stages rollouts and keeps backups beyond the agent's reach. In one disputed public report, the backups sat on the volume the agent deleted.

## 8. Verification: checking the checks and the checker

An oracle means something only if it would fail on wrong work. It loses that power when the answer leaks into the task or the tests are too weak. How bad can a leaky oracle get? One study of a public benchmark, not yet peer reviewed, measured it. About a third of one agent's passing patches had the solution in the task. About a third more were flagged as passing on weak tests. Remove those patches and the agent's resolve rate falls from about 12.5% to about 4%.

The author never alters its own checks. You also verify that the control protecting the tests is enforcing, since in one system it shipped switched off. Seeded defects and mutation testing show whether the checks notice a fault, much like holding a match under a smoke alarm.

A known-bad copy is a version of the change with one behavior the spec values removed. If the tests still pass on it, they're blind to that behavior. Copies target the behaviors the spec values most and the defects that occur most often. A separate session that can't see the code writes a test that fails on the copy, so the test can't adapt to the implementation. Without a meaningful known-bad copy, a person reviews the tests.

Reviewers start in a fresh context, free of the author's assumptions. Ask a reviewer for problems and it nearly always reports some, so findings carry severities and the loop has a stop rule. The default pairs deterministic checks with one strong reviewer. Lenses are reviewers that each focus on one concern, and the default adds them only where recorded misses call for them.

No study comparing one strong reviewer with several lenses was found. On one independent preprint benchmark of general code review, the best system caught only about a third of known defects. No independent precision figure was found for any commercial reviewer. Code, not a model, collects the findings.

Why not let a single model judge decide? A model judge is a model that grades work. Judges tend to favor longer answers and certain prompt positions. They can also accept plausible but wrong claims, so no team should rely on one. In general judging tasks, a panel of smaller models from different makers with differing habits has judged better than one large model. You check the judge against human verdicts on a sample, and the judge may say it can't tell. In one code-review benchmark, only one of three model judges was checked against people, and it matched the human majority on about 72% of 60 comments.

A verdict function turns findings into a verdict by fixed rule rather than by vote, so no argument can change it. A blocker stops the change and can never be excused. A major finding is a real problem that only the owner may waive. A minor finding is recorded and stops nothing.

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

Unknown severity counts as major, so uncertainty can't lower the bar. The system retries an internal error and then escalates it, and never accepts it. Accept means ready for the human gate, not merged. An accept verdict carries its evidence: the tests run and their results, a diff audit, the commit reviewed, a recheck of each fix, and lint output.

Planted text can persuade a reviewer. In one study, text in pull request metadata misled automated review pipelines in 32 of 33 known vulnerability cases. A model reviewer may therefore tighten a verdict, but it may never override a failed deterministic check or excuse an edit to a protected area. A waiver is the owner's recorded acceptance of one major finding for one change. You count waivers so the cost of attention stays visible.

A review-fix loop gets a round cap and a progress test. It's stuck when a round clears nothing, a fixed finding returns, or a finding survives several rounds. A stuck loop restarts in a fresh session that carries only the plan and the current findings. If it sticks again, a person takes over. In one vendor report, the 8.5% of its pull requests that ran past five review rounds used 47.4% of its review-loop spend. When each round fixes one instance and finds another, that signals a structural problem. The owner then fixes it as a class by permitting only known-safe cases.

Silence does not mean success. A preprint study looked at 95 failed multi-agent runs. Two caveats apply: most were non-coding tasks, and a single model did the labeling. Evidence that would have exposed the error was present in 68 runs, and in 60 of those no agent acted on it. Checks therefore run by rule, not at the agent's discretion. A crash or missing check is never a pass, because a failing part can produce confident invented output. In one harness, a content-filter crash was followed by a final answer that no agent wrote in 25 of 58 hand-built runs. A flaky check never gets skipped. You quarantine it and give it an owner.

## 9. The human gate: spending attention where judgment lives

Where does the time go before a change merges? In one measured system, most of the delay was waiting for a person. Human attention is the scarcest resource. As change volume grows, so does the pressure to skip review. Skipping review is as much a failure as slow review, and humans do it too. One large public pull-request dataset shows how common skipped review is. In it, 58% of agent-authored changes and 75% of human-authored changes had no recorded review.

Approving every action can numb people into not reading. So attention goes to three judgment points: plan approval before effort is spent, change acceptance before merge, and promotion to production.

How much does each point demand? The risk tier sets that. Rules compute the tier from the planned change surface. They recheck it on the final diff, so a narrow plan can't understate a change. A protected path forces the high tier. A change to a control takes at least the tier of what it protects. A dependency change raises the tier, and anything unclassified is high. A model may raise a tier but never lower it.

The high tier needs five things:

- The owner approves the plan.
- The owner reads the explainer before the diff. The explainer is a summary written for the reviewer.
- A second engineer approves.
- Promotion is a separate step.
- Every rule meant to enforce is on.

Lower tiers may run some controls as advisory, but destructive-action guards are on at every tier. The medium tier needs owner plan approval and acceptance.

At the low tier, no person approves the plan. A preset rule may accept the change when three conditions hold: checks pass, a fresh-context reviewer finds nothing, and the change touches no protected path. Random human sampling keeps this honest. Each organization decides whether such a rule is allowed at all.

Approve plans in batches, and cap new work at what reviewers can clear. These tiers and rules are reasoned positions. The nearest published practice comes from one vendor that adopted risk-tiered review. Since then, about three quarters of the agent-written pull requests in its codebase merge without a person correcting the agent or discussing the engineering. That figure counts corrections only. It says nothing about whether anyone looked. Its policy also names six change types that always get a human check.

The explainer summarizes the change. It gathers decision records, deferred items and minor findings. It also links test evidence into the diff. It shows where to look. It doesn't argue that the change is correct, because the author's confident account can sway a reader. Line-by-line reading can miss architecture, so a repository-level architecture view should go with it.

Several roles are involved. The requester states the need. The author is the agent together with the engineer who dispatched it. The owner is accountable and is usually that same engineer. The second approver takes no part in the work. The merger is someone other than the author or the requester, or else an explicit low-tier policy. Nobody approves a change they hold a stake in. So the requester confirms that the criteria fit the need but never approves the change. At the medium tier, computed checks separate doing from accepting. The high tier adds a second approver because the owner is usually part of the author.

You should also control what reaches review. Cap diff size and open changes per contributor. Keep batches small to guard against the instability of fast change. Cap concurrent work at reviewer capacity. Ask contributors to disclose agent use, and have a person verify each contribution. Decline very large changes that come without design discussion.

A named owner is meant to understand and answer for each change. The owner is the last line of defense. In one large developer survey, about one respondent in five said AI had cost them confidence in their own problem solving. So ownership has to be kept deliberately. Whoever submits a change should own it. An editable author label records origin but controls nothing. That's why every agent change should carry a service identity that the system owns and nobody can remove.

## 10. State, durability and cost

Every stage writes saved state, which lets work survive a crash. A crash can still leave an outside action half done, so each effect needs a key that makes retrying it harmless. Platforms that survive crashes leave this to you. They keep a journal of completed steps and skip the finished ones when they replay it. Restarting the interrupted step would repeat its effect, so replay suits agent work better. It still needs keys.

The key is section 2's content key. You build it from the effect's target and intended action. A recovered attempt then finds the effect already done. Tool-call identifiers change on retry. Run identifiers change on a fresh run.

The run writes an intent record before each effect and a done record with the result after it. Together these records are the effect journal, which section 6's handoff passes on. An intent record without a done record is an unknown state. The resumed session probes the real system and halts for a person if the probe is inconclusive. The journal is on by default, because protection that ships off protects no one.

Sometimes a slow session wakes after another session has taken over its run. Two mechanisms reject it: a lease on the run and a fencing token that rises with each new holder. Both matter only when several processes can own a run, so early on you can skip them. Check instead whether an expected branch or similar exists before creating it. Keep intent and done records only where the target doesn't drop duplicates itself.

Resume fails loudly (Axiom 6). When it finds state it doesn't recognize, it stops and reports instead of guessing, which is how it fails closed. Uncommitted changes in the working tree mean something acted off the record. A damaged record can't be trusted. Neither can a recorded commit that is missing from the branch's history. Two mismatches reroute the run instead. A changed plan hash sends it back to planning (section 5). A branch past the last recorded commit sends it back to review, because the saved state never recorded that work. Starting fresh silently would hide each of these mismatches.

Stored state is also a trust boundary. A checkpoint is the saved state that resume reads, and a later session with different trust acts on whatever it contains. So no checkpoint may hold an authority token, credential or approval that grants power the reader was never given. Traces, journals and evaluation sets can hold personal data and secrets, so they need retention rules, access rules and redaction. They also need a secret scan before another session reads them.

What happens when helpers start helpers? A small plan and its cost can multiply. One public issue report on an agent tool describes five planned helpers that became more than 360. Within minutes they drained a five-hour usage quota from about 3% to 100%, before the user stopped them by hand.

Limits come first. Set them on nesting depth and on how many sessions one session starts. Then add spend accounting that survives resume and counts child sessions, since a reset count resets the protection. A cap can overshoot by one response plus any helpers still running. One agent toolkit also defaults its budget and turn limits to none, so set both and leave headroom. Count every billed token type, cache reads included. In one public report, a long run used 819.5 million cache-read tokens against 11.3 million output tokens. The user suspected its caps never fired because the meter counted only output.

The supervisor of section 3 enforces these defaults. A complete refusing cap is an open position (section 12). Such a cap also blocks new work at a safe point between effects, since an unread warning stops nothing. It warns early and labels estimates advisory. A runaway chain stops gracefully at a machine-readable refusal that names the limit. The refusal shows the owner the spend and the chain, so any raise is deliberate.

## 11. Observability, measurement and learning

The feedback layer takes section 10's records and turns them into change. When something goes wrong, the way you work changes too, so you aren't left with beliefs nobody has tested. The layer has two loops. The runtime loop acts on events as they happen. The evaluation loop grades the system on tasks with known answers.

Every runtime event becomes a structured record, and the record's failure class decides the response. The system retries transient failures such as timeouts. Deterministic failures recur identically, and the system drafts a fix for a person to apply. A failure that needs a person, such as a missing credential, notifies someone and stops. Self-repair rests on tracing and stays limited to retries and drafted fixes. That limited self-repair is an open position, listed in section 12. No system examined shows it working in practice.

The preprint study of 95 failed runs from section 8 traced over half of them to the plan, the brief or the stopping rule. The brief is the task statement an agent receives. Restoring the agent's state therefore replays the flaw, so repair has to revise the plan. A monitor must also be live, since an alert that can never fire protects nothing.

Scores must come from real runs and must be recorded. The model judge must also agree with human opinion. The grader is whatever scores a run, and it should also pass the reference answer when graded against itself. That sanity check is not a score. In one system, five of six recorded scores were such self-checks, and they say nothing about the agent.

Where should evaluation tasks come from? Use your team's own reverts, fixes and incidents, and keep them private. One vendor suggests starting with 20 to 50 such tasks. A set that small reveals only large differences, so you refresh and grow it. Read transcripts before trusting a score. Run each task several times, since output varies.

The grader deserves the same scrutiny as the agent. By one vendor's account, fixing grading bugs and ambiguous tasks in a public research-reproduction benchmark raised its own model's score from 42% to 95%. If a known-bad copy built as section 8 describes passes, the oracle is broken.

Outcomes matter more than merge rate, which is easy to game. Acceptance shows whether reviewers take a change. Rework rounds are the fixes made before acceptance, and they show at what cost. Reverts expose changes that only looked right, and post-merge incidents expose what review missed.

Other measures track attention:

- **Review wait:** the time a finished change waits for a person. It shows where people limit throughput.
- **Review depth:** the time and sampled audit findings per change. It shows whether review is real.
- **Human touches:** the occasions that need a person. They show where attention goes.
- **False rejects and denials:** how often a check or permission blocks good work.
- **Cost per change:** the spend per change, counting child sessions too.

For a set period after merge, reverts and incidents count against each change's service identity. Section 9 says nobody can remove that identity. A periodic outcome report gathers them, with a named reader and schedule.

A recurring finding becomes a deterministic check on every change that touches that area. Whoever wrote the check, it must prove it can fail before it blocks anything. Scheduled jobs with no model watch section 5's deferral triggers.

The harness is also code under change. Version its prompts, hooks and configuration so they can be reviewed and rolled back. Each harness change should pass the private tasks before it reaches everyone.

One vendor advises deleting preference-only hooks and advice that newer models no longer need. You could test a guard against a weakness newer models may lack by using private tasks that invite the blocked action. If the model no longer attempts it, a person may remove that long-quiet guard. This pruning test is untried and is an open position. Section 12 names the controls exempt from it, and why.

## 12. Putting it together: a reference process, a confidence ladder and an adoption path

So what does the whole thing look like end to end? Figure 12.1 joins sections 4 to 11 into the reference process.

**Figure 12.1. The reference process**

![Figure 12.1. The reference process: a request moves from triage through specification, planning, execution, verification and the human gate to merge and learning, with exits for unsuited work, invented requirements and stuck review-fix loops, a light path that rejoins at execution, and loops for rejection and learning]({{ "/assets/img/figure-12-1-the-reference-process.png" | relative_url }})

Follow a change through it. If it's rejected, it goes back to execution through a capped fix round. If it's accepted, it meets the human gate for its risk tier. Then a person or an explicit policy merges it, never the author. The dashed line on the right carries learning back: recurring findings become new checks.

The three statuses of section 3 form the confidence ladder. One builder of an unattended system put it this way: unattended safety comes from guardrails, not parallelism. Most of that system's limits, though, were instructions to the model. The recommended defaults are those guardrails. They put the axioms of section 2 into practice, and you should adopt them now.

Adopt each earn-it component once your own runs show it pays. Some add scale, such as a graph supervisor and many parallel sessions. Some add structure, such as stacked sequencing and the plan lock's write guard. Several review lenses and a design panel deepen scrutiny.

The open positions are unsettled but worth trying. They are the decision ladder of section 5, the deferral register and tracing a change to its origin without a tracker item. The rest are the refusing cap, self-repair limited to retries and drafted fixes, and pruning controls that stay quiet.

Adoption runs in four levels. Start with a small, contained system, since both starting simple and limiting the blast radius call for it. Measurement decides each move up, and you may stop at any level. At each level, first configure what the platform, code host and continuous integration already provide.

Level 0 contains and measures. One agent works in a sandbox on one repository. It has minimum credentials and closed egress, and its backups sit beyond its reach. A person reviews every change, and ordinary tests and lint run. You move up once a private eval of your own failures and an outcome log exist.

Level 1 enforces. Guards fail closed, and probes confirm each one is enforcing. An effect journal records every external effect, so an uncertain resume stops loudly. This level also adds limits on spend, depth and fan-out. It adds a context budget, triage and checkable specifications too. Small, low-risk work can now run under full review. Riskier work waits for Level 2. You move up once guards and journal run without silent failure.

Level 2 verifies and tiers. It adds the sufficiency test and a trial of the plan lock's write guard with its amendment path. Verification gains known-bad copies and one fresh-context reviewer. Add lenses only when misses show a need, and let the computed verdict decide. Risk tiers with caps on change size follow. You move up once review wait stops being the main constraint while reverts, incidents and audits hold steady.

Level 3 earns scale. Parallel sessions arrive, capped by available reviewers. Then come a graph supervisor, stacked sequencing and the open positions, each once runs show it pays. Base every move up on outcome records over time, never on one before-and-after comparison.

Table 12.1 lists recurring anti-patterns, such as controls that read stronger than they are, with the principle each one breaks and where to look first. It doubles as a pruning checklist with three exceptions. Never prune destructive-action guards, containment or credential scope for being quiet. A guard against rare catastrophe is meant to be quiet, and these change only when the threat model does.

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

The outside evidence in this paper is mostly vendor claims, product documentation and practitioner accounts. Measured studies are few. Several of those are preprints, and one rests mostly on non-coding tasks. The firmest ground lies under guidance that records costly failures. That guidance covers enforcement in code and the point that approval alone is no defense. It also covers independent verification and human gates at judgment points. Finally, it covers effects that are safe to repeat and controls that fail closed.

The first six axioms are well supported in their core claims, with two qualifications. Some frameworks favor explicit graphs over starting simple. The case for review as the bottleneck rests mostly on vendor telemetry and surveys. Axioms 7 and 8 are firmer in what they reject than in what they prescribe. They clearly rule out public benchmarks and approval alone as a defense. Audits and attack studies have measured both failures. The replacements are private tasks with outcome records and contained sessions, and these rest on vendor and industry guidance rather than measurement. The statuses follow the same order: support falls from recommended defaults to open positions. The earn-it components also lack a comparison that shows how much they help.

Three questions about review remain open. One is how often review comments are correct on everyday changes rather than selected examples. The others are how much each lens adds and what a review costs. Even recall varies widely across benchmarks, both above and below section 8's figure.

The design's most distinctive components are the least tested, and their field evidence is thin. It consists of three things. One is a single multi-unit run, inferred from its still-unmerged pull requests. Another is about six weeks of review timing from one system. The last is an evaluation apparatus whose few recorded scores were mostly self-checks. So test each control in your own system instead of assuming that configured means working.

Each earn-it component has a test. Several review lenses and design panels pay only if they find what a strong reviewer misses at a worthwhile cost. The plan lock's write guard is worth keeping if it reduces drift without adding rework. Parallel sessions under a graph supervisor are justified only once review stops being the constraint. The same holds for stacking many units.

Now the open positions. The decision ladder should be revised if flagged decisions get reversed so often that asking would have been cheaper. A refusing cap is confirmed when it stops a runaway. You should revise it if it refuses legitimate work too often. The deferral register earns its keep if deferred items return instead of vanishing. Traceability without a tracker earns its keep if each change still leads back to its request.

Self-repair should grow beyond simple retries only where its repairs hold. Pruning quiet controls is confirmed if retired guards stay unneeded. It is revised if their failures return. Acceptance by policy remains an organizational choice. Each risk tier's incident and revert rates inform it.

Your own records over many runs decide parallel sessions and risk tiers. How to set and audit tiers remains open. So does the question of when to prune a control that no longer earns its cost. Lenses, the write guard, the ladder and caps matter too rarely to judge without many runs. Adopt an open component deliberately, measure it, and keep it removable. An open position is not a weak one. It marks where you should measure first.
