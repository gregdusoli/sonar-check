---
name: sonar-check
description: >
  Performs real Sonar analysis on modified files via MCP SonarQube integration.
  Strictly requires sonar-project.properties and a live MCP connection.
  Respects project-specific coverage inclusions/exclusions and evaluates
  ONLY new and modified files — pre-existing issues are always ignored.
  Triggers on /sonar-check command in the IDE Chat.
  ⚠️ FIRST ACTION ON INVOCATION: before executing any step, emit exactly this
  line in chat: "> ⚠️ **Modelo recomendado: Claude (Anthropic)**. Invocar esta skill com `Auto Mode` ativado pode causar comportamentos imprevisíveis e produzir resultados inconsistentes."
---

# Skill: Sonar Check

> **STOP. Before anything else:** emit this line in chat now:
> `> ⚠️ **Modelo recomendado: Claude (Anthropic)**. Invocar esta skill com \`Auto Mode\` ativado pode causar comportamentos imprevisíveis e produzir resultados inconsistentes.`
> Only after emitting it, continue reading and executing this skill.

---

## 🚫 Strict Communication Protocol (MANDATORY)

**GOLDEN RULE:** Chat output MUST consist **EXCLUSIVELY** of the content from
the appropriate file in `./references/`, after substituting all
`SKILL_PLACEHOLDER_*` values with real data. Strictly follow the spacing and
line breaks defined in those files. Do NOT append any extra content after the
template — including paths, summaries, or explanations. The ONLY exceptions are:

1. The model warning line at the top of this skill, emitted before any other action.
2. The Gate 3 abort message emitted when neither analysis mode is available
   (Scenario 1 — not configured, or Scenario 2 — MCP error). See Gate 3.

Internal steps, diagnostics, and remediation content are NEVER emitted in chat —
they go exclusively into the `.plan.md` file written to disk.

**ANALYSIS SOURCE OF TRUTH (NON-NEGOTIABLE):**

- ALL findings MUST come exclusively from MCP SonarQube tool responses.
- NEVER use own model knowledge, heuristics, or external tools (ESLint, TSLint,
  Prettier, etc.) to generate or infer findings.
- NEVER report an issue that was not explicitly returned by an MCP tool call.
- If an MCP tool returns no issues for a file, that file has zero issues — period.

---

## Phase 0 — Environment Detection & Validation Gates (sequential)

Execute all steps silently. Do not print anything to chat during this phase,
except where explicitly instructed to emit the README path (Gate 3 failure).

All gates are sequential. The first failure aborts execution immediately.
No partial analysis. No fallback scan.

---

### Gate 1 — sonar-project.properties (sine qua non)

Check for the existence of `sonar-project.properties` in the project root.

- **NOT FOUND → ABORT IMMEDIATELY.** Emit `./references/neutral.md` with
  state: `"Projeto não integrado ao Sonar"`.

> This is the absolute prerequisite. Its absence means the project has no
> Sonar integration whatsoever. Do not proceed to any other gate.

---

### Gate 2 — MCP connectivity & project validation

Invoke:

```
get_project_quality_gate_status(projectKey = "{sonar.projectKey}")
```

- **INVOCATION THROWS / TOOL UNAVAILABLE → ABORT.** Emit `./references/error.md`.
- **PROJECT NOT FOUND / PERMISSION DENIED → ABORT.** Emit `./references/error.md`
  with detail: `"Projeto não encontrado ou sem permissão no SonarQube"`.
- **SUCCESS:** Capture gate conditions and thresholds (e.g. `new_coverage`
  minimum, `new_violations` limit) for reference in Phase 2.

> **CRITICAL:** The project-level gate status (`PASSED`/`FAILED`) returned here
> reflects the ENTIRE project including pre-existing issues. It MUST NOT be
> used to determine Case A or Case B. Case determination in Phase 2 is based
> EXCLUSIVELY on issues found in the delta files by Phase 1.2.

---

### Emitting `success.md` (Case A)

