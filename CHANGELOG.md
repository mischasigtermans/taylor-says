# Changelog

All notable changes to this project will be documented in this file.

## [1.3.1] - 2026-01-05

### Fixed
- `personality.md` now explicitly listed in "Before You Begin" section (was only in Context Files, agent wasn't loading it)

## [1.3.0] - 2026-01-05

### Added
- Personality layer (`context/taylor/personality.md`) with sourced Taylor-isms:
  - Pop culture (Kenny vs T1000, Apple philosophy, dresser's backside)
  - Self-deprecating phrases ("I'm a pretty average programmer")
  - Closing energy ("Ship it", "Good grief", "SURFS UP")
  - The Vibe ("Soon we're dead so for now I vibe")
- 3 new anti-patterns (#20-22): Testability Theater section

### Changed
- Taylor-ism checklist expanded from 19 to 22 items

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