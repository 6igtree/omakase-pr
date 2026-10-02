# omakase-pr 🍣

**English** | [日本語](./README.ja.md)

**PR templates, chosen for you.**

GitHub lets you keep several pull request templates, but gives you [no way to pick one](https://github.com/orgs/community/discussions/146146) when you open a PR. So most teams end up with one template that fits nothing well.

omakase-pr is a skill for Claude Code and Codex. Leave it to the chef: it reads your branch, commits, and diff, picks the right template for the kind of change, fills it in, and opens the PR with `gh pr create`.

## What it looks like

```
> omakase

Opened https://github.com/acme/shop/pull/412
Kind: fix, from the branch name fix/login-timeout.
```

The body follows `.github/PULL_REQUEST_TEMPLATE/fix.md`: the bug, the root cause, the fix, and how it was tested. Checkboxes are ticked only when there is evidence, like tests in the diff. It never claims testing that did not happen.

## Setup, once per repo

The first time you run it, omakase-pr looks at your recent branch names and commit messages and proposes a prefix config:

```yaml
# .github/omakase-pr.yml
feature: [feat, feature]
fix: [fix, bugfix, hotfix]
refactor: [refactor]
docs: [docs]
chore: [chore, ci, build, deps]
```

Then it generates one template per kind in `.github/PULL_REQUEST_TEMPLATE/`. If you already have a template, its sections are kept as your house style in every kind. Templates come in English and Japanese, matched to the language of your recent PRs and commits. You review both before anything is committed.

## How it picks the kind

1. The kind you name: `omakase fix`
2. Your branch name: `fix/login-timeout`
3. Your commit messages: `fix: handle expired session`
4. The diff itself: only docs touched means docs

If it still cannot tell, it asks you once.

## Install

### Claude Code

```
/plugin marketplace add 6igtree/omakase-pr
/plugin install omakase-pr@omakase-pr
```

### Codex

```sh
git clone https://github.com/6igtree/omakase-pr.git /tmp/omakase-pr
mkdir -p ~/.agents/skills && cp -r /tmp/omakase-pr/skills/omakase-pr ~/.agents/skills/
```

Requires the [GitHub CLI](https://cli.github.com/) (`gh`), logged in.

## Usage

| Say | What happens |
| --- | --- |
| `omakase` | Opens a PR with the right template. Runs setup first if needed. |
| `omakase fix` | Same, with the kind you choose. |
| `omakase draft` | Opens it as a draft. Reviewers and labels work too. |
| `omakase-pr preview` | Shows the kind, title, and body without pushing or creating anything. |
| `omakase-pr init` | Runs the setup again. |

## License

MIT
