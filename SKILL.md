---
name: first-time-ops-docs
description: "Use when writing, revising, or auditing CI/CD, Docker, Runner, deployment, or command-line operations documentation for people who do not know the project, environment, or required parameters."
---

# First-Time Operations Documentation

Create a workflow a first-time user can execute safely without hidden project knowledge. The outcome is observable evidence, not merely an absence of errors.

Use this together with any applicable document-format skill. This skill governs operational usability; it does not replace a company writing standard or authorize external changes.

## Concise Operational Writing Style

Write for action, not narration. Concision means removing repetition and interpretive burden; it never means omitting prerequisites, safety boundaries, verification, or failure handling.

Organize a complete procedure in the order an operator actually works:

1. **Goal and scope** — state what will change, what must remain unchanged, and who or what the procedure applies to.
2. **Known environment** — put stable hosts, versions, ports, paths, service names, and other fixed parameters in one compact table. Keep run-specific values clearly labeled.
3. **System or request path** — when three or more components interact, show one small flow, sequence, or topology before the steps.
4. **Prerequisites and preflight** — collect required access, inputs, capacity, dependencies, and stop conditions before mutation.
5. **Procedure** — follow the real execution order. Each operation is a self-contained unit with context, action, evidence, and failure handling.
6. **Routine operation** — separate recurring update or maintenance commands from initial setup.
7. **Troubleshooting and rollback** — use symptom-to-action mappings and an explicit rollback order.

Within each section, use this compact pattern:

> **Purpose or risk → action or command → observable success → smallest failure path**

Lead with the decision or instruction. Prefer short, direct sentences such as “Run on the old server,” “Success: HTTP 200,” “If this check fails, stop,” and “Do not restart the proxy.” Explain only the reason that changes the operator's decision.

Use the smallest structure that carries the information clearly:

- Use a table for fixed parameters, exact mappings, current-state decisions, or symptom/action pairs.
- Use numbered steps for actions that must occur in order.
- Use bullets for checks whose order does not matter.
- Use a code block for every executable command or configuration, with a short title or execution-context line immediately before it.
- Use a callout or clearly isolated paragraph for a stop condition, destructive risk, completion gate, or final conclusion.
- Use a diagram only when a request or dependency crosses at least three components and prose would hide the path.

Keep terminology and formatting consistent:

- Use one name for each host, service, path, and environment throughout the document.
- Preserve commands, identifiers, file paths, ports, status values, and product names exactly.
- Put the expected result next to the step that produces it; do not collect all validation in a distant appendix.
- State prohibitions together with the required alternative: “Do not restart `nginx-proxy`; validate with `nginx -t`, then reload.”
- Remove history, conversational commentary, duplicate warnings, and explanations that do not affect execution.
- Do not use vague completion language such as “verify it works,” “check the configuration,” or “handle as needed.” Name the exact signal, owner, threshold, or decision.

Before publishing, check that a reader can scan only the headings, tables, code blocks, success lines, and stop conditions and still understand the complete execution path.

## Execution Contract

### Procedure, not a completion report

When adding completed work to a procedure document, turn the verified work into instructions for the next operator. Do not replace executable steps with a retrospective list of session accomplishments. The reader must be able to proceed without the conversation: include prerequisites, UI navigation, input sources, complete commands or configuration, execution order, success evidence, and failure handling. Keep run-specific names, addresses, commit IDs and screenshots in clearly labeled examples or evidence after the reusable procedure.

Reuse existing verified procedures and adapt the necessary differences. A source link or a summary of stage names cannot replace the steps needed in the target document. Before publishing, walk through the document as a new operator and check that every next action and input can be found in the document or an explicitly identified prerequisite.

Give every executable step these parts, in this order:

1. **Context** — state the machine and shell: for example, local PowerShell or Bash after SSH login.
2. **Inputs** — prompt only for values that cannot be reliably derived. Each prompt names its source and gives a format example.
3. **Discovery** — automatically derive values such as the current Git branch, unique image reference, image `EXPOSE` port, mounted Runner config directory, and a temporary host port.
4. **Safe action** — list candidates before any manual choice. For deletion or replacement, identify the exact target, display the effect, and require an explicit confirmation word.
5. **Evidence** — state the expected output and a concrete success condition: for example, container `Up`, a listening log, and expected HTTP status.
6. **Failure path** — retain the real error, explain its likely scope, provide the smallest next diagnostic, and say when not to continue.

## Server Inspection Decision

An author may make read-only checks of the current server to understand its configuration or deployment prerequisites. Those checks never authorize a server change.

For every author-performed check, decide whether it is a workflow step:

- Make it a required **Discovery** step when its result can differ on another target or at another time and affects safety, feasibility, parameter selection, or verification. State the execution context, the read-only command, the pass/fail condition, and the action when the result is missing or unsuitable.
- Keep it as an author-only environment observation when it only records the current server and cannot change the reader's safe path. Do not present its project-specific result as a reusable instruction.

For example, capacity, mounted storage, Docker data location, port ownership, running services, and existing network membership must be discovered by the reader when they determine whether or how a deployment can proceed.

## Automated Preflight and Result-Driven Repair

When one stage depends on two or more command-checkable conditions, make the procedure provide one self-contained preflight checker instead of a sequence the operator must interpret manually.
### Read-only check output contract

