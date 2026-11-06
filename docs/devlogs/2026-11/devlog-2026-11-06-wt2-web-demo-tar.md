---
title: Web demo tar dependency correction
type: devlog
status: implemented
updated: 2026-11-06
mission: M1109
---

# Web demo tar dependency correction

The `web-demo` lockfile selected `tar@7.4.3` through
`@tailwindcss/postcss@4.1.11 -> @tailwindcss/oxide@4.1.11 -> tar@^7.4.3`.
[GHSA-23hp-3jrh-7fpw](https://github.com/advisories/GHSA-23hp-3jrh-7fpw)
affects versions through 7.5.18 and identifies 7.5.19 as patched. This is the
dependency reported by Dependabot alert 83.

## Source and change

The base is `ea006ab76192422d4b338054d5bd51e9b6ae8b02` in `HUA-Labs/hua-demo`.
Its own manifests, lockfiles, installation instructions and contribution workflow
describe a standalone repository. No projection manifest or CI owner is present;
the inspected HUA-platform public projection targets `HUA-Labs/packages` and does
not include this demo. This correction therefore uses the demo source directly.
The root README prefers `hua-demo-clean`, while the `web-demo` README still
documents this installation path; neither establishes a current deployed service.

From `web-demo`, the normal resolver command was:

```sh
npm update tar --package-lock-only --ignore-scripts --no-audit --no-fund
```

npm 11.17.0 selected `tar@7.5.22`, within the existing `^7.4.3` range and beyond
the advisory's patched floor. It also selected required `minizlib@3.1.0`, removed
the now-unused `mkdirp` entry, and recorded six bundled WASM dependency metadata
entries under the unchanged oxide package. These are resolver output, not a new
override or direct dependency. Tar's published license metadata changed from ISC
to BlueOak-1.0.0. All existing package entries other than tar/minizlib/mkdirp remain
exact, including PostCSS, oxide, Tailwind, React and Vite; `package.json` is unchanged.

## Focused consumption

Environment: macOS arm64, Node 24.22.1, npm 11.17.0. All execution used a disposable
copy of the corrected `web-demo`; no deployed checkout or service was changed.

- `npm ci --foreground-scripts --no-audit --no-fund` succeeded, installing 215
  packages and running the real oxide/esbuild postinstall scripts. The installed
  lock remained byte-exact. npm emitted an allow-scripts warning, retained in the log.
- `npm explain tar` and `npm ls @tailwindcss/postcss @tailwindcss/oxide tar minizlib
  --all` confirmed the unchanged parent chain and the selected patched tar.
- In a separate disposable package directory without the optional platform binary,
  the unchanged oxide `scripts/install.js` downloaded the official
  `oxide-darwin-arm64-4.1.11.tgz` and extracted it through the corrected tar.
  The resulting 2,844,320-byte native binary loaded and its real `Scanner` found
  `font-bold` and `p-4` in ordinary HTML input. No network or executor interception
  was used. This confirms extraction beyond the installer's success exit code.
- The public `postcss` plus `@tailwindcss/postcss` entry generated a `font-bold`
  utility from local CSS (4,855 bytes), confirming the affected parent's normal use.
- `npm run build` passed `tsc -b`, then failed in Vite CSS processing because the
  existing `postcss.config.cjs` registers `tailwindcss` directly instead of the
  Tailwind 4 PostCSS plugin. The config and relevant locked package entries are
  identical to the base. This original build/configuration failure is preserved;
  it is not represented as a passing application build or repaired in this patch.

The first consumer probe completed its behavioral assertions but failed while
recording version metadata via the non-exported `@tailwindcss/postcss/package.json`
subpath. Its source/log are retained. The corrected probe reads that local JSON
file directly and passes; install, extraction and application build were not repeated.

## Evidence and limits

The immutable handoff summary identifies the exact PR/base/head/tree and hashes
of the normal resolver/install, original build failure, oxide fallback and
consumer artifacts. Consumer result SHA256:
`f0e677fdb688bb69154660ff123fb71e7d8c0e578d4d0aec700f0e0a22232353`.
Corrected lock SHA256:
`d8ef9815c3ca953c3cab51c380eda358f94fae00a8b331482fcd061150ce1440`.

No malicious archive or decompression-bomb proof was executed. The traced tar
caller is an installation fallback using an official npm archive; attacker input,
runtime exploitability, current deployment and a release requirement remain
unconfirmed. Other dependency alerts, demos, platforms and live chat API behavior
are outside this correction. No full audit, UI matrix or unrelated lint run was added.

This source candidate awaits independent exact-head review and Tower's merge
selection. Deployment is separate. A source revert would restore the previous
vulnerable lock and is not a security remediation or an automatic operational
rollback. No production recovery operation was performed.
