# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-04-07

### Added
- Plugin scaffolding (package.json, plugin.json, README, LICENSE)
- File templates for ROADMAP.md, MILESTONE.md, TASK.md, and task contracts (standard + complex)
- `setup-roadmap` skill — initialize roadmap directory structure
- `build-roadmap` skill — conversational flow to transform brainstorm drafts into formal roadmap
- `update-roadmap` skill — structural changes (add/cancel milestones, add tasks, reorder phases)
- `update-task` skill — status changes with cascade updates across all index files
- Session-start hook for roadmap detection and skill availability reminders
