---
name: ai-setup-doctor
description: Use when the user asks to check, verify, diagnose, or fix their AI assistant setup — "check my AI setup", "is my agent setup working", "run the setup doctor", "why doesn't Claude see my skills / my instructions / my rulebook" — and also after cloning a repo that uses AGENTS.md + .agents/skills onto a new machine, or whenever instructions or skills appear not to load. Runs a concrete checklist over the project's instruction files, shims, and skill links, then reports pass/warn/fail with a copy-pasteable fix for every failure.
metadata:
  author: custom (this starter kit)
  version: "1.1"
---

# AI setup doctor

Verify that a project assembled from the AI starter kit actually loads its
instructions and skills. Every check below is a file check you can run with your
own tools — no scripts to install. The setup fails *silently* when broken (the
assistant just behaves generically), which is why this checklist exists.

Run read-only. Propose fixes; apply one only after the user confirms. Never
rewrite `AGENTS.md` content beyond the `## Project` header placeholders, and only
when the user supplies the real values.

## Scope

Check exactly one project: the nearest directory at or above the working
directory that contains `AGENTS.md` or `CLAUDE.md`. Never scan sibling or
parent directories for other projects. If the working directory is not inside
a project (e.g. a folder of repos), ask the user which project to check — do
not pick one or check several.

## Report format

Always finish with this exact structure — a table, then a one-line verdict:

```
| Check | Status | Fix |
|---|---|---|
| AGENTS.md present | PASS | — |
| Project header filled | WARN | give me your build/test/run commands and I'll fill them in |
| .claude/skills resolves | FAIL | New-Item -ItemType Junction .claude\skills -Target (Resolve-Path .agents\skills) |
...

Verdict: 2 problems — instructions load, but Claude Code cannot see your skills.
```

Every FAIL row carries a command the user can paste as-is (match their OS).
WARN means "works, but degraded or non-portable"; explain the consequence in the
verdict, not the table.

## The checklist

Run all checks even after the first failure — the user wants the full picture.

1. **`AGENTS.md` exists at the project root.** Missing → FAIL; fix: copy
   `starter-kit/base/AGENTS.md` from the kit and re-run assembly
   (the kit's `handbook/04-new-project.md` is the full procedure).

2. **`## Project` header is filled in.** The Name/Build/Test/Run lines must not
   contain `<...>` placeholders. Placeholders → WARN: an assistant that can't run
   the real test command can't verify anything it builds. Fix: ask the user for
   the real commands and offer to fill them in.

3. **`CLAUDE.md` exists and contains a line `@AGENTS.md`.** Extra content is
   allowed; the import line is what makes Claude Code load the rulebook (it does
   not read a bare `AGENTS.md`). Missing → FAIL; fix: create `CLAUDE.md`
   containing exactly `@AGENTS.md`.

4. **`.claude/skills` exists and resolves.** Inspect it:
   - Windows: `Get-Item .claude\skills | Select-Object LinkType,Target`
   - macOS/Linux: `ls -l .claude/skills`
   - A junction/symlink pointing at `.agents/skills` → PASS.
   - A real directory with skills inside → WARN: valid (this is the kit's
     mouse-only layout) but only Claude Code sees these skills; the portable
     layout is `.agents/skills/` + link.
   - Missing, or a link whose target doesn't exist → FAIL; fix:
     - Windows: `New-Item -ItemType Junction .claude\skills -Target (Resolve-Path .agents\skills)`
     - macOS/Linux: `ln -s ../.agents/skills .claude/skills`
     (remove the dangling link first if there is one).

5. **Every skill is readable.** Each directory under the resolved skills path must
   contain a `SKILL.md` whose frontmatter has `name` and `description`. A dir
   without one → FAIL for that skill; fix: re-copy it from the kit's
   `starter-kit/skills/`.

6. **`.agents/skills/` exists** (the assistant-agnostic home). Skills living only
   in `.claude/skills` → WARN: Copilot/Codex/other assistants won't find them;
   fix: move to `.agents/skills/` and link as in check 4.

7. **`.claude/settings.json` parses as JSON** (if present). Invalid → FAIL with the
   parse error. If valid, advisory: compare its `permissions.allow` entries with
   the Project header's build/test commands — a header saying `./gradlew build`
   while the allowlist only covers `./mvnw` means permission prompts on every
   build; suggest the matching entries.

8. **`GEMINI.md` is identical to `AGENTS.md`** (if present). Compare byte-for-byte
   (`fc.exe AGENTS.md GEMINI.md` / `cmp -s AGENTS.md GEMINI.md && echo same`).
   Drift → FAIL: the two assistants are following different rules, which is
   miserable to debug; fix: re-copy `AGENTS.md` over `GEMINI.md`.

9. **`.gitignore` covers the link.** If `.claude/skills` is a junction/symlink,
   `.gitignore` should contain `.claude/skills/` so the linked content isn't
   committed twice. Missing → WARN; fix: `Add-Content .gitignore ".claude/skills/"`
   / `echo ".claude/skills/" >> .gitignore`.

10. **Addons match the project** (advisory only — never edit). Compare the tech
    sections appended to `AGENTS.md` against the evidence on disk: `pom.xml` or
    `build.gradle` → expect a Spring Boot section; `package.json` with `vue` or
    `@angular/core` → expect the matching frontend section;
    `src/main/resources/templates/` → expect Thymeleaf. Report mismatches
    (missing or leftover sections) as WARN with a pointer to the kit's
    `starter-kit/addons/`.

## Live smoke test

The file checks prove the wiring; this proves the load. You can self-verify:
quote the `**Name:**` line of the `## Project` header *from your own loaded
instructions* (not by reading the file), and list the skills your harness
provides, comparing them against the directories on disk from check 5. If you
cannot quote the header, `CLAUDE.md` isn't loading; if skills are missing from
your list, the link from check 4 is broken — even if the files look right.

To verify a different assistant or a fresh session, give the user this probe:

```
claude -p --model haiku "Quote the Project Name line from your instructions and list every skill available to you."
```

Pass = the reply quotes the exact project name and names every skill directory
present on disk (count them). A generic answer means instructions aren't
loading; a partial skill list means the link or a copy is broken.