**Step 1 — MANDATORY tool call:** Use the file reading tool to open
`./references/success.md`. Do NOT skip this step. Do NOT use memory or
prior knowledge of this file's content. The file must be read from disk
now, in this step, before any output is produced.

**Step 2 — Emit:** Output the **exact string returned by the tool in Step 1**
directly into chat as formatted Markdown. Do not modify a single character:

- Do not remove or replace emojis
- Do not convert `##` or `###` headings to bold (`**text**`)
- Do not wrap in a code block, backticks, or any fenced block
- Do not rewrite, paraphrase, summarise, or add anything

**After emitting `success.md` — STOP.**

---

### Emitting `neutral.md` (Case C)

**Step 1 — MANDATORY tool call:** Use the file reading tool to open
`./references/neutral.md`. Do NOT skip this step. Do NOT use memory or
prior knowledge of this file's content. The file must be read from disk
now, in this step, before any output is produced.

**Step 2 — Emit:** Output the **exact string returned by the tool in Step 1**
directly into chat as formatted Markdown. Do not modify a single character:

- Do not remove or replace emojis
- Do not convert `##` or `###` headings to bold (`**text**`)
- Do not wrap in a code block, backticks, or any fenced block
- Do not rewrite, paraphrase, summarise, or add anything

**After emitting `neutral.md` — STOP.**

---

### Emitting `error.md` (Case D)

**Step 1 — MANDATORY tool call:** Use the file reading tool to open
`./references/error.md`. Do NOT skip this step. Do NOT use memory or
prior knowledge of this file's content. The file must be read from disk
now, in this step, before any output is produced.

**Step 2 — Emit:** Output the **exact string returned by the tool in Step 1**
directly into chat as formatted Markdown. Do not modify a single character:

- Do not remove or replace emojis
- Do not convert `##` or `###` headings to bold (`**text**`)
- Do not wrap in a code block, backticks, or any fenced block
- Do not rewrite, paraphrase, summarise, or add anything

**After emitting `error.md` — STOP.**

---

### Emitting `failure.md` (Case B)

**Step 1 — MANDATORY tool call:** Use the file reading tool to open
`./references/failure.md`. Do NOT skip this step. Do NOT use memory or
prior knowledge of this file's content. The file must be read from disk
now, in this step, before any output is produced.

**Step 2 — Substitute:** In the string returned by the tool, replace the
three placeholders with real data:

1. **`SKILL_PLACEHOLDER_FILES`** — one line per file that has findings:

   ```
   - src/path/to/file.ts (2 violação/ões)
   - src/path/to/other.ts (1 violação/ão)
   ```

   Use relative paths (strip the absolute project root prefix).

2. **`SKILL_PLACEHOLDER_PLAN_NAME`** — filename only:

   ```
   sonar-check_{projectKey}_{hash}.plan.md
   ```

3. **`SKILL_PLACEHOLDER_PLAN_RELATIVE_PATH`** — path relative to the
   **project root** (directory containing `sonar-project.properties`).
   MUST start with `.cursor/plans/`. No leading `/` or `~` or absolute prefix.
   ```
   .cursor/plans/sonar-check_{projectKey}_{hash}.plan.md
   ```

**Step 3 — Emit:** Output the substituted string directly into chat as
formatted Markdown. Do not modify any other character:

- Do not remove or replace emojis
- Do not convert `##` or `###` headings to bold (`**text**`)
- Do not wrap in a code block, backticks, or any fenced block
- Do not rewrite, paraphrase, summarise, or add anything

**After emitting `failure.md` — STOP. Do not append anything.**

---

### Gate 3 — Analysis mode detection

This gate determines which local analysis tool is available and sets the
`ANALYSIS_MODE` variable used in Phase 1.2. Test in this exact order.

**Error classification used in this gate:**

- **TOOL NOT FOUND** — the tool does not exist in the current MCP context
  (the agent cannot see it in the list of available tools). This means the
  MCP server is not configured for this tool at all.
