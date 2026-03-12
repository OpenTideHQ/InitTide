# AGENTS.md - Agent Instructions for OpenTide Repository

You are working on an OpenTide (Open Threat Informed Detection Engineering) repository.

OpenTide is a framework that enables the creation of YAML files called Objects, which model the detection engineering lifecycle. OpenTide manages the complete corpus of knowledge around threats and detection research, alongside detection rules deployment in an as-code manner.

> For architectural details (remote vs. local CoreTide setup, CI/CD integration, secrets management), refer to `README.md`.

## Tech Stack & Tooling

| Technology | Purpose |
|---|---|
| **YAML** | All object files (TVMs, DOMs, MDRs, etc.) |
| **JSON Schema** | Validation schemas in `Schemas/*.json` |
| **TOML** | Configuration files in `Configurations/` |
| **VS Code** | Primary IDE with schema validation, snippets, and recommended extensions |
| **CoreTide** | Remote framework repository (CI/CD pipelines, validation scripts) |

**VS Code integration (pre-configured in `.vscode/`):**
- Schema-backed YAML validation via `redhat.vscode-yaml` — schemas auto-map to object files
- Code snippets for all object types (trigger with prefix, e.g. `☣️ Threat Vectors Template`)
- UUID generation via `motivesoft.vscode-uuid-generator`
- TOML schema validation via `evenBetterToml`

**There are no local build, test, or lint commands.** All validation and CI/CD pipelines are delegated to the CoreTide repository and executed remotely via GitHub Actions, Azure Pipelines, or GitLab CI. Local validation relies on VS Code schema mappings (see `.vscode/settings.json`).

## Prime Directives

- _Correctness_ : You must be exact, and cognitively correct as to how you break down intelligence and complex threat data into modelled OpenTide data.
- _Consistency_ : You must maintain a consistent OpenTide repository. Sometimes, it's better to improve several objects before creating a new one to correctly ingest a change in perspective.
- _Transparency_ : The changes, decisions, proposals you perform must all be rationalized, discoverable and shown to the user.
- _Critical Thinking_ : No vague conclusion, all modelling decision must be critical. You are allowed to disagree with the user, when there is a poor reasoning as to the initial directive.
- _Autonomous_ : As much as possible, you aim at performing end-to-end changes.

### Guardrails

**Only modify content files, not framework files**
- **NEVER** tamper with schemas or templates
- **DO NOT** change configuration files except if explicitly instructed
- **NEVER** create new folders or restructure the repository
- **ALWAYS** avoid calling non-IDE or simple web search tools, unless deemed strictly necessary — if so, interrupt and ask the user.

**Enforce Top Down**
- Given threat intelligence, you create threat object(s).
- Given threat objects, you create detection objective(s).
- Given objective(s), you create rule(s).
- Per run, you avoid mixing object types. You always generate the right object type, and if prompted, you ask if doing it in another MR/PR (and after the object is merged) is not preferable. Mixing object types is NOT the intended workflow.

### Project Structure Integrity
- **NEVER** create new folders unless explicitly requested
- **NEVER** generate or modify UUIDs for existing objects
- **ALWAYS** respect the existing folder structure
- **ALWAYS** validate files against JSON schemas before saving
- **ALWAYS** use templates as a guide for object structure

## OpenTide Framework Concepts

OpenTide structures the Detection Engineering lifecycle as as-code (YAML) objects managed in a git repository. The core object types are:

### 1. Threat Vectors (TVM)
- **Purpose**: Represent atomically defined TTPs (Tactics, Techniques, Procedures) at a low level
- **Source**: Directly generated from threat intelligence
- **Chaining**: Allows interrelating TVMs to one another (bi-directionally) to represent an attack path
- **Schema**: `tvm::2.1` · **Template**: `Schemas/Templates/TVM TEMPLATE.yaml`
- **Location**: `Objects/Threat Vectors/*.yaml`

### 2. Detection Objectives (DOM)
- **Purpose**: Represent detection capabilities for identified threats
- **Relationship**: Support 1:N and N:1 relations with Threat Vectors
- **Components**: Composed of signals (atomic detection rule ideas). Signals can be referenced by Detection Rules.
- **Schema**: `dom::1.0` · **Template**: `Schemas/Templates/Detection Objective.template.yaml`
- **Location**: `Objects/Detection Objectives/*.yaml`

### 3. Detection Rules (MDR)
- **Purpose**: Detection-as-Code files for deployment
- **Relationship**: Directly linked to Detection Objectives, indirectly to Threat Vectors
- **Target**: Can target any configured detection system (Splunk, Sentinel, CrowdStrike, etc.)
- **Schema**: `mdr::2.1` · **Template**: `Schemas/Templates/MDR TEMPLATE.yaml`
- **Location**: `Objects/Detection Rules/*.yaml`

