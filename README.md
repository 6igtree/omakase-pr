# omakase-pr 🍣

**English** | [日本語](./README.ja.md)

**PR templates, chosen for you.**

GitHub lets you keep several pull request templates, but gives you [no way to pick one](https://github.com/orgs/community/discussions/146146) when you open a PR. So most teams end up with one template that fits nothing well.

omakase-pr is a skill for Claude Code and Codex. Leave it to the chef: it reads your branch, commits, and diff, picks the right template for the kind of change, fills it in, sets the assignee and labels, and opens the PR with `gh pr create`.

## What it looks like

```
> omakase

Opened https://github.com/acme/shop/pull/412
Kind: fix, from the branch name fix/login-timeout. Assigned to you, labeled bug.
```

The body follows your fix template: the bug, the root cause, the fix, and how it was tested. Checkboxes are ticked only when there is evidence, like tests in the diff. It never claims testing that did not happen.

The title says what changed in under 60 characters, not which files you touched. The body takes under a minute to read: the point and where to look, not a retelling of the diff. If nothing says why the change was made, it asks you instead of making up a reason.

## Your templates, as they are

No setup is needed. omakase-pr uses the templates your repo already has, and never edits them.

- **Several templates** in `PULL_REQUEST_TEMPLATE/`: each file is a kind, and it picks the one that fits.
- **One template**: used for every kind. The kind still sets the title prefix and labels.
- **Your organization's shared templates** in its `.github` repository: used the same way, and never copied into your repo.
- **None**: it brings its own for feature, fix, refactor, docs, and chore, in English or Japanese to match your recent PRs and commits.

It looks in `.github/`, `docs/`, and the repo root, the same places GitHub does.

## Assignee and labels

Each PR is assigned to you and gets the label for its kind, picked from the labels your repo already has: `bug` for a fix, `enhancement` for a feature, `documentation` for docs. It never creates labels. If you cannot set labels on the repo, as when opening a PR from a fork, it skips them.

## How it picks the kind

1. The kind you name: `omakase fix`
2. Your branch name: `fix/login-timeout`
3. Your commit prefixes: `fix: handle expired session`
4. What your commit messages say: `Handle expired session on login`
5. The diff itself: only docs touched means docs

If it still cannot tell, it asks you once.

## Install

### Claude Code

```
/plugin marketplace add 6igtree/omakase-pr
/plugin install omakase-pr@omakase-pr
```

### Codex

```sh
tmp=$(mktemp -d) && git clone --depth 1 https://github.com/6igtree/omakase-pr.git "$tmp"
mkdir -p ~/.agents/skills && cp -R "$tmp/skills/omakase-pr" ~/.agents/skills/
```

Run the same commands to update.

Requires the [GitHub CLI](https://cli.github.com/) (`gh`), logged in.

## Usage

Commit your changes and push your branch first. omakase-pr opens the PR; it does not commit or push for you.

Try it without creating anything:

```
omakase preview
```

It shows the kind, the title, the labels, and the body. Say "create it" and it opens exactly that PR, or tell it what to change first.

Or open the PR right away:

```
omakase
```

Add whatever you need in plain words:

```
omakase fix, as a draft
omakase with reviewer @alice and label urgent
```

What you say for one PR wins over the config. In Claude Code, `/omakase-pr` works too.

## Settings for your repo

`omakase init` saves settings for your repo in `.github/omakase-pr.yml`. It reads your recent branch names, commit messages, and labels, proposes a config, and writes it once you have checked it:

```yaml
kinds:
  feature: [feat, feature]
  fix: [fix, bugfix, hotfix]
  refactor: [refactor]
  docs: [docs]
  chore: [chore, ci, build, deps]

assignees: ["@me"]
labels:
  feature: [enhancement]
  fix: [bug]
  docs: [documentation]

# Optional. Uncomment to set these on every PR.
# projects: [Roadmap]
# milestone: v1.2
```

If your repo has no templates, `init` also writes its own to `.github/PULL_REQUEST_TEMPLATE/`, so you can edit them to fit your team. Nothing is committed.

## License

MIT