- **EXECUTION ERROR** — the tool exists and was invoked, but returned a
  connection error, authentication failure, timeout, or server-side error.
  This means the MCP is configured but not healthy.

---

#### Step 3a — Probe `analyze_code_snippet` (Mode 1 — preferred)

> **PROBE RULE (CRITICAL):** This call is a connectivity probe ONLY.
> `DELTA_FILES` does NOT exist yet at this point — do NOT use it here.
> Use a fixed dummy payload. Findings are irrelevant and must be discarded.

Invoke with a minimal dummy payload:

```
analyze_code_snippet(
  projectKey  = "{sonar.projectKey}",
  fileContent = "class Probe {}",
  language    = "typescript"
)
```

- **SUCCESS (any response, including empty findings) → SET** `ANALYSIS_MODE = snippet`.
  Do not proceed to Step 3b. Proceed to Gate 4.
- **TOOL NOT FOUND or EXECUTION ERROR → proceed to Step 3b.**
  Internally note the failure type for use in the final abort message if
  Step 3b also fails.

> Mode 1 is preferred. It runs the SonarQube analyzer engine directly via the
> Docker MCP container, requires no IDE plugin, and works in all environments
> including Claude Code.

---

#### Step 3b — Probe `analyze_file_list` (Mode 2 — fallback)

> **PROBE RULE (CRITICAL):** This call is a connectivity probe ONLY.
> `DELTA_FILES` does NOT exist yet at this point — do NOT use it here.
> Use `sonar-project.properties` as the probe file. Its findings are
> irrelevant and must be discarded entirely. The probe result is binary:
> tool responds = Mode 2 available; tool throws = Mode 2 unavailable.

Invoke using the `sonar-project.properties` absolute path as a harmless probe:

```
analyze_file_list(
  file_absolute_paths = ["{absolute_path_to_sonar-project.properties}"]
)
```

- **SUCCESS (any response, regardless of findings content) → SET**
  `ANALYSIS_MODE = ide`. Discard all findings from this probe call entirely.
  Proceed to Gate 4.
- **TOOL NOT FOUND or EXECUTION ERROR → ABORT.** Emit the appropriate message
  below based on the combined failure types from Steps 3a and 3b, then stop.

---

#### Gate 3 abort messages (emit in chat, then stop)

**Scenario 1 — MCP not configured** (both tools returned TOOL NOT FOUND):

```
⚙️ Configuração necessária. Consulte o guia: {absolute_path_to_skill_dir}/README.md
```

**Scenario 2 — MCP configured but not healthy** (at least one tool returned
EXECUTION ERROR, meaning it exists but the server is failing):

```
🔴 O MCP do SonarQube está configurado mas não está respondendo corretamente.

Causas mais comuns:
1. O plugin SonarQube for IDE não está em Connected Mode — abra o plugin na IDE
   e conecte-o à organização corporativa no SonarCloud antes de tentar novamente.
2. Há múltiplos containers Docker do MCP em execução simultaneamente — isso ocorre
   quando o botão "Configure MCP Server" do plugin cria uma segunda entrada no mcp.json
   sem remover a anterior. Verifique com `docker ps | grep mcp/sonarqube` e pare todos
   os containers duplicados, mantendo apenas um.

Consulte o guia: {absolute_path_to_skill_dir}/README.md
```

> `{absolute_path_to_skill_dir}` is the directory containing this SKILL.md
> file, resolved at runtime.

> Mode 2 requires the SonarQube for IDE plugin installed, running, and in
> Connected Mode. NOT available in Claude Code.

---

### Gate 4 — Delta detection (DELTA_FILES establishment — mandatory before any analysis)

> **HARD RULE:** No analysis tool (`analyze_code_snippet`, `analyze_file_list`,
> `search_security_hotspots`, `get_file_coverage_details`, or any other tool
> that operates on source files) may be called before this gate completes
> successfully and `DELTA_FILES` is established. Any tool call on source files
> before this point is a protocol violation.

