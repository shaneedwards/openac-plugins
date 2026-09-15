# openac-plugins

The plugin list the [OpenAC](https://github.com/eriknihlen/OpenAC) launcher reads to populate its
Plugins → Discover tab and to filter blocked plugins out of every character's allow-list at
launch. Proof of concept, hosted on Shane's account for now; the launcher also accepts a
`--plugin-list-uri` override for testing.

## Format

`plugins.json`:

```json
{
  "schemaVersion": 1,
  "plugins": [
    { "id": "edwards.hello", "name": "Hello", "author": "Shane Edwards",
      "description": "Says hello.", "repo": "shaneedwards/openac-plugin-hello" }
  ],
  "blocked": [
    { "id": "someone.bad", "versions": ["*"], "reason": "Sends chat spam." }
  ]
}
```

- `schemaVersion` — currently `1`. Unknown top-level fields are rejected, same strictness as the
  launcher's other manifests.
- `plugins` — one entry per listed plugin. `repo` is `owner/name` on GitHub; the launcher reads
  that repository's releases directly (no GitHub API calls).
- `blocked` — id/version pairs the launcher refuses to install or update to. A blocked plugin that
  is not installed is hidden from Discover; one that is installed shows "Blocked: <reason>" and is
  filtered out of every character's plugin list at launch, with a line in the session log.
  `versions: ["*"]` blocks every version of that id. This lives in the same file as `plugins`
  rather than a separate `blocked.json`: one file to fetch, one file to publish.

`plugins-blocked-test.json` in this repo is not part of the live list. It is the LP-12 runtime
test fixture: the same catalog with `edwards.hello` blocked at every version, reason `"test"`,
published as a throwaway list release during that runtime step and then replaced by a normal list
release again.

## Publishing

1. Update `plugins.json`.
2. Tag `v1` (bump the tag for later catalog changes; the launcher always reads
   `releases/latest/download/plugins.json`, not a specific tag).
3. Create a GitHub release on that tag, not a draft or a prerelease, and attach `plugins.json`
   itself as a release asset (same file name).
4. Make sure GitHub marks it *latest*.

The launcher fetches `https://github.com/shaneedwards/openac-plugins/releases/latest/download/plugins.json`.

Right after a new release, that link can still serve the previous list for about a minute. Wait a
minute, then press **Check now** in the launcher before judging whether the change arrived.
