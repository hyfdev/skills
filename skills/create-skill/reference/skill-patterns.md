## Degrees of Freedom

Match instruction specificity to the task's fragility and variability:

- **High freedom** (text-based instructions) — Use when multiple approaches are valid and decisions depend on context. Example: code review guidelines.
- **Medium freedom** (pseudocode or scripts with parameters) — Use when a preferred pattern exists but some variation is acceptable. Example: report generation with configurable sections.
- **Low freedom** (specific scripts, few parameters) — Use when operations are fragile, consistency is critical, or a specific sequence must be followed. Example: database migrations that must run in exact order.

Think of the agent exploring a path: a narrow bridge with cliffs needs specific guardrails (low freedom), while an open field allows many routes (high freedom).

## Skill Architecture Patterns

Choose the pattern that best fits the skill's use case. Most skills combine one architecture pattern with one or more content patterns.

### 1. Sequential Workflow

Multi-step processes executed in a specific order.

**Key elements:**
- Explicit step ordering with numbered steps
- Dependencies between steps (step 2 needs output of step 1)
- Validation at each stage before proceeding
- Rollback instructions for failures

**Best for:** Onboarding flows, setup wizards, deployment pipelines, scaffolding.

**Structure:**
```markdown
## Workflow
### Step 1: [Action]
[Instructions, validation criteria]
### Step 2: [Action]
[Instructions, depends on Step 1 output]
### Step 3: [Action]
[Instructions, validation, completion criteria]
```

### 2. Iterative Refinement

Output quality improves through repeated cycles.

**Key elements:**
- Initial draft generation
- Quality check (script, checklist, or acceptance criteria)
- Fix identified issues
- Re-validate
- Repeat until quality threshold met

**Best for:** Report generation, content creation, code review, test coverage improvement.

### 3. Context-Aware Tool Selection

Same outcome achieved with different tools depending on the environment.

**Key elements:**
- Decision tree for tool selection (check what is available, then choose)
- Fallback options when primary tool is unavailable
- Transparency about which tool was chosen and why

**Best for:** File operations across OS environments, data processing with multiple possible libraries, deployment to different platforms.

### 4. Domain-Specific Intelligence

The skill's primary value is specialized knowledge beyond tool access.

**Key elements:**
- Domain expertise embedded directly in the instructions
- Compliance rules or constraints checked before actions
- Audit trail of decisions and rationale

**Best for:** Compliance checking, specialized analysis (security, performance, accessibility), domain-specific code generation.

### 5. Multi-MCP Coordination

Workflows that span multiple MCP servers or external services.

**Key elements:**
- Clear phase separation (fetch from service A, process, send to service B)
- Data passing between phases with explicit format
- Validation before each cross-service step
- Error handling for partial failures

**Best for:** Cross-service workflows, data migration, multi-tool orchestration.

---

## Content Patterns

Combine these with the architecture patterns above.

### Task List / Checklist Pattern (High Importance)

Checklists serve two distinct purposes depending on skill type:

**Workflow checklists** — For multi-step workflows, provide a copyable checklist the agent copies into its response and checks off as it progresses.

**Verification checklists** — For domain-knowledge skills, provide a checklist that verifies the knowledge was actually applied in the agent's output. Without this, the skill is just text the agent might read and ignore — there is no mechanism to confirm it was used.

**Why this matters:**
- Prevents the agent from skipping steps or ignoring knowledge
- Makes progress and compliance observable to the user
- Creates a clear contract of what "done" means

**Placement:** Put checklists near the top of the skill when they define the agent's obligations. A checklist buried at the bottom becomes an afterthought — the agent focuses on what it reads first.

**Workflow example:**
```markdown
## Workflow
Task Progress:
- [ ] Step 1: [description]
- [ ] Step 2: [description]
- [ ] Step 3: [description]
```

**Domain-knowledge (checklist-first) example:**
```markdown
# [Skill Name]

[Opening line — why this knowledge matters]

## Verification checklist
- [ ] [Check that knowledge point 1 was applied]
- [ ] [Check that knowledge point 2 was applied]
- [ ] [Check that knowledge point 3 was applied]

## [How it works — mental model]
## [Details per check — expanding each item with examples]
```

Include a workflow checklist for any workflow with 3+ steps. Include a verification checklist for any domain-knowledge skill that teaches practices, constraints, or patterns the agent should apply.

### Examples Pattern

Input/output pairs showing correct format or style.

**Use when:** Output quality depends on seeing examples — commit messages, report formats, code style conventions.

**Format:** Show 2-3 pairs of input -> expected output, covering typical and edge cases.

### Template Pattern

Exact output format the agent fills in.

**Use when:** Output must follow a strict structure — API responses, data formats, config files.

**Strictness levels:**
- "ALWAYS use this exact structure" — for rigid requirements
- "Sensible default, adapt as needed" — for flexible templates
