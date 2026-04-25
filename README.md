# yesglot.toml Specification

Yesglot is configured via a `yesglot.toml` file placed in the root of your Git repository. This file tells Yesglot which technology you use, which language is your source, and which languages to translate into.

---

## Example

```toml
[yesglot]
version = 1

[project]
id = "proj_abc123"
technology = "react-i18next"
custom_prompt = "This is a SaaS product for developers. Use informal tone."

[source_language]
name = "en"
path = "src/locales/en/translation.json"

[[target_language]]
name = "tr"
path = "src/locales/tr/translation.json"

[[target_language]]
name = "fr"
path = "src/locales/fr/translation.json"
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
| `custom_prompt` | No | Extra instructions for the translation agent (tone, style, domain context). |

### `[source_language]`

The language your strings are written in.

| Field | Required | Description |
|---|---|---|
| `name` | Yes | BCP 47 language tag (e.g. `en`, `fr`, `tr`). |
| `path` | Yes | Path to the source translation file, relative to the repo root. |

### `[[target_language]]`

Repeat this block for each language you want to translate into.

| Field | Required | Description |
|---|---|---|
| `name` | Yes | BCP 47 language tag (e.g. `tr`, `fr`, `de`). |
| `path` | Yes | Path to the target translation file, relative to the repo root. |

---

## Supported Technologies

| Value | Framework |
|---|---|
| `react-i18next` | React with i18next |
| `django` | Django i18n (.po files) |
| `rails` | Ruby on Rails i18n (.yml files) |
| `laravel` | Laravel localization |
| `vue-i18n` | Vue with vue-i18n |
| `angular` | Angular with @angular/localize |

---

## How it works

1. Yesglot watches your repository for changes via GitHub.
2. When a commit touches your `source_language.path` or the `yesglot.toml` file itself, a Translation Job is triggered automatically.
3. Yesglot translates the new or changed strings and opens a Pull Request with the updated target language files.
4. Merge the PR — done.

---

## Notes

- **Never change your `project.id`** — it is how Yesglot identifies your project regardless of filename or repo restructures.
- Language tags must be valid [BCP 47](https://www.rfc-editor.org/rfc/rfc5646) codes (e.g. `en`, `tr`, `zh-Hans`).
- All paths are relative to the repository root.
- `custom_prompt` is optional but recommended — the more context you give, the more accurate the translations.