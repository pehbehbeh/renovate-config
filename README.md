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
