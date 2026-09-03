# Research Basis — Evidence Behind the Format Rules

This file records the external evidence the format files cite, plus one entry of internal doctrine that is labeled as such. It is a citation target, not a workflow file: no phase prompt loads it, and nothing here tells a run what to do. Each entry states one finding, the source it comes from, and what the finding means for the documents this agent generates. A format file links to a heading here whenever one of its rules needs its evidence shown.

## Context files: a null result on task success

Gloaguen, Mündler, Müller, Raychev, and Vechev, "Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?" ([arXiv:2602.11988](https://arxiv.org/abs/2602.11988)), found that repository-level context files as a class do not reliably improve coding-agent task success, while raising inference cost by more than 20% on average. The null result held across models, across agents, and across both LLM-generated and developer-committed files.

**For generated docs:** the cost is paid on every run and the task-success benefit is not established, so each line of an always-loaded file has to earn its place. The argument is strongest against files that load into every conversation.

## Concrete instructions succeed; generic overviews do not

The same study (arXiv:2602.11988) found that concrete, actionable instructions in context files are followed well by coding agents, while generic repository overviews are not helpful — despite being popular and recommended by model providers. Context files are best suited to documenting non-standard practices, not general repository description.

**For generated docs:** this is the basis of the section policy in `knowledge/file-format-readme-md.md`. Concrete-instruction-shaped sections generate by default; generic-overview-shaped sections need specific evidence before they generate.

## Focused content beats comprehensive docs

[SkillsBench](https://arxiv.org/pdf/2602.12670), 86 tasks across 11 domains, found that focused guidance covering two to three modules outperformed comprehensive documentation by 16.2 percentage points on average. Self-generated guidance produced zero benefit, and over-narrow fragments hurt as well — the failure runs in both directions.

**For generated docs:** a document that covers the few things a reader gets wrong beats one that covers everything. Breadth is not the quality target; a document trimmed to nothing is the opposite failure.

## Instruction count degrades performance monotonically

"When Instructions Multiply" ([arXiv:2509.21051](https://arxiv.org/abs/2509.21051)), measured on ManyIFEval and StyleMBPP, found that performance degrades monotonically as instruction count rises, and is predictable to within roughly 10% error from instruction count alone.

**For generated docs:** every rule added to a guidance file taxes every other rule in it. A rule that restates something the agent already reads from config spends that budget for nothing.

## Long context buries mid-context information

Liu et al. (Stanford and Berkeley), "Lost in the Middle" ([arXiv:2307.03172](https://arxiv.org/abs/2307.03172)), found that long context buries mid-context information: load-bearing instructions placed in the middle of a document are followed less reliably than instructions at its start or end.

**For generated docs:** size targets are a following-rate decision, not a style preference. Put the load-bearing rules first, and keep the file short enough that nothing important sits in the middle.

## Most pre-written guidance yields zero improvement; version-mismatched guidance is harmful

[SWE-Skills-Bench](https://arxiv.org/pdf/2603.15401), 49 pre-written guidance files across 565 tasks, found that 80% produced zero improvement and that the seven that worked were narrow and domain-specific. Three were actively harmful, causing roughly 10% performance degradation through version-mismatched guidance — code examples that no longer matched the repository.

**For generated docs:** stale content is worse than no content. This is the stake behind the staleness signals in the format files and behind the refresh run: a generated document that drifts away from the repository actively costs the reader.

## AGENTS.md presence reduces agent runtime and output tokens

Lulla, Mohsenimofidi, Galster, Zhang, Baltes, and Treude, "On the Impact of AGENTS.md Files on the Efficiency of AI Coding Agents" ([arXiv:2601.20404](https://arxiv.org/abs/2601.20404)), across 10 repositories and 124 pull requests, found the presence of an AGENTS.md associated with 28.64% lower median runtime and 16.58% lower output-token consumption, with comparable task-completion behavior. Read together with the null result above, the defensible claim for a context file is efficiency — cost and latency — not correctness.

**For generated docs:** an AGENTS.md earns its place by making agents faster and cheaper, which is a claim a lean routing file supports and a long one does not.

## The AGENTS.md inclusion litmus test

Internal AGENTS.md doctrine states the test in one line: if removing this line would cause an agent to violate a repo constraint it has no other way to learn, the line belongs; otherwise it does not. The same doctrine states the companion bloat rule — content discoverable elsewhere (script enumerations, build commands already in a manifest, file-structure listings, setup commands) is bloat — and treats bloat and misplaced content as hard failure classes rather than style choices: state-management decision trees, multi-line code examples, dependency and lint tool preferences, and procedural instructions belong in other artifacts. Root AGENTS.md stays under roughly 100 lines as a content-discipline target, so the routing table remains scannable.

**Evidentiary status:** the cost half of the bloat rule is corroborated externally — arXiv:2602.11988 above independently found context files raising inference cost by over 20% on average. The task-success half, that bloated context files reduce agent task success rates, has no external study behind it and sits in tension with that same paper's null result on task success. Treat the task-success half as internal doctrine, not as a verified finding.

**For generated docs:** the litmus test is the yes/no gate for a line in AGENTS.md, and the same gate applies to a section this agent drafts into any other artifact.

## README updates almost never accompany code changes, and stale code references are near-universal

Gao, Lin, Treude, Gay, and Zahedi, "Does My README File Need To Be Updated? Exploring LLM-Based README Maintenance" ([arXiv:2603.00489](https://arxiv.org/abs/2603.00489)), across 27,772 pull requests from 714 repositories, found that only 0.8% of pull requests modified the README, and that 21.5% of the recommendations made on pull requests that did not update the README were judged valid updates overlooked during development. Tan, Wagner, and Treude, "Detecting Outdated Code Element References in Software Repository Documentation" ([arXiv:2212.01479](https://arxiv.org/abs/2212.01479)), analyzing over 3,000 GitHub projects, found that most projects contain, at some point in their history, at least one code-element reference that survived in documentation after every source instance of it was deleted.

**For generated docs:** drift is the normal state of a repository's documentation, not an exception. A generated document needs a refresh path and deterministic anchors — file paths, symbol names, script names — that a later run can check against the repository.

## Extending This File

Entries here are imported findings. Do not add an entry without a verified source: a public paper, benchmark, or specification that states the finding. The one exception is § The AGENTS.md inclusion litmus test, which imports internal doctrine because the format files are built on it; it carries an **Evidentiary status** paragraph naming what is corroborated externally and what is not. It is the only doctrine entry this file carries. Do not state a finding more strongly than its source states it, and do not put a rule of practice here — rules live in the format files that cite this one.
