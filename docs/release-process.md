# Release Process

## Platform support

| Platform                    | Supported                            |
| --------------------------- | ------------------------------------ |
| Linux 64-bit                | :white_check_mark:                   |
| MacOS 64-bit                | :white_check_mark:                   |
| Windows 64-bit              | :white_check_mark:                   |
| MacOS Apple Silicon (ARM64) | _builds are provided but not tested_ |
| Linux ARMv7                 | _builds are provided but not tested_ |
| Linux ARM64                 | _builds are provided but not tested_ |


## Release Schedule and Hotfixes

Normal releases in this repository that add features such as improved RPC endpoints, improved boot-up time, new event
observer fields or event types, etc., are released on a monthly schedule. The currently staged changes for such releases
are in the [develop branch](https://github.com/stacks-network/stacks-core/tree/develop). It is generally safe to run
a `stacks-node` from that branch, though it has received less rigorous testing than release tags. If bugs are found in
the `develop` branch, please do report them as issues on this repository.

For fixes that impact the correct functioning or liveness of the network, _hotfixes_ may be issued. These are patches
to the main branch which are backported to the develop branch after merging. These hotfixes are categorized by priority
according to the following rubric:

- **High Priority**. Any fix for an issue that could deny service to the network as a whole, e.g., an issue where a particular kind of invalid transaction would cause nodes to stop processing requests or shut down unintentionally. Any fix for an issue that could cause honest miners to produce invalid blocks.
- **Medium Priority**. Any fix for an issue that could cause miners to waste funds.
- **Low Priority**. Any fix for an issue that could deny service to individual nodes.

## Versioning

This repository uses a 5 part version number.

```
X.Y.Z.A.n

X = 2 and does not change in practice unless there’s another Stacks 2.0 type event
Y increments on consensus-breaking changes
Z increments on non-consensus-breaking changes that require a fresh chainstate (akin to semantic MAJOR)
A increments on non-consensus-breaking changes that do not require a fresh chainstate, but introduce new features (akin to semantic MINOR)
n increments on patches and hot-fixes (akin to semantic PATCH)
```

For example, a node operator running version `2.0.10.0.0` would not need to wipe and refresh their chainstate
to upgrade to `2.0.10.1.0` or `2.0.10.0.1`. However, upgrading to `2.0.11.0.0` would require a new chainstate.

## Non-Consensus Breaking Release Process

For non-consensus breaking releases, this project uses the following release process:

1. The release must be timed so that it does not interfere with a _prepare
   phase_. The timing of the next Stacking cycle can be found
   [here](https://stx.eco/dao/tools?tool=2). A release should happen
   at least 24 hours before the start of a new cycle, to avoid interfering
   with the prepare phase. So, start by being aware of when the release can
   happen.

1. Before creating the release, the release manager must determine the _version
   number_ for this release, and create a release branch in the format: `release/X.Y.Z.A.n`. 
   The factors that determine the version number are
   discussed in [Versioning](#versioning). We assume, in this section,
   that the change is not consensus-breaking. So, the release manager must first
   determine whether there are any "non-consensus-breaking changes that require a
   fresh chainstate". This means, in other words, that the database schema has
   changed, but an automatic migration was not implemented. Then, the release manager
   should determine whether this is a feature release, as opposed to a hotfix or a
   patch. Given the answers to these questions, the version number can be computed.
   
1. The release manager enumerates the PRs or issues that would _block_
   the release. A label should be applied to each such issue/PR as
   `X.Y.Z.A.n-blocker`. The release manager should ping these
   issue/PR owners for updates on whether or not those issues/PRs have
   any blockers or are waiting on feedback.

1. The release manager must update the `CHANGELOG.md` file with summaries what
   was `Added`, `Changed`, and `Fixed`. The pull requests merged into `develop`
   can be found
   [here](https://github.com/stacks-network/stacks-core/pulls?q=is%3Apr+is%3Aclosed+base%3Adevelop+sort%3Aupdated-desc). Note, however, that GitHub apparently does not allow sorting by
   _merge time_, so, when sorting by some proxy criterion, some care should
   be used to understand which PR's were _merged_ after the last release. 

1. Once the blocker PRs have merged, the release manager will create a new tag
   by manually triggering the [`CI` Github Actions workflow](https://github.com/stacks-network/stacks-core/actions/workflows/ci.yml)
   against the `release/X.Y.Z.A.n` branch.
   
1. Once the release candidate has been built, and docker images, etc. are available,
   the release manager will notify various ecosystem participants to test the release
   candidate on various staging infrastructure:

   1. Stacks Foundation staging environments.
   1. Hiro PBC testnet network.
   1. Hiro PBC mainnet mock miner.

   The release candidate should be announced in the `#stacks-core-devs` channel in the
   Stacks Discord. For coordinating rollouts on specific infrastructure, the release
   manager should contact the above participants directly either through e-mail or
   Discord DM. The release manager should also confirm that the built release on the
   [Github releases](https://github.com/stacks-network/stacks-core/releases/)
   page is marked as `Pre-Release`.

1. The release manager will test that the release candidate successfully syncs with
   the current chain from genesis both in testnet and mainnet. This requires starting
   the release candidate with an empty chainstate and confirming that it synchronizes
   with the current chain tip.

1. If bugs or issues emerge from the rollout on staging infrastructure, the release
   will be delayed until those regressions are resolved. As regressions are resolved,
   additional release candidates should be tagged. The release manager is responsible
   for updating the `develop -> master` PR with information about the discovered issues,
   even if other community members and developers may be addressing the discovered
   issues.

1. Once the final release candidate has rolled out successfully without issue on staging
   infrastructure, the tagged release shall no longer marked as Pre-Release on the [Github releases](https://github.com/stacks-network/stacks-core/releases/) page. 
   Announcements will then be shared in the `#stacks-core-devs` channel in the
   Stacks Discord, as well as the [mailing list](https://groups.google.com/a/stacks.org/g/announce).

1. Finally, the release branch `release/X.Y.Z.A.n` will be PR'ed into the `master` branch, and once merged, a PR for `master->develop` will be opened. 
