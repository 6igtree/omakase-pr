---
name: omakase-pr
description: >
  Opens a pull request with the right template for the kind of change
  (feature, fix, refactor, docs, chore, or the repo's own templates), fills
  it in from the diff and commits, and sets the assignee and labels, using
  `gh pr create`. Uses the repo's existing templates as they are, and brings
  its own only when the repo has none. Use when the user says "omakase"
  (alone or with options such as "omakase preview", "omakase init",
  "omakase fix as a draft"), "open a PR", "create a PR", "make a pull
  request", or invokes /omakase-pr.
---

# omakase-pr

GitHub lets a repo have several PR templates but gives no way to pick one.
You pick it for the user, fill it in from the actual change, set the
assignee and labels, and open the PR with `gh pr create`.

You do not run git commands that change anything: no commits, pushes,
branches, or rebases. The user commits and pushes; you open the PR.

## Commands

- `omakase`: open a PR. See "Open a PR".
- `omakase preview`: show the PR without creating it. See "Preview".
- `omakase init`: save a config for this repo, plus templates if the repo
  has none. See "Init".

Anything else the user says is an option for this PR, in plain words: a
kind ("omakase fix"), a draft, reviewers, extra labels, a base branch.
Their words win over the config.

## Config

`.github/omakase-pr.yml` is optional. Without it, use these defaults:

```yaml
# kind: prefixes matched against branch names and commit subjects.
# If the repo has several templates, each kind is a template file name.
kinds:
  feature: [feat, feature]
  fix: [fix, bugfix, hotfix]
  refactor: [refactor]
  docs: [docs]
  chore: [chore, ci, build, deps]

assignees: ["@me"]
labels:            # kind: labels, chosen from the repo's existing labels
  feature: [enhancement]
  fix: [bug]
  docs: [documentation]

# Optional. Uncomment to set these on every PR.
# projects: [Roadmap]
# milestone: v1.2
```

Use a default label only if the repo has it (`gh label list --json name`).

## Find the template

Check in this order and use the first that matches. Never edit what you
find.

1. **Several templates**: a `PULL_REQUEST_TEMPLATE/` directory in
   `.github/`, `docs/`, or the repo root. Each file name, without the
   extension, is a kind. For a kind with no file of its name, use the
   template whose name fits it (`bug_report.md` for fix); if none does,
   the single template from 2; if there is none, the closest one.
2. **One template**: `pull_request_template.md` in the same three places.
   Use it for every kind. The kind still sets the title prefix and labels.
3. **The owner's shared templates**: the same files in the `<owner>/.github`
   repository, read with `gh api repos/<owner>/.github/contents/<path>`.
   Use them as in 1 or 2. Read them each time; never copy them into the
   repo.
4. **None**: this skill's `templates/<kind>.md`. If recent PR titles and
   bodies and commit subjects are mostly Japanese, use
   `templates/ja/<kind>.md`. For another language, translate the English
   ones. If the user names a language, use that. For a kind with no
   default, use What, Why, and Testing sections.

Match file names in any letter case, with `.md` or `.txt`.

## Open a PR

1. **Check the branch.** Stop and tell the user why if:
   - the current branch is the base branch. The base is the user's choice,
     else the default branch
     (`gh repo view --json defaultBranchRef --jq .defaultBranchRef.name`).
   - the branch has no upstream, or `HEAD` differs from it
     (`git rev-parse HEAD @{u}`). Ask the user to push; never push for them.
   - an open PR already exists for this branch
     (`gh pr view --json url,state`, with state `OPEN`). Give its URL.

   Uncommitted changes are not in the PR; mention them in the report.
   Then run `git fetch origin <base>`, and compare against `origin/<base>`
   from here on, so commits already on the base are not counted.
2. **Pick the kind,** stopping at the first that gives a clear answer:
   1. The kind the user named ("omakase fix").
   2. The branch name, matched against the prefixes.
   3. The commit subjects on this branch
      (`git log origin/<base>..HEAD --format=%s`), matched against the
      prefixes. Use the kind most commits share.
   4. What the commit messages say the change does, in any language:
      "Fix login timeout" and "ログインのタイムアウトを修正" are fixes.
   5. The diff itself: only docs touched means docs; only dependency, CI, or
      config files means chore; if no public behavior changes and the tests
      are untouched, it is a refactor. Do not guess between feature and fix
      from the diff alone.
3. **Ask once, if needed.** If the kind is still unclear (offer the two
   likeliest), or nothing says why the change was made (see "Body"), ask
   in one message, and ask nothing else later.
4. **Find the template** for the kind. See "Find the template".
5. **Fill the template** from the commits (`git log origin/<base>..HEAD`)
   and the diff. Start with `git diff --stat origin/<base>...HEAD`, then
   read the diff, skipping lock files, generated files, and binaries.
   - Keep every heading and the order of the template.
   - Replace each `<!-- comment -->` with real content, or delete the section
     if the template says to delete it when it does not apply.
   - Tick a checkbox only when it is true and you can see the evidence.
     A test file in the diff proves a test was added, nothing more. A box that
     claims a result ("fails without this fix", "tests pass", "tested
     manually") needs a command you ran in this session, or the user saying
     so. Otherwise leave it unticked. Never claim testing that did not happen.
   - A section that applies but needs something you do not have, such as
     screenshots of a UI change, keeps its comment. List it in the report.
   - Mention an issue only if its number appears in the branch name,
     commits, or the user's message, and `gh issue view <n> --json title`
     shows it is about this change. A branch like `fix/node-18` has a number
     that is not an issue. Write it as a plain reference (`#123`). Use a
     closing keyword (`Closes`, `Fixes`, `Resolves`) only if the user asks.
   - Write in the language of the template, for a reviewer who has one
     minute. See "Body" under "Writing" below.
6. **Write the title.** See "Title" under "Writing" below.
7. **Create it:**

   ```sh
   gh pr create --base <base> --title "<title>" --body-file <tmp file>
   ```

   Add PR fields from the config, or the defaults, then from the user's
   words for this PR. The user's words win. If `gh repo view --json
   viewerPermission` is `READ`, as when opening a PR from a fork to a repo
   you cannot write to, skip assignees, labels, projects, and milestone.

   | Field | Flag | From the config |
   | --- | --- | --- |
   | Assignees | `--assignee` | `assignees` |
   | Labels | `--label` | `labels.<kind>` |
   | Projects | `--project` | `projects` |
   | Milestone | `--milestone` | `milestone` |
   | Reviewers | `--reviewer` | (user's words only) |
   | Draft | `--draft` | (user's words only) |

   If a label, project, or milestone does not exist, `gh` fails. Before
   retrying, run `gh pr view --json url,state` to make sure no open PR was
   created for this branch. Then create it without that field and tell the
   user which one was skipped and why.
   Adding to a project needs the `project` scope; if it is missing, suggest
   `gh auth refresh -s project`.
8. **Report** in a few lines: the PR URL; the kind and why (for example:
   "fix, from the branch name fix/login-timeout"); the fields set; and
   anything left for the user, such as boxes only they can tick, sections
   left with a comment, skipped fields, and uncommitted changes. Without a
   config, add that `omakase init` saves the settings for this repo.

## Preview (`omakase preview`)

Do steps 1-6 of "Open a PR" without asking anything or creating anything.
If a check in step 1 would stop the PR, say so instead of stopping. Where
step 3 would ask, use the likeliest kind and leave a TODO for the reason
(see "Body"), and say so. Show the kind and why, the title, the fields that
would be set, and the filled body.

If the user then asks to create it, create exactly what you showed, with
the changes they ask for; do not write it again. First run step 1's checks
again, and ask for anything the preview left as a TODO.

## Init (`omakase init`)

1. **Find the templates** as in "Find the template".
2. **Learn the repo's habits.** Look at recent merged PRs
   (`gh pr list --state merged --limit 50 --json headRefName,title,labels`)
   and commit subjects (`git log --format=%s -100`). Collect the prefixes
   actually in use, such as `feat/`, `fix-`, `fix:`, `[Bug]`, and the labels
   each kind of PR carries.
3. **Propose the config.**
   - `kinds`: if you found several templates, one kind per template file,
     with each prefix assigned to the template that fits it best.
     Otherwise, start from the defaults, add the prefixes found in step 2
     to the matching kind, and add a kind only if the repo clearly uses one
     (for example `perf` or `release`).
   - `assignees`: `["@me"]`.
   - `labels`: for each kind, labels from
     `gh label list --json name,description` that describe that kind of
     change, by name (`bug` for fix) or because most merged PRs of that kind
     carry them. Leave out a kind that nothing fits. Never create labels.
   - `projects` and `milestone`: commented out, exactly as in "Config".

   Show the file and apply the user's edits. If a config already exists,
   show it next to the proposal, and overwrite it only if the user agrees.
4. **Write the files**, and show the user the list. Do not commit.
   - The config.
   - If the repo has no templates (4 in "Find the template"), this skill's
     templates for each kind in `.github/PULL_REQUEST_TEMPLATE/`, so the
     team can edit them.
   - If the repo has one template and the user asks to split it by kind,
     write one template per kind in `.github/PULL_REQUEST_TEMPLATE/`. Each
     keeps the existing template's sections and adds the kind's sections
     from this skill's templates. Leave the existing template in place;
     the GitHub web UI still uses it.

## Writing

### Title

The title is the one line a reviewer reads in a list of PRs. It must say what
this PR changes, so they can tell it apart from every other PR.

- Say what changes for the user or the system, not which files you touched.
  Good: `fix: return 401 when the session has expired`.
  Bad: `fix: update auth.py`, `fix: bug fix`, `Login changes`.
- One change, one title. If you need "and" to describe it, name the main
  change and leave the rest to the body.
- At most 60 characters in English, about 35 characters in Japanese,
  including any prefix.
- Follow the repo's style from recent merged PR titles: prefix (`fix:`,
  `[Fix]`), capitalization, and mood. With no history, use the
  conventional-commit form `<type>: <summary>` in the imperative
  ("add", not "added"), with the type that fits the change (`feat`,
  `fix`, `refactor`, `docs`, `chore`), even if the kind is named after a
  template like `bug_report`. No trailing period.
- Do not copy a commit subject blindly. With several commits, summarize the
  whole branch.

### Body

The body is read once, quickly, before the reviewer opens the diff. It must
give them the point and where to look, not retell the diff.

- Lead each section with its most important point.
- Each section: one to three sentences, or up to three bullets. Checklists
  that come from the template stay as they are. The whole body should take
  under a minute to read: about 150 words in English, or about 400 characters
  in Japanese, excluding headings and checkboxes.
- Explain why and what to look at. Do not list every changed file or restate
  code line by line; the diff already shows that.
- Do not repeat the title or the same fact in two sections.
- No filler: no "This PR...", no "In this pull request we...", no summary of
  the summary.
- Delete optional sections that do not apply instead of writing "N/A".
- Never invent the reason. What goes in sections that state why the change
  was made (Why, Background, Motivation, 背景, 理由, 目的) must come from an
  issue, commit messages, code comments, or the user. What was broken can be
  read from the diff and tests; why the change was wanted cannot. If none of
  them say it, ask (step 3 of "Open a PR"). In preview, leave
  `<!-- TODO: why -->` in that section and say so.

Before showing or creating the PR, reread the title and body once and cut
anything a reviewer would not miss.

## Never

- Never invent test results, issue numbers, screenshots, or reviewers.
- Never edit the repo's templates, and never create labels, projects, or
  milestones.
- Never commit, push, or create branches.
