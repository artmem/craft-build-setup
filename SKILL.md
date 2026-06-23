---
name: craft-build-setup
description: Bootstrap a new Craft CMS 5 project on DDEV through the deterministic setup phase — DDEV init, project scaffold, plugin install, env config, directory structure, Vite + CUBE CSS scaffold, and CLAUDE.md scoping. Use this whenever starting a new Craft site build, spinning up a fresh Craft install, setting up a Craft project locally, or when the user says "new Craft build", "start a Craft project", "set up Craft", or names a new site to build on Craft. Trigger before any field/schema or template work — this is the phase that comes first.
---

# Craft Build — Setup Phase

Walks a new Craft CMS 5 build from empty directory to a running DDEV install
with the house stack scaffolded, ready for schema and template work.

This skill owns the **setup phase only**. Field/structure creation and template
building are separate phases handled by other tools (happycog craft-skill for
validated schema mutations; Stimmt craft-mcp for schema-aware templating). Stop
when the install is running and the scaffold is in place.

## Before starting — confirm with the user

Ask for these unless already stated. Make reasonable calls on the rest.

1. **Project name / slug** — drives the directory name and DDEV project name.
2. **Where it lives** — default `~/dev/web/<slug>/`.
3. **Plugin set** — confirm the default block below or override per project.
4. **Anything non-standard** — multi-site? Commerce? Headless/GraphQL-first?
   If none mentioned, assume single-site, Twig-rendered, no Commerce.

Do not proceed past preflight without a project slug.

## The house stack (assumed unless overridden)

- Craft CMS 5, PHP 8.2+, DDEV
- Twig templates, minimal JS
- Vite for assets
- CUBE CSS + Every Layout primitives + Utopia fluid scale
- Semantic color tokens (OKLCH), WCAG 2.2 AA contrast
- Lives under `~/dev/web/<slug>/` with directory-scoped CLAUDE.md

## Default plugin set

Base set goes on every build. Client set adds on top when the project is a client
site (not a personal/imprint build). Confirm at the start; override per project.

```
# BASE — every build
nystudio107/craft-vite         vite          # asset pipeline
nystudio107/craft-typogrify    typogrify     # smart typography
craftcms/ckeditor              ckeditor      # rich text

# CLIENT sites — add on top of base
solspace/craft-freeform        freeform      # forms
nystudio107/craft-seomatic     seomatic      # SEO

# OPTIONAL — offer at setup, install only on confirmation
wrav/oembed                    oembed        # embeds; only if site needs video
putyourlightson/craft-sprig    sprig         # reactive components
putyourlightson/craft-blitz    blitz         # static cache; usually pre-launch, not setup
craftcms/feed-me               feed-me       # imports; only if a feed exists
```

Ask once whether this is a client build or a personal/imprint one, then install
the matching base + client set. For the optional tier, name them and install only
the ones the user confirms — don't add them silently. Blitz in particular is
usually a pre-launch decision, not a setup one; flag that if offered.

## Sequence

Run these in order. After each step that can fail, check the result before moving on.

### 1. Preflight

- Confirm `ddev` is installed (`ddev --version`). If missing, stop and tell the
  user to install DDEV — do not try to work around it.
- Confirm the target directory doesn't already exist. If it does, stop and ask.
- Confirm PHP/Composer availability is handled by DDEV (it is — don't require
  host PHP).

### 2. Scaffold directory + DDEV

```bash
mkdir -p ~/dev/web/<slug> && cd ~/dev/web/<slug>
ddev config --project-type=craftcms --docroot=web --create-docroot
ddev start
```

`--project-type=craftcms` gives the right PHP, web server, and DB defaults plus
Craft's post-start hooks.

### 3. Create the Craft project

```bash
ddev composer create -y craftcms/craft
ddev craft install/craft \
  --interactive=0 \
  --username=admin \
  --email=<user-email> \
  --password=<temp-or-prompt> \
  --site-name="<Project Name>" \
  --site-url='$PRIMARY_SITE_URL'
```

Prompt the user for admin email; never invent credentials. Use a clearly-temporary
password and tell them to change it.

### 4. Install plugins

For each plugin in the confirmed set:

```bash
ddev composer require <package>
ddev craft plugin/install <handle>
```

Install one at a time so a single failure doesn't abort the batch. Report which
installed cleanly.

### 5. Env + config

- Set sensible `.env` defaults (`CRAFT_ENVIRONMENT=dev`, dev mode on, devMode
  allowed admin changes on for local).
- Confirm `config/general.php` has `aliases` for `@web` / asset base sane for DDEV.
- For Imager-X / remote asset sourcing in local dev, note the config pattern but
  don't hardwire production buckets.

### 6. Front-end scaffold

Read `references/stack-scaffold.md` and lay down:

- `package.json` + Vite config wired to Craft's `vite` plugin or a `web/dist`
  manifest pattern
- CSS architecture: `css/` with CUBE layers (composition / utility / block /
  exception), Utopia fluid type+space custom properties, OKLCH semantic token
  layer, a reset
- Base Twig: `_layouts/`, Every Layout primitives as Twig includes/macros
  (`stack`, `cluster`, `sidebar`, `switcher`, etc.), `_components/`
- Do **not** pull in Tailwind or any utility-class framework.

Only read the reference file when you reach this step.

### 7. CLAUDE.md scoping

Drop a project-level `CLAUDE.md` at `~/dev/web/<slug>/CLAUDE.md` that:

- Inherits the global stack conventions (don't restate them — point to them)
- Notes anything project-specific: plugin set, multi-site, content-model quirks
- Records the DDEV project name and primary URL

### 8. Handoff

Stop here. Report:

- DDEV URL (`ddev describe`), admin login, temp password reminder
- Which plugins installed
- That schema/field creation is the next phase (happycog craft-skill) and
  templating after that (Stimmt craft-mcp)

Do not start creating sections or fields. That's the next phase, deliberately
kept separate.

## What this skill does NOT do

- Schema: sections, fields, entry types, field layouts → next phase
- Template logic beyond the empty scaffold → templating phase
- Production env, deploy, hosting → out of scope
- Editing `config/project/*.yaml` by hand → never; that's what the schema tools
  exist to avoid
