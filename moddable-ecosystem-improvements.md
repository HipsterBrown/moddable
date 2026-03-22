# Moddable Ecosystem Improvement Plan

> Compiled from analysis of the Moddable SDK `public` branch (v7.2.0, Jan 2026).
> Covers the SDK itself, xs-dev, embedded.js.org, community infrastructure, and ecosystem growth.

---

## Priority framework

- **P0** — Blocks new developers from getting started or kills retention in the first hour
- **P1** — High-leverage: benefits the majority of active developers, enables ecosystem growth
- **P2** — Meaningful quality-of-life improvements for established users
- **P3** — Valuable but non-urgent; good for contributors

---

## 1. Onboarding & Developer Experience

### P0

- **`xs-dev` official endorsement** — xs-dev eliminates the hardest parts of SDK setup (pre-compiled tools, env vars, device SDK install) but is maintained by one community member. Moddable should officially endorse it, link it from the main README as the recommended install path, and establish a contribution/maintenance relationship. Model: `rustup` is the official Rust toolchain manager, not a community side project.

- **Dead Gitter link** — `contributing.md` links to a Gitter chatroom that shut down in 2023. Any new developer who clicks it hits a dead end. Replace immediately.

### P1

- **REPL / dev-mod host** — The biggest DX gap vs. MicroPython/CircuitPython. Implementation path: pre-built "dev host" firmware per supported device (flashed once via `xs-dev flash-dev-host --device esp32/moddable_two`), then all iteration via `mcpack mcrun`. Host preserves eval/parser, includes full ECMA-419 IO + networking. The `examples/packages/run-mod` already demonstrates the pattern; it needs promotion and an official host image per device.

- **Onboarding friction audit** — Document time-to-first-blink for each supported platform. Identify and fix the steps that consistently block new users (currently: env var setup, ESP-IDF version matching, understanding manifest vs. package.json).

- **`xs-dev` container / Nix support** — A `devcontainer.json` or Nix flake for reproducible environments. Critical for teams, CI, and developers on locked-down machines.

### P2

- **`xs-dev repl` command** — A CLI wrapper around the dev-mod host that hides the `mcrun` invocation and streams `trace()` output. Even a 2–3 second compile round-trip is acceptable if the UX is `type → enter → see result`.

- **Guided project templates** — `xs-dev init` with template options: `--template ble-peripheral`, `--template display-app`, `--template wifi-sensor`. Each template includes a working manifest, example main.js, and README explaining what to change.

---

## 2. Documentation & Discoverability

### P0

- **API index / search** — 88 markdown docs with no search. A Docusaurus or Starlight site (xs-dev already uses Starlight as a model) with full-text search would make existing content 10x more accessible. The content quality is high; the delivery mechanism is the problem.

### P1

- **`mcpack` + npm ecosystem guide** — The most important undocumented feature in the SDK. Deserves a dedicated top-level guide covering: auto-detected globals, `"moddable"` condition in package.json imports, npm package compatibility rules, the `@moddable/` npm scope, and the full `mcpack mcconfig` / `mcpack mcrun` workflow. Currently buried in `examples/packages/readme.md`.

- **ECMA-419 → Moddable SDK bridge** — Each ECMA-419 class on embedded.js.org should link to the corresponding Moddable `modules/io/` source and a runnable example. Each Moddable IO module doc should link back to the ECMA-419 spec section. Currently these two reference systems are islands.

- **`pins/` → `io/` (ECMA-419) migration guide** — Two hardware APIs coexist with no guidance on which to use or how to migrate. The ESLint config marks `Host` and `System` globals as deprecated but there's no migration path documented. A clear guide + deprecation timeline would reduce confusion.

- **Changelog** — `tools/VERSION` is updated but there's no `CHANGELOG.md` and GitHub releases have no notes. Developers can't know what changed between versions. Even a manually maintained changelog covering breaking changes and major additions would help.

### P2

- **`nodered2mcu` dedicated documentation page** — Currently buried in `tools/` with minimal docs. This is a potential killer feature for the IoT automation audience (Node-RED has millions of users). It deserves its own guide, a video, and a featured example.