```bash
git status --short
```

**Keep ONLY files matching ALL of these criteria:**

- Status: Staged (`A`, `M` in index) OR unstaged modified (`M` in worktree)
  OR untracked (`??`) — new files not yet added to the index are included
- Path: under `sonar.sources` value from properties file
  (defaults: `src/`, `lib/`, `packages/`, `app/`)

**Exclude unconditionally regardless of path:**

- Extensions: `.md`, `.json`, `.yaml`, `.yml`, `.txt`, `.cursorrules`
- Patterns: `.env*`, `*lock.json`, `yarn.lock`, `pnpm-lock.yaml`

**IF filtered list is empty → ABORT IMMEDIATELY.** There is nothing to analyse.
Emit `./references/neutral.md` with state: `"Nenhuma alteração em código-fonte detectada"`.
Do not proceed to Phase 1.

**SUCCESS:** Store the filtered list as `DELTA_FILES`. Only now may Phase 1 begin.

---

## Phase 1 — Context Sync & Local Analysis (silent)

### 1.1 — Policy & Gate Sync via MCP

Read from `sonar-project.properties`:

- `sonar.projectKey`
- `sonar.test.inclusions`
- `sonar.coverage.exclusions`
- `sonar.sources`

Invoke (failures = **partial degradation** — continue, flag under `⚠ Dado indisponível`):

```
list_quality_gates()
```

→ Find the active gate (the one with `isDefault: true` or matching the project).
→ Store ALL its conditions as `GATE_CONDITIONS`. Example structure:

```
GATE_CONDITIONS = [
  { metric: "new_reliability_rating", op: "GT", error: "1" },
  { metric: "new_security_rating",    op: "GT", error: "1" },
  { metric: "new_coverage",           op: "LT", error: "80" },
  ...
]
```

> **This is the source of truth for Case determination in Phase 2.**
> These are the EXACT conditions the pipeline uses. Do not substitute
> with hardcoded severity thresholds.

```
get_component_measures(
  component  = "{sonar.projectKey}",
  metricKeys = "new_coverage,new_violations,new_bugs,
                new_vulnerabilities,new_code_smells,
                new_security_hotspots,coverage"
)
```

→ Capture current New Code period metrics for the Coverage Strategy section.

---

### 1.2 — Local Analysis (bifurcated by ANALYSIS_MODE)

Execute ONLY the branch matching the `ANALYSIS_MODE` set in Gate 3.
Never mix calls from both branches.

---

#### MODE 1 — `analyze_code_snippet` (ANALYSIS_MODE = snippet)

For **each file** in `DELTA_FILES`, read the complete file content from disk,
then invoke:

```
analyze_code_snippet(
  projectKey  = "{sonar.projectKey}",
  fileContent = "{complete current on-disk content of the file}",
  language    = "{detected language: typescript|javascript|java|python|...}",
  scope       = "MAIN"
)
```

- Call **once per file**. Do NOT batch multiple files in one call.
- `fileContent` MUST be the current on-disk content — not the last committed
  version. This is what enables genuine pre-push local analysis.
- `projectKey` connects the analysis to the corporate quality profile and
  rules in SonarCloud, ensuring corporate rule compliance.
- Findings are structurally scoped to this file's current content only.
  Pre-existing issues from other files cannot appear.

---

#### MODE 2 — `analyze_file_list` (ANALYSIS_MODE = ide)

Invoke once with all delta files:

```
analyze_file_list(
  file_absolute_paths = [
    "{absolute_path_to_delta_file_1}",
    "{absolute_path_to_delta_file_2}",
    ...
  ]
)
```

- Pass absolute paths. The plugin resolves corporate rules via Connected Mode.
- Findings are scoped to the provided file list only.

---

#### Post-analysis mandatory filter (both modes)

After receiving findings from either mode, apply this filter before bucketing.

**Path matching (CRITICAL — absolute vs relative):**
`analyze_file_list` returns absolute paths in `filePath`. `DELTA_FILES` contains
relative paths from `git status`. Normalize before comparing:

