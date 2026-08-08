# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) config presets, mostly aimed at Elixir/Phoenix projects.

## Usage

Extend the default preset:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>pehbehbeh/renovate-config"]
}
```

Any other preset is referenced by its path, without the `.json` extension:

```json
{
  "extends": [
    "github>pehbehbeh/renovate-config",
    "github>pehbehbeh/renovate-config//customManagers/hexTailwind",
    "github>pehbehbeh/renovate-config//security/minimumReleaseAgeHex"
  ]
}
```

To pin a preset to a specific revision, append `#<tag-or-sha>`.

## Presets

### `default`

The base preset. Extends [`config:best-practices`](https://docs.renovatebot.com/presets-config/#configbest-practices), disables rate limiting, assigns `pehbehbeh` as reviewer, and labels PRs with `dependencies` plus the dependency's categories.

### `customManagers/`

Elixir projects install some frontend tooling through Mix wrappers that pin the version in `config.exs` instead of in a lockfile. These presets add regex managers so those versions get updated too. Each one matches `config.exs` in any directory and resolves against the `npm` datasource.

| Preset | Updates |
| --- | --- |
| `customManagers/hexBun` | `config :bun, version: "…"` |
| `customManagers/hexEsbuild` | `config :esbuild, version: "…"` |
| `customManagers/hexTailwind` | `config :tailwind, version: "…"` |

### `group/`

| Preset | Description |
| --- | --- |
| `group/portainer` | Groups all `portainer/*` images into a single PR. |

### `policy/`

| Preset | Description |
| --- | --- |
| `policy/portainerLts` | Only raises updates for Portainer LTS releases, for `portainer/portainer-ce`, `portainer/portainer-ee` and `portainer/agent`. |

Portainer ships an LTS release every six months and an STS release roughly monthly ([lifecycle policy](https://docs.portainer.io/start/lifecycle)). Renovate has no way to tell the two apart: the images carry `lts` and `sts` Docker tags, but the `docker` datasource does not populate the tag map that `followTag` reads, so LTS status cannot be resolved to a version number. The preset therefore hardcodes the LTS series in `allowedVersions` and **needs a manual update whenever a new LTS lands**:

1. Check the [lifecycle policy](https://docs.portainer.io/start/lifecycle) for the new LTS version.
2. Add it to the `allowedVersions` regex in `policy/portainerLts.json` as one more `|2.XX` alternative.

Currently listed: `2.21`, `2.27`, `2.33`, `2.39`, and `2.45` (announced for August 2026, added ahead of release). Next expected around February 2027. Only the series are listed, not individual releases — patch updates inside an allowed series are picked up on their own, so the list only changes twice a year.

`allowedVersions` is a single string, so the list has to be either a regex or a version range. The range form (`2.21.x || 2.27.x || …`) reads more nicely and does work — Renovate falls back to npm semver syntax when the range is invalid for the `docker` versioning in use. It is not used here because semver reads the suffix on a tag like `2.39.5-alpine` as a prerelease, and prereleases never satisfy a plain range, so every suffixed tag would be filtered out and those deployments would silently stop receiving updates. The regex is matched against the raw tag and handles them correctly.

Two further caveats:

- The preset deliberately does not match every `portainer/*` image. Images like `portainer/pause` do not follow this version scheme, and applying the allowlist to them would stop their updates entirely.
- If a deployment currently sits on an STS version, Renovate will raise nothing until the next LTS overtakes it, since every allowed version is lower than the current one. Move it to an LTS tag when adopting the preset.

### `security/`

| Preset | Description |
| --- | --- |
| `security/minimumReleaseAgeHex` | Waits three days before raising an update for a Hex package, giving malware scanners a chance to catch a malicious release. Mirrors the upstream [`security:minimumReleaseAgeNpm`](https://docs.renovatebot.com/presets-security/#securityminimumreleaseagenpm) preset, which Renovate does not ship for Hex. |

Only the `hex` datasource is covered. Dependencies pulled in via git (`git-tags`, `github-tags`, `gitlab-tags`) and the `customManagers/hex*` presets above (which use the `npm` datasource) are not — pair with `security:minimumReleaseAgeNpm` if you use those.

### `workarounds/`

| Preset | Description |
| --- | --- |
| `workarounds/mixGitVersioning` | Forces `semver` versioning for git-sourced `mix` dependencies, which Renovate otherwise fails to compare correctly. |
| `workarounds/umamiVersioning` | Teaches Renovate to parse the `<compatibility>-v<semver>` tag format used by the Umami Docker image. |

## Validating changes

```sh
bunx --package renovate renovate-config-validator <path-to-preset>.json
```

## License

MIT — see [LICENSE](LICENSE).