- **`xsbug` tutorial** — The debugger is best-in-class but most developers don't know it exists or how to use the memory instrumentation panel. A screencast + written guide walking through a real debugging session would be high-impact.

- **Platform conformance table** — A single page showing which ECMA-419 IO classes are available on each supported platform (ESP32, ESP8266, nRF52, RP2040, Zephyr, etc.). The #1 question every hardware developer asks before starting.

### P3

- **Inline API examples in documentation** — Most API docs describe parameters but have no runnable code examples. Adding a minimal working example to each class/method description significantly reduces the gap between "reading docs" and "writing working code."

- **Video content** — MicroPython and CircuitPython have extensive YouTube ecosystems; Moddable has almost none outside Moddable's own account. The simulator is uniquely well-suited to screencasts since no hardware is required.

---

## 3. embedded.js.org (ECMA-419 Reference)

### P1

- **Complete stub pages before adding more** — Several existing pages ("Oxygen Sensor", "Ethernet Network Interface") have no content yet. A complete reference for the 10 most-used IO classes (Digital, Analog, I2C, SPI, Serial, PWM, PulseCount, PulseWidth) is more valuable than broad-but-thin coverage.

- **Conformance table** — Which platforms implement which classes. The most practical question for a developer evaluating whether ECMA-419 works for their hardware.

- **j5e cross-promotion** — `dtex/j5e` builds on ECMA-419 and targets the same audience. Cross-linking would bring in the Johnny-Five / Firmata community.

### P2

- **"Try it" links** — Each class page links to a Moddable simulator or xs-dev runnable example. Documentation becomes a starting point, not just a reference.

- **Contributor guide** — The site accepts contributions but the path to contributing a new class doc isn't obvious. A `CONTRIBUTING.md` with the doc template and review process would lower the bar.

### P3

- **Version selector** — ECMA-419 is on its 3rd edition (June 2025). As the spec evolves, the site should indicate which edition each API was introduced in.

---

## 4. Package Ecosystem

### P1

- **`@moddable` npm org conventions** — Establish and document a convention for publishing Moddable-compatible npm packages: `"moddable"` keyword, `"moddable"` condition in imports, `exports` field structure. A published style guide + at least 3–4 reference packages (`@moddable/eventemitter`, `@moddable/cbor`, `@moddable/mqtt-client`) seeds the pattern.

- **`mcpack` compatibility badge / CI action** — A GitHub Action that runs `mcpack mcconfig -t build` for simulator, allowing npm package authors to verify their package works with Moddable before publishing. A passing badge on a package README signals Moddable-compatibility to developers.

### P2

- **Curated "works with Moddable" list** — A community-maintained page (on embedded.js.org or the Moddable docs site) of npm packages tested with `mcpack`. Browseable by category: data encoding, sensors, cloud APIs, UI utilities.

- **`xs-dev` package search** — `xs-dev search cbor` queries npm for packages with the `"moddable"` keyword and shows compatibility info. Discoverability from the CLI that developers already use.

### P3

- **`@moddable/testing` package** — A standard test assertion library compatible with `testmc`. Currently there's no standard assertion API; tests use ad-hoc patterns. A tiny, well-documented assertion library would encourage community modules to ship tests.

---

## 5. Production & Deployment

### P1

- **OTA update reference implementation** — The device-side `update` module exists for ESP32 but there's no reference server or documented protocol. A minimal HTTP server spec (or documented Golioth/Hawkbit integration) that the `update` module can poll is the #1 gap between "works on my desk" and "runs in production." This should be an official Moddable reference, not left to each developer to invent.

- **GitHub Actions for Moddable builds** — A `moddable-build` action that installs the SDK at a pinned version and runs `mcconfig -t build` for the simulator. This unblocks all open-source Moddable libraries from having CI. Currently there is nothing. Highest-leverage single contribution for ecosystem health.

### P2