### 4. Lookup Metadata
- **Purpose**: Metadata for lookup tables (CSV reference data) used by detection rules for enrichment and filtering
- **Schema**: `Schemas/Lookup Metadata Schema.json` · **Template**: `Schemas/Templates/LOOKUP METADATA TEMPLATE.yaml`
- **Location**: `Lookups/Global/`, `Lookups/Splunk/`, `Lookups/Sentinel/`

### Deprecated Object Types

> **CDM** (Cyber Detection Models) and **BDR** (Business Detection Requests) are deprecated and replaced by **Detection Objectives (DOM)**. Schemas, templates, and issue templates still exist for legacy support but should not be used for new objects.

## Repository Structure

```
.
├── Objects/
│   ├── Threat Vectors/        # TVM files (*.yaml)
│   ├── Detection Objectives/  # DOM files (*.yaml)
│   └── Detection Rules/       # MDR files (*.yaml)
├── Schemas/
│   ├── *.json                 # JSON Schema definitions for each object type
│   ├── Templates/
│   │   └── *.yaml             # YAML templates showing object structure
│   ├── Indexes/
│   │   └── *.json             # Object reference indexes (auto-managed by CoreTide)
│   └── Exports/
│       ├── Objects Table.csv  # Exported objects table
│       └── ATT&CK Navigator Layer.json
├── Lookups/
│   ├── Global/                # Cross-system lookup tables
│   ├── Splunk/                # Splunk-specific lookups
│   └── Sentinel/              # Sentinel-specific lookups
├── Configurations/
│   ├── *.toml                 # Core configuration (deployment, documentation, schema, lookups, visibility)
│   └── systems/
│       └── *.toml             # Detection system configs (splunk, sentinel, crowdstrike, etc.)
└── .vscode/
    ├── settings.json          # YAML/TOML schema mappings for validation
    ├── extensions.json        # Recommended VS Code extensions
    └── Model Templates.code-snippets  # Snippets for quick object scaffolding
```

### File Naming Conventions
- **TVM**: Object files are YAML in `Objects/Threat Vectors/`, named descriptively (e.g. `T000001 - Credential Dumping via LSASS.yaml`)
- **DOM**: YAML in `Objects/Detection Objectives/`, named descriptively
- **MDR**: YAML in `Objects/Detection Rules/`, named descriptively
- **Lookups**: CSV files in the appropriate `Lookups/` subdirectory, accompanied by a YAML metadata file

## Workflow Guidelines

### When Creating Objects from Intelligence Reports

1. **Analyse the Intelligence (if presented)**
   - Review the provided intelligence thoroughly
   - Identify distinct TTPs and detection opportunities
   - Map relationships between threats and detection capabilities
   - **Avoid inferring from intuition or knowledge that is not initially present in the intelligence**
     > When there is not enough data from the intelligence presented, you are allowed to propose alternatives but **STOP** from proceeding before the user accepts the plan.
   - **STOP** and ask the user for precision, if you are unsure of which type of objects you are supposed to create.

2. **Plan the Object Hierarchy**
   > Important: The following is the default hierarchy. Based on user prompting, you may need
   > to focus on specific object types at a time, or generate the entire structure in one go.
   - Determine which Threat Vectors (TVMs) need to be created and chaining opportunities
   - Identify corresponding Detection Objectives (DOMs)
   - Plan Detection Rules (MDRs) if applicable

3. **Check for Existing Content**
   - Search the repository for related objects
   - Identify if updates to existing files are more appropriate than creating new ones
   - Structure and plan work to decide updates vs. creation
   - If updating existing files, ensure you preserve coherency, existing UUID, and other relations (unless you explicitly want to modify them)
   - If you conclude that new TVM(s) are warranted, investigate existing TVM content for chaining opportunities

4. **Consult Templates and Schemas**
   - Load the appropriate template from `Schemas/Templates/*.yaml`
   - Reference the relevant JSON Schema `Schemas/*.json` for detailed field requirements
   - Understand required vs. optional fields
   - Note any enumerated values or validation rules

5. **Generate UUIDs**
   - Use system tools to generate unique UUIDs (e.g., `uuidgen` on Unix/Linux/macOS, `[guid]::NewGuid()` on PowerShell)
   - If unable to generate UUIDs, clearly instruct the user to add them manually
   - **NEVER reuse UUIDs from existing objects**

6. **Create Objects**
   - Start with Threat Vectors (foundational)
   - Then create Detection Objectives (link to TVMs)
   - Finally create Detection Rules (link to DOMs)
   - Place files in the correct folders per project structure

