# Documentation Refresh Design

**Status:** Approved
**Date:** 2026-01-06

## Problem Statement

Documentation has drifted significantly from the codebase after major refactoring:
- CLI split from monolithic `notebooklm_cli.py` into `cli/` package with 12 modules
- Service layer removed; API changed from `NotebookService(client)` to `client.notebooks.list()`
- Ghost methods documented (`add_pdf()`) that don't exist
- Stale file references, line numbers, and repository structure diagrams

## Target Audiences

1. **LLM agents** using CLI for automation
2. **Python developers** building applications with the library
3. **Human CLI users** running commands directly
4. **Contributors** extending or debugging the library
5. **End users** who use LLMs to interact with NotebookLM

## Design Decisions

### Cleanup (Deletions)

| File | Reason |
|------|--------|
| `docs/FILE_UPLOAD_IMPLEMENTATION.md` | Implementation complete, info stale |
| `docs/designs/architecture-review.md` | Refactoring done, was planning doc |
| `docs/scratch/2026-01-05-e2e-test-analysis.md` | Info in KnownIssues |
| `docs/scratch/2026-01-05-extraction-verification.md` | Temporary work |
| `docs/scratch/2026-01-05-test-fix-summary.md` | Temporary work |
| `GEMINI.md` | Merged into AGENTS.md |

### Consolidations

| From | To |
|------|-----|
| `docs/reference/internals/*.md` (5 files) | `docs/reference/internals/discovery.md` |
| `docs/API.md` + `docs/EXAMPLES.md` | `docs/python-api.md` |
| `docs/reference/KnownIssues.md` | `docs/troubleshooting.md` |
| `GEMINI.md` + `AGENTS.md` | `AGENTS.md` |

### New Files

| File | Purpose |
|------|---------|
| `docs/getting-started.md` | Install → login → first workflow |
| `docs/cli-reference.md` | CLI for humans + LLMs |
| `docs/contributing/architecture.md` | Code organization, layers |
| `docs/contributing/debugging.md` | Network capture, RPC tracing |
| `docs/contributing/testing.md` | Running tests, E2E auth |
| `docs/contributing/rpc-protocol.md` | Moved from reference/ |

### Final Structure

```
Root:
├── README.md              # Lean: pitch, install, quick start, links
├── CONTRIBUTING.md        # Human + agent rules, PR process
├── CLAUDE.md              # Claude Code behavioral hints
├── AGENTS.md              # Other LLMs (merged from GEMINI.md)
├── CHANGELOG.md           # Release history (unchanged)

docs/
├── getting-started.md     # Install → login → first workflow
├── cli-reference.md       # Quick reference table + intent-based + workflows
├── python-api.md          # Full API reference + examples
├── troubleshooting.md     # Errors, known issues, workarounds
├── contributing/
│   ├── architecture.md    # Code organization, layers
│   ├── debugging.md       # Network capture, RPC tracing
│   ├── testing.md         # Test running, E2E auth
│   └── rpc-protocol.md    # Deep dive (moved from reference/)
└── reference/
    └── internals/
        └── discovery.md   # Consolidated reverse-engineering notes
```

## Content Guidelines

### README.md (Lean)

- Project pitch (one paragraph)
- Installation commands
- Quick start (5 CLI commands showing core workflow)
- Links to detailed documentation
- License

### cli-reference.md (Dual-Format)

**Section 1: Quick Reference (for humans)**
```markdown
| Command | Description | Example |
|---------|-------------|---------|
| `source add <url>` | Add URL source | `source add "https://..."` |
```

Full options and flags per command.

**Section 2: By Intent (for LLMs + humans)**
```markdown
### "I want to make a podcast about my documents"
notebooklm generate audio "focus on the main themes"
notebooklm generate audio --format debate "compare viewpoints"
```

Natural language headers map user intent to commands.

**Section 3: Workflows**
```markdown
### Research → Podcast workflow
User: "Find articles about climate change and make a podcast"

1. notebooklm create "Climate Research"
2. notebooklm use <notebook_id>
3. notebooklm source add-research "climate change" --mode deep
4. notebooklm generate audio "summarize key findings"
5. notebooklm download audio ./podcast.mp3
```

Multi-step recipes for common tasks.

### python-api.md

- Quick start example (5-10 lines)
- Full API reference (every method, parameter, return type)
- Common patterns and recipes
- All enums documented

### CLAUDE.md

- Slim behavioral hints only
- Updated repository structure
- Fixed file references (no `notebooklm_cli.py`, show `cli/` package)
- Link to CONTRIBUTING.md for shared rules

### Contributing Docs

- **architecture.md**: Three-layer design, file organization, `_*.py` naming
- **debugging.md**: Network capture, Chrome DevTools, RPC tracing
- **testing.md**: pytest commands, E2E auth setup, fixtures
- **rpc-protocol.md**: Full protocol reference (moved from reference/)

## Migration Steps

1. Delete obsolete files
2. Create `docs/contributing/` directory
3. Consolidate internals → `discovery.md`
4. Write new `getting-started.md`
5. Write new `cli-reference.md` (dual format)
6. Merge and rewrite `python-api.md`
7. Merge KnownIssues → `troubleshooting.md`
8. Move `RpcProtocol.md` → `contributing/rpc-protocol.md`
9. Write contributor docs (architecture, debugging, testing)
10. Update `README.md` (slim down)
11. Update `CLAUDE.md` (fix all stale references)
12. Merge `GEMINI.md` into `AGENTS.md`, delete `GEMINI.md`
13. Update `CONTRIBUTING.md` with enhanced contributor guide
14. Update `docs/README.md` to reflect new structure
15. Delete old files (`docs/API.md`, `docs/EXAMPLES.md`, `docs/reference/KnownIssues.md`)
