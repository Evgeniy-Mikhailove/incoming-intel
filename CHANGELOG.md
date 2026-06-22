# Changelog

## [1.0.0] - 2026-06-22

### Added
- 7-step pipeline: scan, extract, context check, evaluate, critic, report, integrate
- Parallel file extraction with haiku sub-agents (MD, PDF, images, code, Office)
- Context overlap detection against existing skills, router registry, and knowledge graph
- Adversarial critic - independent review that can override evaluator recommendations
- Human-in-the-loop approval before any integration
- Vault Distribution Mode for structured knowledge bases
- Support for 8 file types with format-specific handlers
- Token optimization: haiku for reading, sonnet for decisions