- A finding is IN scope if its `filePath` **ends with** any path in `DELTA_FILES`.
- Example: `/Users/dev/project/src/foo.ts` ends with `src/foo.ts` → IN scope.
- Discard any finding whose `filePath` does not end with a `DELTA_FILES` entry.

**Status field:**
`analyze_file_list` findings do NOT include a `status` field — they are always
current local analysis results. Do NOT filter by status for Mode 2 findings.
Retain ALL findings whose path matches a `DELTA_FILES` entry.
For Mode 1 (`analyze_code_snippet`), same rule applies — no status filter.

For any retained finding where rule detail is needed:

```
show_rule(ruleKey = "{finding.rule}")
```

**Hotspot fetch — call ONCE PER FILE (both modes):**

```
search_security_hotspots(
  projectKey      = "{sonar.projectKey}",
  files           = "{relative_path_of_single_delta_file}",
  sinceLeakPeriod = true
)
```

For individual hotspot detail when available:

```
show_security_hotspot(hotspotKey = "{hotspot.key}")
```

**Coverage detail per delta file (both modes):**

```
get_file_coverage_details(key = "{projectKey}:{relative_path}")
```

**Aggregate ALL retained findings into a single `DELTA_FINDINGS` list.**

Do NOT pre-classify by severity or type at this stage. Retain every field
returned by the tool: `severity`, `message`, `filePath`, `textRange`,
`rule` (if present), `type` (if present).

> **CRITICAL — missing fields in Mode 2:** `analyze_file_list` does NOT return
> `type` or `rule` fields — only `severity`, `message`, `filePath`, `textRange`.
> Do NOT attempt to infer type from message text.
> Apply the DEFAULT RULE below instead.

---

## Phase 2 — Workflow Decision & Template Mapping

> **BURDEN OF PROOF RULE (NON-NEGOTIABLE):**
> The SonarQube for IDE plugin (Mode 2) and the SonarQube analyzer (Mode 1)
> only return findings that the corporate quality profile considers violations.
> If a finding was returned for a delta file, it IS a violation — period.
> The skill does NOT need to match findings to gate conditions to classify
> them as violations. That matching is done by the Sonar engine before
> returning results.

### Step 2.1 — Default rule: ALL delta findings = gate violations

**`gate_violations` = ALL entries in `DELTA_FINDINGS`** (after path filter).

There are NO non-blocking findings from `analyze_file_list` or
`analyze_code_snippet`. Every finding returned by these tools is a real
Sonar rule violation detected in the current local code.

> This rule exists because `analyze_file_list` does not return `type` or
> `rule` fields, making it impossible to map findings to specific gate
> conditions. But this is irrelevant: the tool already applies the corporate
> quality profile internally. If it returned the finding, the pipeline
> would flag it.

**`improvements`** = empty (no findings are non-blocking from analysis tools).

**`hotspots`** = results from `search_security_hotspots` (informational only,
never affect Case).

### Step 2.2 — Select template

| Case                                | Condition                                   | Template                        |
| ----------------------------------- | ------------------------------------------- | ------------------------------- |
| **A — Compliant**                   | `DELTA_FINDINGS` is empty after path filter | `success.md`                    |
| **B — Violations**                  | `DELTA_FINDINGS` has one or more entries    | `failure.md` → then **Phase 3** |
| **C — No Integration / No Changes** | Gate 1 failed, or delta empty after filter  | `neutral.md`                    |
| **D — Connection Error**            | Gate 2 throws or project not found          | `error.md`                      |

**Hotspots** never determine Case. Reported as `INFO` in plan section 5 only.

**`GATE_CONDITIONS`** from `list_quality_gates` is used exclusively for the
plan header (which gate conditions exist) and Coverage Strategy section
(coverage threshold). It does NOT gate-keep Case determination.

---

## Phase 3 — Mandatory Plan Mode (Case B only)

### Plan file path

