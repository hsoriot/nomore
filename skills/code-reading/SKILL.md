---
name: code-reading
description: Deep code reading and comprehension for a specific file, function, or context. Use when the user asks to understand, explain, or analyze a piece of code — including its call graph, dependencies, business context, and design rationale. Produces a top-down structured HTML document with concrete evidence (actual code references, signatures, and data flow). Output is a self-contained .html file for human reading.
---

# Code Reading

Generate a structured, top-down comprehension document for a given code target (file, function, class, or context). Output is a **self-contained HTML file** using the template at `references/template.html`.

## Trigger

Activate when the user says things like:
- "Help me understand this file/function/module"
- "Explain what X does"
- "Read through this code and tell me what's going on"
- "Analyze the design of X"

## Input

The user provides one or more of:
- A file path
- A function/class name
- A description of a context or feature area

## Process

### Phase 1: Locate and Anchor

1. If given a file path, read the full file.
2. If given a function/class name, search the codebase to locate its definition.
3. If given a context description, identify the entry point(s) that best match.
4. Record the exact location: `file_path:line_number`.

### Phase 2: Top-Down Analysis

Analyze in this order (general to specific):

1. **Purpose** — What problem does this code solve? One sentence.
2. **Position in System** — Where does this sit in the overall architecture? What layer/module does it belong to?
3. **Public Interface (Spec)** — For each exported/public function or method:
   - Signature (params with types, return type)
   - What it does (one line)
   - Preconditions / side effects if any
4. **Call Graph (Outgoing)** — What does this code call? List direct dependencies:
   - Internal calls (same repo) — with `file_path:line_number`
   - External calls (libraries/frameworks)
5. **Call Graph (Incoming)** — Who calls this code? Find callers in the codebase.
6. **Data Flow** — Trace key data: what comes in, how it's transformed, what goes out.
7. **Business Scenario** — In what user-facing or system scenario is this code executed? Describe the concrete trigger (e.g., "user clicks submit", "cron job fires at midnight").
8. **Design Rationale** — Why is it designed this way? Look for:
   - Patterns used (factory, observer, middleware, etc.)
   - Trade-offs made (performance vs readability, etc.)
   - Comments or commit messages that reveal intent

### Phase 3: Evidence Collection

For every claim, provide concrete evidence:
- Quote the relevant code snippet (keep it short, 3-8 lines)
- Reference exact locations as `file_path:line_number`
- If inferring intent without direct evidence, mark with `<span class="inferred">inferred</span>`

## Output

### Read the template

Read `references/template.html` from this skill's directory. This is a self-contained HTML page with embedded CSS.

### Fill the template

Replace all `{{PLACEHOLDER}}` tokens with actual analysis content. Handle repeated sections (`{{#EACH ...}} ... {{/EACH}}`) by duplicating the inner HTML block for each item.

Key placeholders:
- `{{TARGET_NAME}}` — Name of the function/class/file analyzed
- `{{PURPOSE}}` — One-sentence purpose
- `{{FILE_PATH}}:{{LINE_NUMBER}}` — Source location
- `{{POSITION_DESCRIPTION}}` — System position description
- `{{#EACH INTERFACE}}` — Repeat for each public function/method
  - `{{SIGNATURE}}` — Full signature with types
  - `{{DESCRIPTION}}` — What it does, preconditions, side effects
- `{{#EACH OUTGOING}}` — Repeat for each outgoing dependency
  - `{{CALLEE_NAME}}`, `{{CALLEE_LOCATION}}`, `{{CALLEE_ROLE}}`
- `{{#EACH INCOMING}}` — Repeat for each caller
  - `{{CALLER_NAME}}`, `{{CALLER_LOCATION}}`, `{{CALLER_CONTEXT}}`
- `{{#EACH DATAFLOW}}` — Repeat for each data flow step
  - `{{STEP}}` — Description of this transformation step
- `{{BUSINESS_SCENARIO}}` — Concrete business scenario description
- `{{DESIGN_RATIONALE}}` — Design reasoning (use `<span class="inferred">inferred</span>` tag where applicable)
- `{{#EACH EVIDENCE}}` — Repeat for each code evidence block
  - `{{EVIDENCE_LABEL}}` — Short description of what the snippet shows
  - `{{EVIDENCE_LOCATION}}` — `file_path:line_number`
  - `{{EVIDENCE_CODE}}` — The code snippet (HTML-escaped, with `<span class="line-num">` for line numbers)
- `{{GENERATED_DATE}}` — ISO date string of generation time

### Write the file

Write the completed HTML to the project directory as:
```
.code-reading/<target-name>.html
```

Where `<target-name>` is the function/class/file name in kebab-case. Create the `.code-reading/` directory if it does not exist.

After writing, tell the user the file path so they can open it in a browser.

## Rules

1. **Top-down only** — Start from purpose, end at implementation. Never begin with line-by-line walkthrough.
2. **Evidence-based** — Every section must reference real code locations. No hand-waving.
3. **Spec-first for functions** — Always state the interface (params + return) before explaining internals.
4. **Bounded depth** — Analyze the target and its direct neighbors (1 hop). Do not recursively deep-dive into every dependency unless the user explicitly asks.
5. **Mark uncertainty** — If the business scenario or design rationale is not obvious from code alone, use `<span class="inferred">inferred</span>` and explain reasoning.
6. **Concise over verbose** — Prefer a table row over a paragraph. The reader wants to scan fast.
7. **Preserve naming** — Use the exact function/variable names from the code. Do not rename or abstract them.
8. **Valid HTML** — The output must be a valid, self-contained HTML file that renders correctly when opened directly in a browser. Escape special characters in code snippets (`<`, `>`, `&`).
