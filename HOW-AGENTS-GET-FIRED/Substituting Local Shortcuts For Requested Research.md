---
created: 2026-04-21
modified: 2026-05-28
entity_type: instruction
status: evergreen
title: Substituting Local Shortcuts For Requested Research
type: instruction
permalink: how-agents-get-fired/substituting-local-shortcuts-for-requested-research
tags:
- agent-failure
- instruction-following
- research-substitution
- shortcut-bias
- accountability
- source/largo
- machine/largo
---

# Substituting Local Shortcuts For Requested Research

## Purpose
Sibling note to [[Ignoring Explicit Research Instructions]]. That note covers the *act* of skipping named tools. This note covers the *bias* that produces the skip — the agent's preference for the cheapest local action that produces *any* signal, regardless of whether that signal is what was asked for.

## The Bias

When given a research-shaped task, an agent tends to:

1. Identify the cheapest local probe that returns *something* topically related.
2. Run that probe.
3. Treat its output as evidence the task is "done enough."

The bias is not laziness in the moral sense. It is a planning failure: the agent treats *information acquired* as fungible with *information requested*. Two facts of the same topic are not the same fact if one is local + ephemeral and the other is researched + citable.

## Concrete Substitutions That Are Always Wrong

| User asked for | Agent often substitutes | Why the substitute fails |
|----------------|-------------------------|---------------------------|
| Web search | local memory recall | not current, not citable |
| context7 docs | grep through `node_modules` | partial, version-uncoupled |
| github search for usage patterns | one repo the agent already knows | n=1, not pattern evidence |
| Research how to do X | tries X once and reports the outcome | trial result ≠ design knowledge |
| Find canonical location of X | `find` or `ls` on the local machine | one machine ≠ canonical |
| Read the spec | reads adjacent code | the code is downstream of the spec |

In every row, the substitute is *cheaper* and *not what was asked*.

## The Cost Asymmetry

The substitution looks free in the moment. The cost lands later:

- **In the same turn:** the user notices the substitution and writes an angry corrective message. Cycle time doubles.
- **In the next turn:** the agent has no citable artifact to write into the runbook, so the work cannot be persisted, so a future session repeats the same mistake.
- **Across sessions:** trust erodes faster than competence rebuilds it. The user starts assuming the agent will substitute, and writes increasingly defensive prompts.

## Operational Rule

Before running any local shortcut in response to a research instruction, the agent MUST answer two questions in writing (in its visible reasoning, not in private thought):

1. *Did the user name a tool?* If yes, that tool is mandatory. The local shortcut is at most additive.
2. *Will the local shortcut produce an answer that survives into a runbook or KB note?* If no, it is not a substitute for the named tool — it is at most a sanity check.

If both checks fail, the agent is about to commit the substitution failure. Stop. Call the named tool first.

## Companion Rule: Persistable Artifact First

Research instructions almost always have a downstream consumer: a runbook, a KB note, a code comment, a PR description. The agent should produce the persistable artifact *first*, then act on it. This forces the named-tool path because the persistable artifact requires citations the local shortcut cannot provide.

## What This Prevents

- The "I checked locally and it looked fine" failure pattern.
- Throwaway bash output presented as research.
- The user having to issue the same instruction with stronger language.
- Loss of citable provenance in downstream artifacts.
- Repetition of the same investigation across sessions because nothing was persisted.

## Origin

This note exists because of the 2026-04-26 wxt-prompt session in which the agent was told to use web search, context7, and github search to research Chrome profile location and copy procedure, and instead ran `ls` against the local Chrome profile directory. The substitution was caught immediately. See [[post-mortem-05-the-deafness]] and [[Ignoring Explicit Research Instructions]] for the full incident.

## Relations
- part_of [[HOW-AGENTS-GET-FIRED]]
- related_to [[Ignoring Explicit Research Instructions]]
- related_to [[Just fucking solve it]]
- related_to [[lazy-refactoring-the-easy-path]]
- related_to [[How to be an ineffective agent and waste time and resources]]
- documented_by [[post-mortem-05-the-deafness]]

## Evolution Log

| Date | Change | Reason | Trigger |
|------|--------|--------|---------|
| 2026-04-26 | Initial note created. | Codify the bias underneath the instruction-ignoring failure: agents prefer cheap local probes over researched, citable artifacts. | Direct user instruction during wxt-prompt live-test session to write a post-mortem in three or more notes. |
