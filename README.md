# tj-skills

Custom agent skills for Kilo (kilo.ai) — specialized expertise packs that extend agent capabilities across frameworks, languages, and domains.

## Skills

### Game Development

| Skill | Description |
|-------|-------------|
| [bevy](skills/bevy/) | Bevy game engine: ECS architecture, component-driven design, system ordering, UI, build strategies, and common pitfalls |
| [bevy-game-engine](skills/bevy-game-engine/) | Bevy fundamentals: ECS, rendering, input, asset management, states, events, and game project structure |

### Web Development

| Skill | Description |
|-------|-------------|
| [htmx](skills/htmx/) | HTMX development guidelines for dynamic web apps with minimal JavaScript |
| [htmx-expert](skills/htmx-expert/) | Comprehensive htmx: attributes, patterns, server responses, events, extensions, security, and debugging |
| [frontend-design](skills/frontend-design/) | Production-grade frontend interfaces with bold, distinctive aesthetics and high design quality |

### Flutter

| Skill | Description |
|-------|-------------|
| [flutter-apply-architecture-best-practices](skills/flutter-apply-architecture-best-practices/) | Layered Flutter architecture (UI, Logic, Data) for scalable project structure |
| [flutter-build-responsive-layout](skills/flutter-build-responsive-layout/) | Adaptive layouts with `LayoutBuilder`, `MediaQuery`, `Expanded`/`Flexible` |
| [flutter-fix-layout-issues](skills/flutter-fix-layout-issues/) | Fix overflows, unbounded constraints, and other Flutter layout errors |
| [flutter-setup-declarative-routing](skills/flutter-setup-declarative-routing/) | `MaterialApp.router` with `go_router` for deep linking and browser history |
| [flutter-setup-localization](skills/flutter-setup-localization/) | Initialize `flutter_localizations`, `intl`, and `l10n.yaml` for i18n support |
| [flutter-implement-json-serialization](skills/flutter-implement-json-serialization/) | Model classes with `fromJson`/`toJson` using `dart:convert` |
| [flutter-use-http-package](skills/flutter-use-http-package/) | REST API integration with the `http` package |
| [flutter-add-widget-preview](skills/flutter-add-widget-preview/) | Interactive widget previews via the `previews.dart` system |
| [flutter-add-widget-test](skills/flutter-add-widget-test/) | Component-level tests with `WidgetTester` for UI rendering and interactions |
| [flutter-add-integration-test](skills/flutter-add-integration-test/) | App-level integration tests with Flutter Driver and `integration_test` |

### Backend

| Skill | Description |
|-------|-------------|
| [fastapi-templates](skills/fastapi-templates/) | Production FastAPI projects with async patterns, DI, repositories, and error handling |
| [dbmodel](skills/dbmodel/) | DBModel ORM framework for SQLite: models, queries, joins, CRUD, and migrations |

### Meta

| Skill | Description |
|-------|-------------|
| [write-a-skill](skills/write-a-skill/) | Create new agent skills with proper structure, progressive disclosure, and bundled resources |

## Structure

Each skill follows the Kilo skill format:

```
skill-name/
├── SKILL.md           # Main instructions (required)
├── REFERENCE.md       # Detailed docs (optional)
├── references/        # Reference documentation (optional)
└── scripts/           # Utility scripts (optional)
```

The `SKILL.md` frontmatter defines when the skill is loaded:

```yaml
---
name: skill-name
description: What it does. Use when [specific triggers].
---
```

## Usage

Skills installed in this directory are available to Kilo agents. Place the directory in your Kilo config path or reference it from `kilo.json`.
