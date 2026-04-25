# yesglot.toml Specification

Yesglot is configured via a `yesglot.toml` file placed in the root of your Git repository. This file tells Yesglot which technology you use, which language is your source, and which languages to translate into.

---

## Table of Contents

- [Minimal Example](#minimal-example)
- [Advanced Example](#advanced-example)
- [Fields](#fields)
  - [[yesglot]](#yesglot)
  - [[project]](#project)
  - [[source_language]](#source_language)
  - [[[target_language]]](#target_language)
  - [[glossary.{lang}]](#glossarylang)
- [Supported Technologies](#supported-technologies)
- [How it works](#how-it-works)
- [Notes](#notes)

---

## Minimal Example

The simplest possible configuration — one source language, one target language:

```toml
[yesglot]
version = 1

[project]
id = "proj_abc123"
technology = "react-i18next"

[source_language]
name = "en"
path = "src/locales/en/translation.json"

[[target_language]]
name = "tr"
path = "src/locales/tr/translation.json"
```

---

## Advanced Example

Full configuration with multiple target languages, custom prompts, and glossaries:

```toml
[yesglot]
version = 1

[project]
id = "proj_abc123"
technology = "react-i18next"
tracked_branch = "main"
custom_prompt = "This is a SaaS product for developers. Use informal tone."

[source_language]
name = "en"
path = "src/locales/en/translation.json"

[[target_language]]
name = "tr"
path = "src/locales/tr/translation.json"
custom_prompt = "Use informal second person (sen, not siz)."

[[target_language]]
name = "fr"
path = "src/locales/fr/translation.json"
custom_prompt = "Use vous. This market is enterprise."

[[target_language]]
name = "de"
path = "src/locales/de/translation.json"

[glossary.tr]
"Like"    = "Beğen"
"Follow"  = "Takip et"
"Explore" = "Keşfet"
"Reel"    = "Reel"
"Story"   = "Story"

[glossary.fr]
"Like"    = "J'aime"
"Follow"  = "Suivre"
"Explore" = "Explorer"
"Reel"    = "Reel"
"Story"   = "Story"

[glossary.de]
"Like"    = "Gefällt mir"
"Follow"  = "Folgen"
"Explore" = "Entdecken"
"Reel"    = "Reel"
"Story"   = "Story"
```

In this example, the translation agent for Turkish receives:

```
"This is a SaaS product for developers. Use informal tone. Use informal second person (sen, not siz)."
```

---

## Fields

### `[yesglot]`

| Field | Required | Description |
|---|---|---|
| `version` | Yes | Spec version. Always `1` for now. |

### `[project]`

| Field | Required | Description |
|---|---|---|
| `id` | Yes | Your Yesglot project ID. Generated during project creation. **Never change this.** |
| `technology` | Yes | The i18n framework used. See supported values below. |
| `tracked_branch` | No | The branch Yesglot watches for changes. Defaults to your repository's default branch. |
| `custom_prompt` | No | Extra instructions applied to all target languages. |

### `[source_language]`

The language your strings are written in.

| Field | Required | Description |
|---|---|---|
| `name` | Yes | BCP 47 language tag (e.g. `en`, `fr`, `tr`). |
| `path` | Depends on technology | Path to the source translation file, relative to the repo root. See supported technologies below. |

### `[[target_language]]`

Repeat this block for each language you want to translate into.

| Field | Required | Description |
|---|---|---|
| `name` | Yes | BCP 47 language tag (e.g. `tr`, `fr`, `de`). |
| `path` | Yes | Path to the target translation file, relative to the repo root. |
| `custom_prompt` | No | Extra instructions specific to this language. Appended after `project.custom_prompt`. |

### `[glossary.{lang}]`

Optional. A key-value map of enforced translations for a specific target language. The language code must match a declared `[[target_language]]` name.

Use this to ensure specific terms are always translated consistently, or to protect terms that should never be translated by mapping them to themselves.

```toml
[glossary.tr]
"Like"  = "Beğen"   # enforced translation
"Reel"  = "Reel"    # protected — never translate
```

---

## Supported Technologies

| Value | Framework | `source_language.path` |
|---|---|---|
| `react-i18next` | React with i18next | Required |
| `django` | Django i18n (.po files) | Not supported — source strings live in the target language files as `msgid` |
| `rails` | Ruby on Rails i18n (.yml files) | Required |
| `laravel` | Laravel localization (.php files) | Required |
| `vue-i18n` | Vue with vue-i18n | Required |
| `angular` | Angular with @angular/localize | Not supported — source strings live in the target language files as translation units |

### Example — Django (no `source_language.path`)

```toml
[yesglot]
version = 1

[project]
id = "proj_abc123"
technology = "django"

[source_language]
name = "en"

[[target_language]]
name = "tr"
path = "locale/tr/LC_MESSAGES/django.po"

[[target_language]]
name = "fr"
path = "locale/fr/LC_MESSAGES/django.po"
```

---

## How it works

1. Yesglot watches your repository for changes via GitHub.
2. When a commit is pushed to your `tracked_branch` and touches your `source_language.path` or the `yesglot.toml` file itself, a Translation Job is triggered automatically.
3. Yesglot translates the new or changed strings and opens a Pull Request with the updated target language files.
4. Merge the PR — done.

---

## Notes

- **Never change your `project.id`** — it is how Yesglot identifies your project regardless of filename or repo restructures.
- Language tags must be valid [BCP 47](https://www.rfc-editor.org/rfc/rfc5646) codes (e.g. `en`, `tr`, `zh-Hans`).
- All paths are relative to the repository root and must not start with `/`.
- `tracked_branch` is optional — if omitted, Yesglot uses your repository's default branch.
- `target_language.custom_prompt` is appended after `project.custom_prompt` — both are sent to the agent together.
- Every key in `[glossary.{lang}]` must match a declared `[[target_language]]` name.