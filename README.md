# craft-build-setup

A Claude Code skill that bootstraps a new Craft CMS 5 project on DDEV through the
setup phase — DDEV init, project scaffold, plugin install, env config, directory
structure, Vite + CUBE CSS scaffold, and CLAUDE.md scoping. It stops once the
install is running and the scaffold is in place.

Setup only. Field/schema creation and template building are deliberately separate
phases handled by other tools (happycog craft-skill for validated schema
mutations; Stimmt craft-mcp for schema-aware templating). This gets you to a
running install with the house stack ready.

## Install

Drop the folder into your Claude Code skills directory, structure intact:

```
~/.claude/skills/craft-build-setup/
├── SKILL.md
└── references/stack-scaffold.md
```

Project-level instead of global: `.claude/skills/` inside a repo, committed so the
whole team gets it.

## Use

Triggers on "new Craft build", "set up Craft", "start a Craft project", or naming
a site to build. It asks for a project slug and whether it's a client or personal
build, then runs the sequence. The references file loads only at the scaffold step.

## Stack

Craft 5, PHP 8.2+, DDEV, Twig, minimal JS, Vite, CUBE CSS + Every Layout + Utopia,
OKLCH semantic tokens, WCAG 2.2 AA. Lives under `~/dev/web/<slug>/`. No Tailwind.

## Plugins

Three tiers: base on every build, client adds on top, optional installed only on
confirmation. Edit the block in SKILL.md to change defaults.

| tier | plugins |
|------|---------|
| base | Vite, Typogrify, CKEditor |
| client | + Freeform, SEOMatic |
| optional | oEmbed, Sprig, Blitz, Feed Me |

## Untested

First run is a debug pass. Not yet validated against a live DDEV build:

- DDEV command sequence and `ddev composer create` flags
- The `install/craft` command's quoting of `$PRIMARY_SITE_URL`
- Package handles, especially `wrav/oembed` and the putyourlightson packages