- **Firmware signing pipeline** — A `mcconfig` post-build step to sign firmware images + device-side verification before applying OTA. The security primitive that enterprise/industrial customers need before deploying at scale.

- **`xs-dev deploy --ota`** — CLI command wrapping `mcbundle` + HTTP upload to a device over mDNS/local network. A complete deploy story without requiring any cloud vendor.

- **Golioth integration reference** — Golioth is the cloud platform most aligned with the Moddable use case (resource-constrained MCUs, FOTA, LightDB state). A `@moddable/golioth` package covering OTA + telemetry + device management would give Moddable a credible answer to "how do I run this in production?" with a reputable cloud partner.

### P3

- **Balena / fleet management guide** — Balena supports ESP32 and is the standard answer for fleet management. A guide or community integration for managing Moddable devices via Balena would complete the production story.

---

## 6. Community Infrastructure

### P1

- **Discord server** — The primary expected gathering place for real-time support and community building. Best for quick help, show-and-tell, and converting lurkers to contributors. Lower barrier than any forum. Should exist alongside async resources, not replace them.

- **Async knowledge base** — Discord is ephemeral; answered questions disappear. A linked Discourse or GitHub Discussions for evergreen Q&A, project showcases, and design RFCs. The expectation: solved Discord threads get summarized to the permanent forum.

### P2

- **CLA bot** — The current CLA process (email a PDF to info@moddable.com) is a significant contributor friction point. A GitHub CLA bot (cla-assistant.io is free for open source) reduces this to a single checkbox in a PR.

- **Contributor guide expansion** — `contributing.md` covers the basics but has no dev environment setup, no PR workflow description, no code style guide, and no "good first issue" convention. A fuller guide would reduce the activation energy for first contributions.

- **Regular community showcases** — A monthly "what are people building" post/stream. The `contributed/` directory has interesting projects (modClock, BLE challenge, conversationalAI) but they have no visibility. Regular showcases build community identity and generate content.

### P3

- **"Awesome Moddable" list** — A community-maintained `awesome-moddable` GitHub repo. Standard format, searchable, signals ecosystem health to evaluators.

---

## 7. SDK Architecture (Lower Priority Technical Debt)

### P2

- **`pins/` API deprecation roadmap** — Publish a timeline for deprecating the legacy `pins/` API in favor of ECMA-419 `io/`. The APIs coexist silently today with no guidance. A public timeline gives library authors time to migrate and prevents new code from using the deprecated path.

- **`MODDABLE_MODULES` env var override** — Allow pointing at a different modules directory while using the same tools installation. Enables multiple SDK versions on one machine without full path switching. Low complexity change, high value for power users and CI.

### P3

- **XS engine as standalone C library** — The `xs/` directory is already self-contained. Publishing it as a versioned, independently releasable C library would enable Zephyr module, Arduino library, and other integrations that currently require cloning the entire SDK repo.

- **Repo split evaluation** — The monorepo couples tools, modules, examples, and engine at the same version. The cost is that multi-version setups are awkward. A formal evaluation of splitting `xs/` (stable, slow-changing) from `modules/` + `examples/` (fast-changing) is worth doing even if the conclusion is "stay monorepo."

---

## Summary matrix

| Area | P0 | P1 | P2 | P3 |
|---|---|---|---|---|
| Onboarding | xs-dev endorsement, dead Gitter link | REPL/dev-mod host, friction audit | xs-dev container, project templates | — |
| Documentation | API search site | mcpack guide, migration guide, changelog | nodered2mcu docs, xsbug tutorial | Inline examples, video content |
| embedded.js.org | — | Complete stubs, conformance table | "Try it" links, contributor guide | Version selector |
| Package ecosystem | — | @moddable conventions, CI action | Curated compat list, xs-dev search | Testing library |
| Production | — | OTA reference impl, GitHub Actions | Firmware signing, xs-dev deploy | Balena guide |
| Community | — | Discord, async knowledge base | CLA bot, contributor guide | Awesome list |
| SDK architecture | — | — | API deprecation roadmap, MODDABLE_MODULES | XS standalone lib, repo split eval |
