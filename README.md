# Moztopia Localization

This repository is the central source for translation catalogs used by Moztopia applications.

Each application owns a directory containing one JSON catalog per supported locale. Applications can retrieve catalogs from a branch, release tag, or commit while retaining bundled assets as an offline fallback.

## Repository structure

```text
localization/
├── example/
│   ├── de.json
│   ├── en.json
│   ├── es.json
│   ├── fr.json
│   ├── my.json
│   ├── ru.json
│   ├── th.json
│   └── zh.json
├── LICENSE
└── README.md
```

Each top-level application directory provides an independent set of translation catalogs:

```text
<application>/<locale>.json
```

For example:

```text
example/en.json
example/th.json
```

## Catalog format

Every catalog must contain a JSON object. Nested objects organize related translation keys:

```json
{
  "example": {
    "welcome": "Welcome to the localization example",
    "language": "Language",
    "preview": "This text changes when the selected language changes."
  }
}
```

All locale catalogs for an application should have the same key structure as its English catalog.

English is the canonical fallback language and source of truth for required translation keys.

## Supported example locales

| Locale | Language |
| --- | --- |
| `en` | English |
| `th` | Thai |
| `es` | Spanish |
| `zh` | Chinese |
| `ru` | Russian |
| `my` | Burmese |
| `fr` | French |
| `de` | German |

Locale filenames are lowercase and use the locale code expected by the consuming application.

## Raw catalog URLs

Applications retrieve catalogs through GitHub’s raw-content endpoint:

```text
https://raw.githubusercontent.com/moztopia/localization/<reference>/<application>/<locale>.json
```

Examples:

```text
https://raw.githubusercontent.com/moztopia/localization/main/example/en.json
https://raw.githubusercontent.com/moztopia/localization/main/example/th.json
```

`<reference>` may be:

- `main` for the current catalog release;
- a version tag for a stable historical release; or
- a commit SHA for an exact immutable revision.

## Flackage configuration

A Flutter application using Flackage can describe its local and remote sources in `assets/localization.config.yaml`:

```yaml
fallbackSource: language

localSource: assets
localPath: translations

remoteSource: git
remoteRepository: https://github.com/moztopia/localization
remotePath: example
remoteReference: main

languages:
  - locale: en
    name: English
    label: EN
  - locale: th
    name: ไทย
    label: TH
  - locale: es
    name: Español
    label: ES
  - locale: zh
    name: 中文
    label: ZH
  - locale: ru
    name: Русский
    label: RU
  - locale: my
    name: မြန်မာ
    label: MY
  - locale: fr
    name: Français
    label: FR
  - locale: de
    name: Deutsch
    label: DE
```

The corresponding bundled fallback catalogs are stored in the application:

```text
assets/translations/en.json
assets/translations/th.json
```

The intended loading hierarchy is:

1. Require a valid bundled English catalog.
2. Load the bundled catalog for the selected locale when available.
3. Retrieve the selected locale from this repository.
4. Merge remote values over bundled values.
5. Fall back to bundled English or the translation key according to application configuration.

Remote-loading failures must not prevent an application from using valid bundled catalogs.

## Adding an application

Create a directory named for the consuming application:

```text
my_application/
├── en.json
├── th.json
└── ...
```

Then configure the application with:

```yaml
remotePath: my_application
```

Application directories must not depend on catalogs from other application directories.

## Adding a translation key

1. Add the new key and English text to `<application>/en.json`.
2. Add the same key path to every supported locale catalog.
3. Preserve the existing JSON hierarchy.
4. Validate every modified JSON file.
5. Confirm that all locale catalogs contain the same key paths as English.
6. Submit the changes together in one pull request.

Example:

```json
{
  "navigation": {
    "home": "Home",
    "settings": "Settings"
  }
}
```

Do not rename or remove an existing key without coordinating the corresponding application change.

## Adding a language

1. Add `<locale>.json` under the appropriate application directory.
2. Copy the complete key structure from `en.json`.
3. Translate every value without changing its key.
4. Add the locale to the application’s localization configuration.
5. Confirm that the application includes any required bundled fallback catalog.

## Validation

Validate an individual catalog with `jq`:

```shell
jq empty example/en.json
```

Validate all JSON catalogs:

```shell
find . -name '*.json' -print0 | xargs -0 -n1 jq empty
```

Compare key paths against English:

```shell
jq -r '[paths(scalars) | map(tostring) | join(".")] | sort | .[]' \
  example/en.json > /tmp/en.keys

jq -r '[paths(scalars) | map(tostring) | join(".")] | sort | .[]' \
  example/th.json > /tmp/th.keys

diff -u /tmp/en.keys /tmp/th.keys
```

A successful key comparison produces no output.

## Translation rules

- Preserve JSON key names exactly.
- Translate values, not keys.
- Preserve placeholders such as `{name}`, `{count}`, and positional arguments.
- Preserve plural and gender structures expected by the application.
- Keep technical product names unchanged unless an approved localized name exists.
- Use natural language rather than word-for-word translation.
- Save files as UTF-8.
- Do not add comments because standard JSON does not support them.
- Do not store secrets, credentials, tokens, personal information, or private operational data in catalogs.

## Branches and releases

`main` represents the most current released catalogs.

Applications that should follow the current release may use:

```yaml
remoteReference: main
```

Applications requiring repeatable, immutable resolution should use a release tag:

```yaml
remoteReference: v0.0.1
```

Catalog changes should be reviewed and validated before merging into `main`. Use version tags when applications need to remain on a known catalog revision.

## Security

This is a public repository. Treat every committed file as publicly accessible.

Future loader versions may support catalog signatures, hashes, authenticated repositories, and persistent caching. Until then, consuming applications should always retain a valid bundled English fallback catalog.

## License

See [LICENSE](LICENSE).
