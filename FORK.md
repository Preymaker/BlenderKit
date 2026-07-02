# Preymaker BlenderKit Fork

This is Preymaker's internal fork of [BlenderKit](https://github.com/blenderkit/blenderkit),
maintained for central, read-only deployment across the studio.

Upstream base: **v3.19.2.260411**

---

## What's different

### API key via environment variable

Set `BLENDERKIT_API_KEY` to any valid BlenderKit API token before launching Blender
and the addon will use it automatically, without requiring a manual login through the UI.

```bash
export BLENDERKIT_API_KEY=your_token_here
blender
```

The environment variable takes priority over any key stored in the addon preferences or
in the persistent JSON preferences file. The key will also appear in the addon preferences
panel so the UI reflects the active key.

This is useful for render nodes or other non-interactive sessions where going through
the OAuth login flow is not practical.

The studio API key is set in the wiz package definition rather than here. If the key
ever needs rotating, update it there and redeploy the package.

---

### Managed install mode

Set `BLENDERKIT_MANAGED_INSTALL` to any non-empty value to put the addon into managed
install mode:

```bash
export BLENDERKIT_MANAGED_INSTALL=1
blender
```

When active:

- All automatic update checks are suppressed on startup
- The "Check now" and "Update now" buttons in addon preferences are disabled and show
  a warning message if clicked
- The updater preferences panel is replaced with a notice explaining that updates are
  managed centrally, along with the current version and a Copy Info button
- Startup popups are suppressed — both server-fetched announcements (e.g. promotional
  content from BlenderKit's servers) and the local random tips that appear on first launch

This mode exists because the updater would otherwise attempt to write files into the
addon's own directory, which is not possible when deployed to a read-only central
location. With `BLENDERKIT_MANAGED_INSTALL` unset the addon behaves exactly as upstream,
so this can be proposed as a PR without affecting normal users.

---

## Deployment

This fork is intended to be deployed to a shared, read-only location (e.g.
`/maker/sw/noarch/BlenderKit/BlenderKit-3.19.2.260411-1/blenderkit/`). Both env vars above should
be set in the studio environment for all Blender sessions.

The BlenderKit-Client binary is **not** included in the repository (upstream gitignores
`client/v*/`). It is compiled from the Go source in `client/` at package build time by
the `blenderkit-noarch` package builder template. See that repo for build details.

User data (downloaded assets, preferences, cache) is written to each user's
`blenderkit_data` directory as normal and is not affected by the read-only addon location.

---

## Versioning

Fork releases are tagged as `v{upstream-version}-{N}`, e.g. `v3.19.2.260411-1`.

- The upstream version portion matches the BlenderKit release this fork is based on
- `-N` increments for each fork revision against the same upstream base
- When rebasing onto a new upstream release, reset N to 1

Python packaging tools will normalise `-N` to `.postN` (e.g. `3.19.2.260411.post1`);
this is expected and the two forms refer to the same version.

---

## Updating from upstream

1. Check the [upstream releases](https://github.com/blenderkit/blenderkit/releases) for
   the new version
2. Merge or rebase onto the upstream tag
3. Verify the changes here (particularly `addon_updater_ops.py` and
   `persistent_preferences.py`) are not overwritten or conflicted away
4. Tag the result as `v{new-upstream-version}-1`
5. Rebuild and redeploy via the package builder
