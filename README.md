# Orng Catalog

Community content for Bitwig Studio - devices, modulators and Grid modules -
that [Orng Registry](https://github.com/zezic/orng-tools) installs from.

**This is not a file host.** An item is twenty to thirty kilobytes; the entire
Bitwig factory device set is 4.7 MB. Hosting is a non-problem.

Orng Catalog is an **identity authority**. Its job is to guarantee that a UUID
means one thing, permanently, across contributors who do not know each other.
Every rule below follows from that, and from one fact about Bitwig: a project
stores a device's identity and its parameter values, while the structure comes
from a file resolved by that identity. Republishing changed structure under a
UUID someone already used therefore reaches back into projects that exist.

## Layout

```
content/
  <author>/
    <slug>/
      <name>.bwdevice      the document; device, modulator or Grid module
      orng.toml            what the document cannot say about itself
      README.md            optional
owners.toml                author -> GitHub account
index.json                 generated, never hand-edited
```

One content root, grouped by author rather than by kind. A document already
states its own kind and the index republishes it, so a path that stated it too
would be a third copy to keep in step. Grouping by author is also what
per-directory ownership is checked against: an author owns one prefix rather
than three, and their items stay together for whoever reviews them.

`orng.toml` carries **only what is not already inside the document**:

```toml
version    = "1.0.0"
author     = "caviio"
license    = "MIT"
min_bitwig = "6.1"
# homepage   = "https://example.com"
# supersedes = ["..."]
```

Identity, name, kind, description and category are read from the document
itself, and a manifest that tries to restate them is refused.

## Contributing

Nobody has write access, including maintainers. Everything arrives as a fork
pull request.

1. Fork, and add your item under `content/<you>/<slug>/`.
2. Open a pull request. If you are new, add yourself to `owners.toml` in the
   same pull request; a maintainer reviews that once.
3. Checks run. If you already own every path you touched, it merges by itself.

### The rules the checks apply

1. **A UUID is permanent.** Once merged it is bound to that item forever.
2. **Uniqueness is checked, never assumed.** Name-derived UUIDs collide
   deterministically, so new submissions should use random identities.
3. **Content changes under a stable identity are constrained.** A compatible
   change keeps the UUID and raises the version. Anything that alters the
   parameter set or its order takes a **new** UUID in a new directory, with the
   old one left published so existing projects still load, linked by
   `supersedes`.
4. **Display names should be unique.** Bitwig's browser is flat. A collision
   warns rather than blocks; the application offers a rename at install time,
   which is free because the name is not part of the identity.

### Checking before you push

The validator is the application's own library, published as a binary, so the
thing that gates a merge here and the thing that installs from the catalog
cannot disagree about what a valid item is.

```bash
cargo install --git https://github.com/zezic/orng-tools --locked orng-catalog-lint
orng-catalog-lint check --root .
```

## How a contribution is judged

GitHub has no path-scoped write permission, and `CODEOWNERS` grants no access -
it routes and can require review, nothing more. So ownership is enforced in CI.

| Workflow | Trigger | Token | What it does |
| --- | --- | --- | --- |
| `pull-request.yml` | `pull_request` | read-only | Ownership first, because it is the cheap rejection, then validation |
| `auto-merge.yml` | `workflow_run` | write | Re-decides ownership itself, and enables auto-merge for an owner update |
| `publish.yml` | push to `master` | write | Regenerates `index.json` from history and publishes it |

Two details are load-bearing and easy to undo by accident:

- **The owners file is read from the base branch**, never from the pull request.
  A contributor checked against their own copy would simply add themselves as
  owner of someone else's directory in the commit that edits it.
- **The numeric account id is what is compared**, not the login. A login can be
  changed by its holder, and the freed name can then be claimed by somebody
  else, who would inherit access. An account id is immutable and never reused.

`auto-merge.yml` is the only workflow here with a write token, so it never
checks out the pull request and believes nothing the pull request's own run
produced. `pull_request` workflows execute the fork's copy of themselves, which
is safe only because their token is read-only and nothing downstream trusts
their output - so the privileged workflow re-runs the ownership check against
`master`'s owners file and the file list as the API reports it.

A new directory can never auto-merge, which is also when a first-time
contributor's work should get eyes on it.

## Setting this repository up

**The workflows above are not protection on their own.** Rulesets, required
status checks and auto-merge are repository settings, and settings cannot be
committed. Until these are applied, this repository only looks guarded.

- [ ] **Settings > Actions > General > Workflow permissions**: *Read and write
      permissions*. Without it `auto-merge.yml` and `publish.yml` cannot get the
      write token their `permissions:` blocks ask for.
- [ ] **Settings > General > Pull Requests**: enable *Allow auto-merge*, and
      *Allow squash merging*. Squash is what makes one merged contribution one
      commit on `master`, which is what `git log -1 --first-parent` over an
      item's directory names in the index.
- [ ] **Ruleset on `master`**: require a pull request before merging; require
      the status checks **`Ownership`** and **`Validate`**; block force pushes.
- [ ] **In that ruleset, restrict file paths** so that `.github/**`,
      `owners.toml` and `index.json` cannot be changed by a pull request.
      Maintainers, and the Actions bot for `index.json`, are the only bypass.
- [ ] **Confirm the bypass list** lets `github-actions[bot]` push `index.json`
      to `master`, or `publish.yml` will fail at its commit step.
- [ ] **Bump `LINT_REV`** in all three workflows. It currently holds a
      placeholder that predates `index --from-git`, which `publish.yml` needs,
      so publishing will fail until it points at a commit of `zezic/orng-tools`
      that has it. It is pinned on purpose: the rules that gate merges must not
      change silently under the repository, so moving to a newer validator is a
      visible commit bumping that line.

Verify the result by opening a pull request from a throwaway account that owns
nothing. It should be refused by `Ownership`, not merged and not left green.

## Distribution

`index.json` is published as a release asset on every merge, so the application
fetches one stable URL:

```
https://github.com/zezic/orng-catalog/releases/latest/download/index.json
```

A release per merge, rather than one rolling release updated in place, so that
what the URL serves is always tied to a single reviewed commit.

Each row carries a digest and a size, and the change that published it. That
last one is the review a user is shown before installing: a document is a DSP
graph Bitwig executes, and review is the only trust boundary there is.

## Licensing

Each item states its own licence in `orng.toml`, and an author's choice for
their device is independent of anything else here. The tooling is licensed
separately; see [LICENSING.md](https://github.com/zezic/orng-tools/blob/master/LICENSING.md).

Not affiliated with or endorsed by Bitwig GmbH.
