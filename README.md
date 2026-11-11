# OpenCode checkout entries

A GUI-only patch for OpenCode **2.2.6 and 2.2.7**: manually add permanent clones of the same repository as separate project entries.

- Separate Home, Settings, New Session, and organized-sidebar entries.
- Persistent checkout selection and independent closing.
- New sessions run in the selected checkout, not the canonical clone.
- Secondary checkouts can have independent client-local names.
- Checkout-local icon colors and uploaded images, including the primary checkout when sibling entries exist.
- Organized sidebar groups all visible servers, with their projects and sessions underneath. Switching sessions does not hide the other server's projects.
- No automatic cloning, worktree allocation, recycling, branch switching, or backend identity changes.

This is a small patch distribution repository, not a fork of the entire upstream source tree. The upstream project is [anomalyco/opencode](https://github.com/anomalyco/opencode).

## Apply

Requires Git and Bun **1.4.4**. Use a fresh clone; do not apply over an installed app or a checkout with unrelated changes.

```sh
git clone --branch v2.2.6 --depth 1 https://github.com/anomalyco/opencode.git opencode-patched
cd opencode-patched
git switch -c checkout-entries
git apply --check /path/to/opencode-checkout-patch/checkout-entries-v2.2.6.patch
git apply /path/to/opencode-checkout-patch/checkout-entries-v2.2.6.patch
bun install --frozen-lockfile
```

Base commit: `dc2ffcf8bf26f9e669b6aa22b95f50e12be15e06`.
The patch also applies to **2.2.7**, base commit `fedc6f208aa2bd54c1019227214568083f6380ad`. Substitute `v2.2.7` in the clone command; the patch filename is unchanged.

Icon preferences are local to this GUI, not synced through the repository's shared backend identity. Existing colors are inherited until you customize each checkout. Single-entry primary projects keep normal server-backed appearance settings. Commands and other repository-level settings remain shared.

## Try the browser GUI without replacing the installed app

```sh
cd packages/app
VITE_OPENCODE_SERVER_MODE=none bun run build
bun run serve -- --host 127.0.0.1 --port 4444
```

Open <http://127.0.0.1:4444>, connect to your existing OpenCode 2.2.6 server, and manually add each checkout with **Add project**. Browser preferences are separate from desktop preferences; sessions remain on the existing server. Do not start or replace the backend.

For a native Electron build, follow upstream's `packages/desktop/README.md`. This repository does not include a native installer. The patch does not alter or restart an installed application or service.

### Green macOS app icon

Before building the native app, overlay the green icon assets from this repository onto the upstream checkout:

```sh
cp -R /path/to/opencode-checkout-patch/macos-icons/. packages/desktop/icons/prod/
```

Build with `OPENCODE_CHANNEL=prod`. The assets cover the running Dock icon, the ICNS fallback, and the macOS 26 Icon Composer icon. They retain OpenCode's design with a green tint so the patched app is distinguishable from the official app. The bundle identity and existing preferences are unchanged.

See `CHECKOUT_ENTRIES.md` inside the applied patch for behavior, limitations, and verification commands.

## Verification

Multi-server sidebar follow-up on **2.2.7**:

- Each visible server keeps its own project order, expansion state, and session list.
- Regression covers identical project names, repository IDs, paths, and session IDs on two servers, correct session routing, and expansion persistence across reload.
- Production-build regression passed three concurrent repetitions without retries. App typecheck and focused lint passed; the four checkout/icon regressions still pass.

Icon-isolation follow-up on **2.2.7**:

- App typecheck and 10 registry/persistence tests: passed.
- Four production-build browser regressions: passed. These cover independent colors and uploaded icons across reload, plus existing single-project autosave, removal, and retry behavior.
- The extended clone regression failed on the previous patch because changing the secondary color changed the primary color.

Original **2.2.6** patch verification:

- Canonical `bun run check`: passed (upstream lint warnings remain).
- App unit suite: 2,583 passed.
- Registry/persistence suites repeated ten times: 100 passed.
- Production-build Home browser suite: 20 passed.
- Focused existing Settings/New Session checks: passed.
- Extended clone regression, including exact session-creation directory and independent rename across reload: passed.
- Standalone production browser build: passed.

The optional `typecheck:e2e` command reports existing fixture/type errors and missing Vite asset declarations; it is not clean in this upstream version.

## License

The upstream source and this patch are distributed under the MIT license; see `LICENSE`.
