# SU_HDG Release Artifacts

This public repository is only for SU_HDG auto-update artifacts.

## Contents

- `SU_HDG.rbz`: installable SketchUp extension archive.
- `update-manifest.json`: public manifest read by the plugin updater.

## Do Not Commit Here

- No Ruby source files.
- No private development notes.
- No GitHub tokens, API keys, or secrets.

Source code belongs in the private repository:

```text
BLACK-BOT25/SRC_PLUGIN_HDG
```

## Manifest URL

Use this URL in SketchUp `Extensions > SU_HDG > Update Settings`:

```text
https://raw.githubusercontent.com/BLACK-BOT25/SRC_PLUGIN_HDG_RELEASE/refs/heads/main/update-manifest.json
```

## Release Update Steps

1. Build `dist\SU_HDG.rbz` from the private source repository.
2. Copy it here as `SU_HDG.rbz`.
3. Update `update-manifest.json` version and notes.
4. Commit and push only the RBZ and manifest changes.
