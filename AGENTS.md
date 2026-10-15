## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

## neutral project instructions

Before project work, read `.ai/README.md` and follow its reading order: `.ai/PRINCIPLES.md`, `.ai/WORKFLOW.md`, `.ai/PROJECT.md`, `.ai/ARCHITECTURE.md`, `.ai/TENANCY.md`, and `.ai/SAFETY.md`. Then read the applicable `.ai/` area instruction: `.ai/instructions/api.md`, `.ai/instructions/frontend.md`, or `.ai/instructions/mobile.md`. The current source is the final authority; verified `.ai/` guidance takes precedence over historical provider-specific rules.

The canonical documentation map, copy roles, status definitions, and isolated-checkout limitations live in [.ai/README.md](./.ai/README.md). Use that map rather than assuming all files under API docs override frontend-specific documentation. Historical documents retain their original decisions; only verified current guidance governs new work.

Permanent technical memory follows [.ai/WORKFLOW.md](./.ai/WORKFLOW.md), section 5. Before implementation, read the canonical module documentation and relevant decisions, then verify the source. After a significant change is finished and validated, update the corresponding existing documentation with evidence, limitations and separate implementation/integration/deployment status. Apply the small-change exception and authorization limits in that procedure; do not duplicate its rules or create a parallel changelog.

When the user types `/graphify`, use the installed graphify skill or instructions before doing anything else.

Rules:
- For codebase questions, use Graphify when it adds value for discovering structure, relationships, dependencies, or impact. It is not required for trivial/local changes whose scope is already clear. For medium/high-risk changes or cross-product investigations, prefer `graphify query "<question>"` as the initial map when useful; use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. The source remains the final authority.
- Dirty graphify-out/ files are expected after hooks or incremental updates; dirty graph files are not a reason to skip graphify. Only skip graphify if the task is about stale or incorrect graph output, or the user explicitly says not to use it.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying graph-relevant code, run `graphify update .` when authorized and useful; do not impose it on trivial/local changes or when the user prohibits Graphify updates.