7. **Validate**
   - If using VS Code, open the file — the YAML extension auto-validates against the mapped JSON schema (see `.vscode/settings.json` for mappings)
   - Verify all required fields are populated
   - Confirm UUIDs are unique across the repository
   - Review relationships between objects (cross-reference UUIDs)
   - There are no local CLI validation commands; full validation runs in CI/CD via CoreTide

8. **Present and Confirm**
   - Show a report of the generated content to the user
   - Explain the relationships and structure

9. **Commits and Merge/Pull Request**
   - If the user asks you to automate commit and merge, do it
   - Discover your environment to see if you have the tools to perform PRs creation/update, else report to the user

## Constraints

### Data Handling
- ✅ **DO**: Use provided intelligence and repository content as your primary source
- ❌ **DON'T**: Rely on pre-training data for threat intelligence or TTPs
- ✅ **DO**: Search and reference existing objects when relevant
- ❌ **DON'T**: Hallucinate or invent threat intelligence

### Schema Compliance
- ✅ **DO**: Always validate against JSON schemas
- ✅ **DO**: Use templates to understand structure before consulting full schemas
- ✅ **DO**: Search schemas for specific field requirements (schemas may be too large to load entirely)
- ❌ **DON'T**: Generate objects without understanding the schema requirements

### UUID Management
- ✅ **DO**: Generate unique UUIDs using system tools
- ✅ **DO**: Clearly indicate when users must add UUIDs manually
- ❌ **DON'T**: Reuse existing UUIDs
- ❌ **DON'T**: Make up or hardcode UUIDs

### Authoring
- ✅ **DO**: Use British English by default consistently
- ✅ **DO**: Focus on one object type creation (for example, if presented intelligence, perform a review of TVMs) unless the user explicitly asks for a more complex operation (multiple object type creation)
- ❌ **DON'T**: For TVM, when writing the `terrain` section, create extremely long sections — keep it coherent and well documented but relatively concise
- ❌ **DON'T**: Overassume when you need to generate a detection rule query — better to generate the whole object but not the query and propose it to the user as inspiration (if high confidence, add it; else put pseudocode as a comment, and mention your proposal to the user)

## Communication Protocol

### When Starting a Task
1. Acknowledge the task
2. Outline your understanding and approach
3. Ask clarifying questions if needed
4. Create a plan and to-dos

### During Task Execution
1. Explain what you're doing at each major step
2. Update to-dos and manage context memory
3. Highlight any decisions or assumptions made

### When Completing a Task
1. Summarize what was created/modified
2. Explain the relationships between objects
3. Provide any necessary next steps or manual actions required
4. Confirm all files are in the correct locations
5. If you focused on one object type, and already identified opportunities for additional object type creation (for example, you generate TVMs, and can already start working on DOM creation), propose explicitly to the user a follow-up operation with what you can assess as needed.

## Quick Reference

| Task | Steps | Files Involved |
|---|---|---|
| Create TVM from report | Analyse → Plan → Generate → Validate | `Objects/Threat Vectors/*.yaml` |
| Create DOM for TVM | Identify TVM → Define signals → Generate | `Objects/Detection Objectives/*.yaml` |
| Create MDR for DOM | Identify DOM → Choose system → Generate rule | `Objects/Detection Rules/*.yaml` |
| Update existing object | Find object → Show changes → Update | Varies |
| Validate objects | Open in VS Code or check CI/CD pipeline | `Schemas/*.json`, `.vscode/settings.json` |

| Object Type | Schema | Template | Location |
|---|---|---|---|
| TVM | `Schemas/TVM Schema.json` | `Schemas/Templates/TVM TEMPLATE.yaml` | `Objects/Threat Vectors/` |
| DOM | `Schemas/Detection Objective.schema.json` | `Schemas/Templates/Detection Objective.template.yaml` | `Objects/Detection Objectives/` |
| MDR | `Schemas/MDR Schema.json` | `Schemas/Templates/MDR TEMPLATE.yaml` | `Objects/Detection Rules/` |
| Lookup | `Schemas/Lookup Metadata Schema.json` | `Schemas/Templates/LOOKUP METADATA TEMPLATE.yaml` | `Lookups/` |

## Error Handling

- **Schema Validation Errors**: Discover error in IDE validation errors, then compare against Schema and patch the issue before completing the task.
- **Missing UUIDs**: Clearly indicate where UUIDs are needed and how to generate them
- **Conflicting Information**: Ask the user for clarification rather than making assumptions
- **Uncertain Relationships**: Present options to the user rather than guessing

---

**Remember**: Your primary goal is to support the creation of a well-structured, high-quality OpenTide repository (knowledge graph of Detection Engineering objects), making intelligence and modelling quicker, faster and more correct.
