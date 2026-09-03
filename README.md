# Typewriter

Typewriter is a Contentful-owned fork of Segment's CLI for generating strongly typed analytics clients from Tracking Plans. The fork exists to protect the performance of bulk AI actions by reusing compiled AJV validators during runtime schema validation. It is owned by `group:team-core-ai` and published as the restricted `@contentful/typewriter` package.

## Why This Fork Exists

This validator-cache change is the primary reason Contentful maintains a fork. Bulk AI actions can validate many analytics messages in one operation. Upstream generated clients created a new AJV instance and recompiled the same event schema for every validation; at bulk-action volume, that repeated compilation would severely degrade performance.

The Contentful templates create one AJV instance and cache compiled validators by schema `$id`:

```text
Before: event -> create AJV -> compile schema -> validate
After:  event -> look up schema $id -> reuse validator
                               cache miss -> compile once -> cache -> validate
```

This behavior is implemented in the analytics-js and analytics-node templates. It applies to generated development/runtime-validation clients; production output does not include AJV. Schemas without a `$id` cannot use the cache and are compiled on demand.

The main maintenance rule for this repository is: **never move AJV construction or schema compilation back into the per-event validation path.** See [ARCHITECTURE.md](./ARCHITECTURE.md) for the fork-specific design and [the cache ADR](./docs/ADRs/2026-02-04-cache-ajv-validators.md) for the decision record.

## Getting Started

| Tool | Version | Notes |
| --- | --- | --- |
| Node.js | 18 (see `.nvmrc`) | Run `nvm use`. |
| Yarn | Not pinned | CI uses Yarn with the committed `yarn.lock`. |
| GitHub Packages token | Authorized for `@contentful` | Export `GITHUB_TOKEN`; `.npmrc` reads it. |

```bash
git clone git@github.com:contentful/typewriter.git
cd typewriter
yarn install --frozen-lockfile   # source: .github/workflows/ci.yml
yarn build                       # source: package.json → scripts.build
```

## Running Tests

```bash
yarn test   # source: package.json → scripts.test
```

The suite uses Jest snapshot integration tests plus generated-client TypeScript checks. It does not require a local service; fixtures are created under `test-env/` (`jest.config.js`, `src/__tests__/`).

## Documentation Map

| Document | What it covers |
| --- | --- |
| [ARCHITECTURE.md](./ARCHITECTURE.md) | Internal structure, data flows, and integration points |
| [AGENTS.md](./AGENTS.md) | Agent-facing routing and guardrails |
| [CONTRIBUTING.md](./CONTRIBUTING.md) | Code-generation and release procedures |
| [Architecture Decisions](./docs/ADRs/) | Why significant choices were made |

## For AI Agents

If you are an AI coding agent working in this repository, read [AGENTS.md](./AGENTS.md) first. It tells you where to find architectural context, development setup, decision records, and repo-specific rules.

## Preserved Upstream Documentation

The content below is preserved from the inherited README for team review. When it conflicts with the Contentful setup, test, or release instructions above, follow the Contentful instructions above.

## Contentful Fork

This repository is a fork of Segment Typewriter. Its primary maintained difference is the validator-cache behavior described above.

Summary of changes:
- Reuse cached AJV validators during runtime schema validation to avoid recompilation overhead in bulk AI actions.

Use this package with `npx @contentful/typewriter`.

Since this repo will likely only rarely be changed publishing happens manually.To publish a new version of this package increment the version in package.json and run:

Publishing requires explicit approval.

```sh
  yarn build
  npm publish
```

<p align="center">
	<br>
	<br>
  <img src=".github/assets/typewriter-logo.svg?sanitize=true" alt="Typewriter logo" />
  <br>
  <br>
  <br>
  <br>
  <a href="http://www.npmjs.com/package/typewriter">
    <img src="https://img.shields.io/npm/v/typewriter.svg" alt="NPM Version">
  </a>
  <a href="./.github/LICENSE">
    <img src="https://img.shields.io/npm/l/typewriter.svg" alt="License">
  </a>
  <a href="https://snyk.io/test/github/segmentio/typewriter?targetFile=package.json">
    <img src="https://snyk.io/test/github/segmentio/typewriter/badge.svg?targetFile=package.json" alt="Known Vulnerabilities" data-canonical-src="https://snyk.io/test/github/segmentio/typewriter?targetFile=package.json">
  </a>
  <br>
  <br>
  <br>

  <img src=".github/assets/readme-example.gif" alt="Typewriter GIF Example" width="70%"/>
</p>

- 💪 **Strongly Typed Analytics**: Generates strongly-typed [Segment](http://segment.com) analytics clients that provide compile-time errors, along with intellisense for event/property names, types and descriptions.

- 👮 **Analytics Testing**: Validate your instrumentation matches your [spec](https://segment.com/docs/protocols/tracking-plan/) before deploying to production, so you can fail your CI builds without a manual analytics QA process.

- 🌐 **Cross-Language Support**: Supports native clients for `analytics.js`, `analytics-node`, `Analytics-Kotlin`, and `Analytics-Swift`.

- ✨ **Segment Protocols**: Built-in support to sync your `typewriter` clients with your [centralized Segment Tracking Plans](https://segment.com/docs/protocols/tracking-plan/).

## Get Started

```sh
# Walks you through setting up a `typewriter.yml` and generating your first client.
$ npx @contentful/typewriter init
```

For more instructions on setting up your `typewriter` client, such as adding it to your CI, see our [documentation](https://segment.com/docs/protocols/typewriter).

## Contributing

- To submit a bug report or feature request, [file an issue here](https://github.com/contentful/typewriter/issues).
- To develop on `typewriter` or propose support for a new language, see [our contributors documentation](./CONTRIBUTING.md).

## Migrating from v7

Check the instructions on our [documentation](https://segment.com/docs/protocols/typewriter)

- You'll need to change your Segment Config API Token for a Public API Token
- v8 doesn't support **Analytics-iOS** nor **Analytics-Android**. We recommend using **Analytics-Swift** and **Analytics-Kotlin** instead, which are supported.
  If you need to use these libraries you can run v7 specifying the version with your commands:

```sh
$ npx typewriter@7 build
```
