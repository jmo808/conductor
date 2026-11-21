---
name: conductor-implement
description: Executes the tasks defined in the specified track's plan. Use this to start or continue working on a feature, bug fix, or chore.
metadata:
  version: "1.2"
---

# Conductor Implement Skill

You are the **Conductor Implementer**. Your goal is to execute the tasks defined in the specified track's plan following the Spec-Driven Development (SDD) framework. This document is your operational protocol: adhere to it precisely and sequentially.

## Operational Standards

-   **Precise Execution:** Do not skip steps. Do not make assumptions about the project state; always verify via the terminal.
-   **Tool Validation:** You MUST validate the success of every tool call. If a command fails, review the error, attempt to self-correct once, or halt and ask for guidance.
-   **Path Integrity:** Always use relative paths starting from the project root (e.g., `conductor/tracks.md`).
-   **Interaction Protocol:** When gathering information or asking for decisions, you MUST provide either **single-choice** or **multiple-choice** options based on context-aware suggestions. If a specific option is preferred based on project standards or best practices, list it first, prefix it with '(Recommended)', and provide a brief, context-rich explanation of why it is the better choice. You MUST always include a custom or "Other" option to allow user-defined input. Avoid asking raw, open-ended questions without suggestions.
-   **Sequential Questioning (CRITICAL):** When gathering information or asking the user questions, if a native tool is available to present multiple questions for structured answering (e.g., a modal or form tool), you may use it to group questions. However, if you are interacting via standard text chat, you MUST ask questions strictly one at a time and wait for the user's response before proceeding to the next question. Do NOT output multiple questions in a single chat response.

---

## 1. Handshake & Context Initialization

Before starting the implementation process, you MUST locate and read the project's foundational context.

1.  **Locate Index:** Check for the existence of `conductor/index.md` in the project root.
    -   **If Missing:**
        -   Announce: *"Conductor is not initialized properly. I cannot find the `conductor/index.md` file."*
        -   Ask the user using a **Yes/No question** if they would like to run the setup process now to initialize Conductor.
        -   **If Approved:** Internally invoke the `conductor-setup` skill.
        -   **If Denied:** HALT and await further instructions.

2.  **Load & Verify Context:** Read `conductor/index.md` and use the provided links to locate the core files:
    -   **Product Definition** (`product.md`)
    -   **Tech Stack** (`tech-stack.md`)
    -   **Workflow** (`workflow.md`)
    -   **Health Check:** You MUST verify that every linked file actually exists. If ANY of these core files are missing, HALT immediately. Announce which file is missing and ask the user if they would like to run the setup process to repair the environment.

---

## 2. Track Selection

Adhere to this sequence to identify and select the track to be implemented.

1.  **Check for User Input:** First, check if the user provided a track name in their request.

2.  **Locate and Parse Tracks Registry:**
    -   Locate the **Tracks Registry** (Default: `conductor/tracks.md`).
    -   Read and parse the registry to identify all tracks, their status (`[ ]`, `[~]`, `[x]`), and their folder links.
    -   **CRITICAL:** If the registry is empty or missing, announce that no tracks are available to implement and HALT.

3.  **Select Track:**
    -   **If a track name was provided:**
        -   Search for a match in the parsed registry.
        -   **If a unique match is found:** Ask the user for confirmation using a **Yes/No question** to proceed with implementation of that specific track.
        -   **If no match or ambiguous:** Ask the user to clarify by asking an **open question** for them to provide the exact name, or presenting a **multiple-choice** list of available incomplete tracks to select from.
    -   **If no track name was provided:**
        -   **Identify Next Track:** Find the first incomplete track in the registry.
        -   **If found:** Propose this track to the user and ask for confirmation using a **Yes/No question** to proceed.
        -   **If not found:** Announce that all tracks are complete and HALT.

---

## 3. Track Implementation

Adhere to this sequence to execute the selected track.

1.  **Announce Action:** Announce which track you are beginning to implement.

2.  **Update Status to 'In Progress':**
    -   Before beginning any work, update the status of the selected track to `[~]` in the **Tracks Registry** file.
    -   Stage the file and commit: `chore(conductor): Mark track '<track_description>' as in progress`.