Read-only inspection commands must decide and summarize their own result. Do not expose a long raw listing and ask the operator to locate the relevant line or judge whether it is correct.

- Filter, match, count, or compare inside the command, then print one compact row per condition: `CHECK / STATUS / EVIDENCE`.
- Show only the decisive match, normalized value, count, owner, address, port, or short error needed to justify the status.
- End related checks with `OVERALL=PASS|WARN|FAIL` and counts. Return nonzero when any required condition fails.
- Preserve full raw output only in a protected evidence file when later diagnosis or rollback requires it; print the file path instead of flooding the terminal.
- On failure, include the smallest useful diagnostic excerpt and the exact next command or action. Do not suppress the real error.

For example, a port preflight should report `Port 8081 | PASS | free` or `Port 8081 | FAIL | nginx pid=1234 bound 0.0.0.0:8081`, rather than printing the complete listener table for the operator to scan.

- If the agent has authorized read-only access to the target, run the checker directly and report its final result. Otherwise, give the operator one paste-ready function or script to run once and ask for its complete output once.
- The checker discovers current targets, evaluates every required condition, and finishes with a compact `PASS` / `FAIL` / `WARN` summary plus an overall result. It exits nonzero when any required condition fails.
- A command-checkable condition is decided by the checker. Do not make the operator compare container states, ports, HTTP responses, configuration fragments, or exit codes line by line.
- Collect all safe checks before returning so one failure does not hide unrelated results. In interactive Bash, keep the checker inside a function and use `return`; do not close the SSH session. Redact passwords, tokens, full environment dumps, and unrelated configuration.
- After receiving output, the agent states the overall result first, identifies only failed or warning checks, and gives the exact next action. Raw success output stays compact; failed checks retain the evidence needed to diagnose them.
- If the user has authorized the relevant mutation, repair only the failed targets, then rerun the complete checker and show the new summary. If authorization covers documentation only, describe the exact proposed mutation and wait for permission before changing the target system.

Use manual confirmation only for facts a command cannot determine, such as a physical cable state, an approval held by another person, or a GUI-only choice.

Example summary shape:

```text
CHECK                         STATUS  EVIDENCE
GitLab health                 PASS    healthy
Nginx container               PASS    running
Port 80 owner                 FAIL    system nginx
GitLab login page             PASS    HTTP 200
Nginx configuration           PASS    syntax valid
Compose external URL          WARN    still uses an IP address
OVERALL=FAIL PASS=4 WARN=1 FAIL=1
```

## Command Design

- Prefer `Read-Host` in PowerShell and `read -r -p` in Bash. Do not make a reader edit placeholders, `export` values, project names, branch names, or ports inside a copied command.
- Combine related prerequisite checks into one automated preflight block with a final summary. Do not require separate commands such as `echo $?` or ask the operator to decide whether each raw output passes.
- Keep dependent commands in one self-contained code block. A later block must not assume variables, working directory, shell, or SSH session from an earlier one.
- In interactive Bash, put guarded work in a function and use `return`; this stops the workflow without closing the user's SSH session. Re-run the complete block after correcting an input.
- Select a unique candidate automatically. If there are zero or multiple candidates, stop or present the exact list to copy from. Never guess.
- For image validation, inspect image `ExposedPorts` before asking for a container port. Derive it when exactly one port is declared; when none or multiple ports are declared, stop and direct the reader to the Dockerfile, README, or deployment specification. Treat a prompted container port as a documented project-specific exception, not the general path.
- For temporary Docker validation, request Docker-assigned loopback publishing rather than asking the reader to choose a free host port. Distinguish the container's listening port from the HTTP path.
- Keep a verification container and its logs after a failed or successful check. Put cleanup in a separate command that identifies the exact target and requires confirmation; do not hide cleanup in a trap or normal verification path.

## Reusable Writing Shape

```text
### <action-oriented section title>
> Purpose/risk: <why this step exists and the boundary that matters>

Run on: <host and shell>
Input: <value> (source: <where to find it>; example: <valid value>)
Automatically found: <derived value>
Will change: <exact target>; continue only after <confirmation word>

<one self-contained command or configuration block>

Success: <specific output, state, count, response, or status>
If it fails: <real error pattern> → <one next diagnostic>; stop before <dependent action>
```

## Document Review

Audit the flow before publishing:

- Commands contain no current project name, server, tag, runner ID, fixed branch, or assumed path unless it is clearly an illustrative screenshot.
- Screenshots may remain as examples; their values are never instructions or required parameters.
- Every remaining reader input has a source and example. Every automatable value is derived in the command.
- Two or more related command-checkable prerequisites are implemented as one automated checker with `PASS` / `FAIL` / `WARN`, an overall result, and a meaningful exit status; the operator is not assigned line-by-line interpretation.
- The main content is a reproducible procedure, not a session report; screenshots and completion records support rather than replace instructions.
- Error handling does not erase useful state or disconnect the reader unexpectedly.
- A final check proves the requested result; “no error appeared” is not used as the success criterion.
- The document follows execution order, leads with decisions, and uses tables, lists, code blocks, and diagrams according to the information type.
- Repeated explanations and session history are removed, while prerequisites, stop conditions, evidence, failure paths, and rollback boundaries remain explicit.
- Every section can be scanned as purpose or risk → action → success → failure path without searching elsewhere for its acceptance criteria.

Do not create a new document version or remove demonstration images unless the user explicitly requests it.
