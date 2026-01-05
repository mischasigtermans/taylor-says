# Changelog

All notable changes to this project will be documented in this file.

## [1.2.0] - 2026-01-05

### Added
- Knowledge system with categorized Laravel best practices (`context/taylor/knowledge/`)
- Auto-loading of relevant knowledge based on file patterns being reviewed
- `@taylor cocacola` command for comprehensive reviews with all knowledge loaded
- 9 knowledge categories: eloquent, collections, controllers, validation, routing, authorization, blade, events, testing

### Changed
- Taylor now automatically loads relevant knowledge when reviewing code (no user action required)
- Curated 74 tips from research, excluded 4 that contradict Laravel philosophy

## [1.1.0] - 2026-01-05

### Added
- Inline Taylor quotes and signature phrases in agent prompt
- Laravel Boost MCP tool documentation (search-docs, application-info, database-schema)

### Changed
- Enhanced `agents/taylor.md` with authentic persona traits and voice
- Expanded Taylor-ism checklist from basic to 14 specific anti-patterns

## [1.0.0] - 2026-01-04

### Added
- Initial release
- Taylor Otwell code review agent (`agents/taylor.md`)
- Claude Code plugin configuration
- Basic Taylor philosophy and review standards
- 7 RESTful controller pattern enforcement
- Anti-repository pattern stance
- Developer happiness mission