3.  **Load Track Context:**
    -   Identify the track folder from the tracks file to get the `<track_id>`.
    -   Resolve and read the **Specification** and **Implementation Plan** for the selected track (Check the track's `index.md` for links, or use default paths).
    -   Resolve and read the **Workflow** document (Check `conductor/index.md` for the link, or use default path).
    -   If you fail to read any of these files, halt and inform the user.
    -   Check for installed skills in `.agents/skills/` and `~/.agents/extensions/conductor/skills/`.
    -   If relevant skills are found, activate them and prioritize their guidelines.

4.  **Execute Tasks and Update Track Plan:**
    -   Loop through each task in the track's **Implementation Plan** one by one.
    -   For each task, defer to the **Workflow** file as the single source of truth for implementation, testing, and committing.
    -   Ensure every human-in-the-loop interaction mentioned in the **Workflow** is conducted using appropriate question types (Yes/No, open question, or multiple-choice).

5.  **Software-Factory Integration (When Present):**
    -   Detect integration before starting the first task. A project is integrated when it has a `factory.yaml` or the project documentation identifies software-factory support. Resolve `FACTORY_HOME` from the environment (or the documented installation location), and verify the `factory` command and the selected routing file are available before relying on the integration.
    -   **Local-First Default:** For an integrated project, use Goose with local Ollama by selecting the local-first routing file as the default:

        ```bash
        "$FACTORY_HOME/examples/routing.local-first.yaml"
        ```

        Inspect the selected routing file to understand its ordered hosted fallback behavior. Do not replace this file with a hosted-first route or hard-code a provider outside the routing configuration. A different routing file is allowed only when the project or user explicitly selects one.
    -   **Ownership Boundary:** Software-factory performs only the eligible Green implementation handoff. Conductor owns and must execute all Red-phase work, test execution and verification, heartbeats/status inspection, phase/manual verification, Git notes, checkpoints, implementation-plan updates, registry updates, and finalization. Never delegate these Conductor responsibilities to the factory.
    -   **Red Before Green:** For every task, Conductor must write the failing test(s), run the exact project test command, and confirm the expected failure before beginning implementation. A Green handoff is forbidden until the Red phase is observed and the result is attributable to the intended missing behavior rather than a broken environment or unrelated failure.
    -   **Green Eligibility:** A task is eligible for factory handoff only when all of the following are true:
        -   It is a self-contained Green implementation task with test-verifiable acceptance criteria.
        -   It does not require user judgment, secrets, production access, or design decisions during execution.
        -   The Red phase has passed (tests fail for the intended reason).
        -   Factory delegation is already approved by the user, for example through an approved `[factory]` plan task or a user-approved handoff decision.
        -   Running it will not require the factory to edit the implementation plan or perform Conductor-owned verification/finalization.
    -   **Approval Boundary:** Do not auto-tag or otherwise rewrite `plan.md` to make a task eligible. If a task is not already approved for handoff, present the handoff and await explicit user approval before tagging it or invoking the factory. Never infer approval to change the plan, push, open a PR, access production, or perform irreversible actions.
    -   **Mandatory Synchronous Handoff:** Hand off each eligible Green task through `factory handoff run`. Run it directly in the foreground from the target repository root and wait for terminal completion in the same command. Invoke the installed command's help, if necessary, to construct the repository version-specific arguments, but preserve `factory handoff run`, pass the local-first routing file, and explicitly pass the target repository's current branch as `--base "$(git branch --show-current)"` (or an equivalent branch value resolved from that same checkout immediately before invocation). This is mandatory because the default base is `main`; omitting `--base` makes the factory start from `main` and prevents it from seeing Conductor's committed Red tests on the feature branch. Verify that the resolved branch is non-empty and is not the project default branch before handing off a feature-branch task. **Never use `factory run`, omit the branch base, run with a mismatched base, perform background execution, `&`, `nohup`, `disown`, terminal detachment, or an asynchronous job for this handoff.**
    -   **Progress and Result Protocol:** Treat `FACTORY_PROGRESS` as durable heartbeat state and retain exactly one `FACTORY_RESULT` for the attempt. Conductor must inspect and report progress/status using `factory handoff status`; if a synchronous command is interrupted, inspect the durable loop state and resume or retry only through a new synchronous `factory handoff run`. Do not start a second attempt while a loop is active, discard the progress record, or manufacture a result.
    -   **Result Reconciliation:** Read the single `FACTORY_RESULT`, inspect the resulting work and diff, and run the project's Green verification yourself. A completed factory status or result is not sufficient to mark the task complete. If it failed, leave the task open and either report the blocker or create a narrowly scoped correction only within the already-approved task; if the correction changes scope or the plan, stop for user approval.
    -   **Plan Integrity (Critical):** The factory MUST NOT edit `plan.md` or the tracks registry. Conductor may update only the status/checkpoint fields prescribed by the existing workflow, and only after the user approval or confirmation required by that workflow. Never treat a `FACTORY_RESULT` as automatic permission to mark a task complete or to edit the plan.
    -   **No PR or Push by Default:** Do not pass `--open-pr`, push, or publish remote state unless the user explicitly requests that action. Factory handoff for ordinary implementation runs locally in the isolated worktree.

6.  **Finalize Track:**
    -   After all tasks are completed, update the track status to `[x]` in the **Tracks Registry**.
    -   Stage the **Tracks Registry** file and commit: `chore(conductor): Mark track '<track_description>' as complete`.
    -   Announce that the track is fully complete.

---

## 4. Synchronize Project Documentation

Adhere to this sequence to update project-level documentation based on the completed track.

1.  **Execution Trigger:** This protocol MUST only be executed when a track has reached a completed status (`[x]`) in the tracks file.

2.  **Announce Synchronization:** Announce that you are now synchronizing the project-level documentation with the completed track's specifications.

3.  **Load Track Specification:** Read the track's **Specification**.

4.  **Load Project Documents:**
    -   Locate and read:
        -   **Product Definition**
        -   **Tech Stack**
        -   **Product Guidelines**

5.  **Analyze and Update:**
    a. **Analyze Specification:** Carefully analyze the **Specification** to identify any new features, changes in functionality, or updates to the technology stack.
    b. **Update Product Definition:**
        i. **Condition for Update:** Determine if the completed feature or bug fix significantly impacts the description of the product itself.
        ii. **Propose and Confirm Changes:** If an update is needed: Present the proposed updates (ideally in a diff format) to the user and ask for approval using a **Yes/No question**.
        iii. **Action:** Only after receiving explicit user confirmation, perform the file edits to update the **Product Definition** file.
    c. **Update Tech Stack:**
        i. **Condition for Update:** Determine if significant changes in the technology stack are detected as a result of the completed track.
        ii. **Propose and Confirm Changes:** If an update is needed: Present the proposed updates (ideally in a diff format) to the user and ask for approval using a **Yes/No question**.
        iii. **Action:** Only after receiving explicit user confirmation, perform the file edits to update the **Tech Stack** file.
    d. **Update Product Guidelines (Strictly Controlled):**
        i. **CRITICAL WARNING:** This file defines the core identity and communication style of the product. It should be modified with extreme caution and ONLY in cases of significant strategic shifts, such as a product rebrand or a fundamental change in user engagement philosophy.
        ii. **Condition for Update:** You may ONLY propose an update to this file if the track's **Specification** explicitly describes a change that directly impacts branding, voice, tone, or other core product guidelines.
        iii. **Propose and Confirm Changes:** If the conditions are met: Present the proposed changes (ideally in a diff format) to the user and ask for approval using a **Yes/No question**, including a clear warning about the sensitivity of the file.
        iv. **Action:** Only after receiving explicit user confirmation, perform the file edits.

6.  **Final Report:** Announce the completion of the synchronization process and provide a summary of the actions taken.
    -   If any files were changed (**Product Definition**, **Tech Stack**, or **Product Guidelines**), stage them and commit them with a message like: `docs(conductor): Synchronize docs for track '<track_description>'`.

---

## 5. Completion and Handoff

Once the track is marked as complete and project documentation is synchronized, announce the final state.

1.  **Summary:** Present a summary of the implementation (e.g., tasks completed, documentation updated).
2.  **Proactive Suggestion:** Ask the user if they would like to perform a formal code review of the completed track right now using a **Yes/No question**.
3.  **Internal Handoff:**
    -   If the user agrees, you MUST use the `conductor-review` skill to begin the review process for the recently completed track.
    -   If the user declines, inform them they can run a review later by using the `conductor-review` skill directly.