```
{project_root}/.cursor/plans/sonar-check_{projectKey}_{timestamp_hash}.plan.md
```

`{timestamp_hash}` = first 8 hex chars of `sha256(current_unix_timestamp_ms)`.

Example: `.cursor/plans/sonar-check_my-project_a3f2c1b8.plan.md`

Create `.cursor/plans/` directory if it does not exist.

### Plan format

Read `./references/plan.md` and substitute every `{{placeholder}}` with real
data before writing to disk. Rules:

| Placeholder               | Value                                                                               |
| ------------------------- | ----------------------------------------------------------------------------------- |
| `{{projectKey}}`          | `sonar.projectKey` from properties file                                             |
| `{{analysisMode}}`        | `Modo 1 — analyze_code_snippet` or `Modo 2 — analyze_file_list`                     |
| `{{timestamp}}`           | ISO-8601 current timestamp                                                          |
| `{{deltaFiles}}`          | Real comma-separated relative paths from `DELTA_FILES`                              |
| `{{totalFindings}}`       | Total count of entries in `DELTA_FINDINGS`                                          |
| `{{degradationWarning}}`  | `⚠ Dado indisponível: {tool_name}` if partial degradation occurred, otherwise empty |
| `{{tasks}}`               | One `- [ ]` task per finding — see format below                                     |
| `{{findingsTable}}`       | One table row per finding — see format below                                        |
| `{{remediationSections}}` | One `###` section per finding — see format below                                    |
| `{{newCoverage}}`         | Value from `get_component_measures` new_coverage metric                             |
| `{{gateThreshold}}`       | Coverage threshold from `GATE_CONDITIONS`                                           |
| `{{coverageGap}}`         | `{{gateThreshold}} - {{newCoverage}}` (0 if coverage passes)                        |
| `{{uncoveredFiles}}`      | Delta files without corresponding test files, or `Nenhum`                           |
| `{{sonarCloudUrl}}`       | `SONARQUBE_CLOUD_URL` env variable value                                            |
| `{{hotspotsTable}}`       | One table row per hotspot, or `Nenhum hotspot encontrado.`                          |

**`{{tasks}}` format — one line per finding:**

```
- [ ] **{relative_path}:{startLine}** — {message} _(Severidade: {severity})_
```

**`{{findingsTable}}` format — one row per finding:**

```
| {severity} | {relative_path} | {startLine} | {message} |
```

**`{{remediationSections}}` format — one section per finding:**

```
### {relative_path}:{startLine} — {message}

**Severidade:** {severity}
**Risco técnico:** {technical risk description based on the rule}

**Correção:**
1. {concrete step referencing the Sonar rule}
2. {additional step if needed}

**Exemplo corrigido:**
\`\`\`{language}
// antes
{problematic code}

// depois
{corrected code}
\`\`\`
```

> **NEVER leave any `{{placeholder}}` unreplaced in the output file.**
> If data is unavailable for a placeholder, use `N/A` or `Não disponível`.
> Generic values like `[file_1.ts]` or `[folder_path]` are FORBIDDEN.

---

## MCP Tools Reference

### ✅ MUST USE

| Tool                              | Gate / Phase               | Purpose                                                                                                |
| --------------------------------- | -------------------------- | ------------------------------------------------------------------------------------------------------ |
| `get_project_quality_gate_status` | Gate 2                     | Connectivity check + capture gate thresholds. NOT used for Case determination.                         |
| `search_my_sonarqube_projects`    | Gate 2 (fallback)          | Validate project exists if `get_project_quality_gate_status` throws.                                   |
| `analyze_code_snippet`            | Gate 3a + Phase 1.2 Mode 1 | Probe Mode 1 availability; primary local analysis engine via Docker MCP.                               |
| `analyze_file_list`               | Gate 3b + Phase 1.2 Mode 2 | Probe Mode 2 availability; local analysis via SonarQube for IDE plugin.                                |
| `list_quality_gates`              | 1.1                        | Capture active gate conditions and numeric thresholds.                                                 |
| `get_component_measures`          | 1.1                        | New Code period metrics for Coverage Strategy section.                                                 |
| `search_security_hotspots`        | 1.2                        | Per-file hotspot fetch. Informational only.                                                            |
| `show_rule`                       | 1.2                        | Rule detail by `ruleKey` — required for remediation guidance. Never use model knowledge as substitute. |
| `get_file_coverage_details`       | 1.2                        | Line-level coverage gaps per delta file.                                                               |
| `show_security_hotspot`           | 1.2 (optional)             | Enrich plan section 5 with hotspot category and description.                                           |

### ❌ MUST NOT USE

| Tool                              | Reason                                                                                                                            |
| --------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `search_sonar_issues_in_projects` | Queries server-side committed issues only — cannot see local uncommitted changes. Produces false negatives for pre-push analysis. |
| `search_files_by_coverage`        | Project-wide scope — surfaces pre-existing coverage gaps unrelated to delta.                                                      |
| `get_duplications`                | Not a gate criterion. Out of scope.                                                                                               |
| `search_duplicated_files`         | Same reason as above.                                                                                                             |
| `search_metrics`                  | Discovery utility only. No analysis value.                                                                                        |
| `list_pull_requests`              | Unrelated to local delta analysis.                                                                                                |

### 🚫 FORBIDDEN — DO NOT CALL UNDER ANY CIRCUMSTANCE

These tools mutate project state. This skill is strictly read-only.
Calling any of these against 300+ production projects causes irreversible
side effects.

| Tool                             | Risk                                                                          |
| -------------------------------- | ----------------------------------------------------------------------------- |
| `change_sonar_issue_status`      | Permanently alters issue triage state in SonarCloud for all team members.     |
| `change_security_hotspot_status` | Permanently marks hotspots as reviewed/dismissed without human validation.    |
| `toggle_automatic_analysis`      | Disables/enables automatic analysis at project level — catastrophic at scale. |

---

## Compatibility Notes

### Supported Environments

| Environment                | Mode 1 — `analyze_code_snippet` | Mode 2 — `analyze_file_list`     |
| -------------------------- | ------------------------------- | -------------------------------- |
| **Cursor + Claude**        | ✅ Yes                          | ✅ Yes                           |
| **VS Code + Claude**       | ✅ Yes                          | ✅ Yes                           |
| **IntelliJ + Claude**      | ✅ Yes                          | ✅ Yes                           |
| **Claude Code**            | ✅ Yes                          | ❌ No — IDE plugin not supported |
| **Cursor + default model** | ⚠️ Uncertain                    | ⚠️ Uncertain                     |
| **Claude.ai**              | ❌ No                           | ❌ No                            |

### Mode 1 prerequisites

- SonarQube MCP docker container running locally.
- Environment variables set: `SONARQUBE_TOKEN`, `SONARQUBE_ORG`,
  `SONARQUBE_CLOUD_URL`, `SONARQUBE_IDE_PORT`.
- `analyze_code_snippet` is available only when MCP runs as a standalone
  Docker container — it is disabled in the MCP embedded in SonarQube Cloud.

### Mode 2 prerequisites

- SonarQube for IDE plugin installed and running in the editor.
- Plugin configured in Connected Mode to the corporate SonarCloud organization.
- MCP docker container running with `SONARQUBE_IDE_PORT` set so the MCP
  can communicate with the plugin instance.

### Why Claude.ai is not supported

- No local terminal — Gate 4 cannot run `git status`.
- No local filesystem — Gate 1 cannot read `sonar-project.properties`;
  Phase 3 cannot write `.plan.md`.
- No local Docker MCP connectivity.

### Setup guide

If Gate 3 fails (neither mode available), the skill emits the path to
`{absolute_path_to_skill_dir}/README.md`. Follow the instructions there
to configure Mode 1 or Mode 2.

---

## 🚀 How to Invoke

In the IDE Chat, type `/sonar-check`.
