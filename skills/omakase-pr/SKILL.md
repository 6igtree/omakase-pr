---
name: omakase-pr
description: >
  Opens a pull request with the right template for the kind of change
  (feature, fix, refactor, docs, chore, or the repo's own kinds), filled in
  from the diff and commits, using `gh pr create`. On first use in a repo,
  sets up a prefix config and per-kind templates under
  .github/PULL_REQUEST_TEMPLATE/. Use when the user says "omakase" (alone or
  followed by options such as "omakase fix", "omakase preview", "omakase
  init", "omakase as a draft"), "omakase pr", "open a PR", "create a PR",
  "make a pull request", or invokes /omakase-pr.
---

# omakase-pr

GitHub lets a repo have several PR templates but gives no way to pick one.
You pick it for the user, fill it in from the actual change, and open the PR
with `gh pr create`. Everything else is plain `gh`.

## Files in the repo

- `.github/omakase-pr.yml`: which prefixes map to which kind, and the PR
  fields to set. Everything except `kinds` is optional and empty by default.

  ```yaml
  # kind: prefixes matched against branch names and commit subjects
  kinds:
    feature: [feat, feature]
    fix: [fix, bugfix, hotfix]
    refactor: [refactor]
    docs: [docs]
    chore: [chore, ci, build, deps]

  # Optional. Uncomment to set these on every PR.
  # assignees: ["@me"]
  # labels:          # kind: labels
  #   feature: [enhancement]
  #   fix: [bug]
  # projects: [Roadmap]
  # milestone: v1.2
  ```

- `.github/PULL_REQUEST_TEMPLATE/<kind>.md`: one template per kind. The file
  name is the kind.

If `.github/omakase-pr.yml` is missing, run setup first, then continue with
the PR the user asked for. In preview, do not run setup: use the default
prefixes and this skill's templates, and say that setup has not run yet.

## Setup (`omakase init`, or first use)

1. **Learn the repo's habits.** Look at recent branch names
   (`gh pr list --state merged --limit 50 --json headRefName,title` if
   available, otherwise `git branch -r`) and commit subjects
   (`git log --format=%s -100`). Collect the prefixes actually in use, such as
   `feat/`, `fix-`, `fix:`, `[Bug]`.
2. **Propose the prefix config.** Start from the default above, add prefixes
   found in step 1 to the matching kind, and add a new kind only if the repo
   clearly uses one (for example `perf` or `release`). Include the optional
   fields commented out, exactly as shown above, so the user can find them.
   Do not fill them in. Show the file to the user and apply their edits.
3. **Generate templates**, one per kind:
   - If the repo has templates in `.github/PULL_REQUEST_TEMPLATE/`, keep them
     and map each to a kind. Only create the missing ones.
   - If it has a single `pull_request_template.md` (any letter case, in
     `.github/`, `docs/`, or the repo root), treat its sections as house style: keep them
     in every kind, and add the kind-specific sections from this skill's
     `templates/` directory.
   - If it has none, copy this skill's default templates. For a kind with
     no default, write a short template with What, Why, and Testing.
   - Pick the language from the existing templates, recent PR titles and
     bodies, and commit subjects. If they are mostly Japanese, use
     `templates/ja/<kind>.md`; otherwise use `templates/<kind>.md` (English).
     For another language, translate the English defaults. If the user names
     a language, use that.
4. **Write the files** and show the user the list. Do not commit unless they
   ask. Leave any existing single template in place; the GitHub web UI still
   uses it.

When setup runs as part of opening a PR, the files it wrote are not part of
that PR. Leave them uncommitted, never add them to the PR's commits, and tell
the user they can commit them separately.

## Open a PR

1. **Check the branch.**
   - If a PR already exists for this branch (`gh pr view --json url`), do not
     create another. Ask whether to update its title and body instead, and if
     yes, use `gh pr edit` with the same title and body rules.
   - If there are uncommitted changes other than files written by setup, ask
     whether to commit them first.
   - If the branch has no upstream, push it with `git push -u origin HEAD`.
   - Base branch: the user's choice, else the repo's default branch
     (`gh repo view --json defaultBranchRef --jq .defaultBranchRef.name`).
2. **Pick the kind,** stopping at the first that gives a clear answer:
   1. The kind the user named ("omakase fix").
   2. The branch name, matched against the prefixes.
   3. The commit subjects on this branch (`git log <base>..HEAD --format=%s`),
      matched against the prefixes. Use the kind most commits share.
   4. The diff itself: only docs touched means docs; only dependency, CI, or
      config files means chore; no new public behavior and tests unchanged
      means refactor. Do not guess between feature and fix from the diff alone.
   If it is still unclear, ask once, offering the two likeliest kinds.
3. **Fill the template** from the diff (`git diff <base>...HEAD`) and the
   commits:
   - Keep every heading and the order of the template.
   - Replace each `<!-- comment -->` with real content, or delete the section
     if the template says to delete it when it does not apply.
   - Tick a checkbox only when it is true and you can see the evidence.
     A test file in the diff proves a test was added, nothing more. A box that
     claims a result ("fails without this fix", "tests pass", "tested
     manually") needs a command you ran in this session, or the user saying
     so. Otherwise leave it unticked. Never claim testing that did not happen.
   - Mention an issue only if its number appears in the branch name, commits,
     or the user's message. Write it as a plain reference (`#123`), never with
     a closing keyword (`Closes`, `Fixes`, `Resolves`).
   - Write in the language of the template, for a reviewer who has one
     minute. See "Body" under "Writing" below.
4. **Write the title.** See "Title" under "Writing" below.
5. **Create it:**

   ```sh
   gh pr create --base <base> --title "<title>" --body-file <tmp file>
   ```

   Add PR fields from the config, then from the user's words for this PR.
   The user's words win over the config. Set nothing that neither provides.

   | Field | Flag | From the config |
   | --- | --- | --- |
   | Assignees | `--assignee` | `assignees` |
   | Labels | `--label` | `labels.<kind>` |
   | Projects | `--project` | `projects` |
   | Milestone | `--milestone` | `milestone` |
   | Reviewers | `--reviewer` | (user's words only) |
   | Draft | `--draft` | (user's words only) |

   If a label, project, or milestone does not exist, `gh` fails. Before
   retrying, run `gh pr view` to make sure no PR was created for this branch.
   Then create it without that field and tell the user which one was skipped
   and why.
   Adding to a project needs the `project` scope; if it is missing, suggest
   `gh auth refresh -s project`.
6. **Report** the PR URL, the kind chosen, and why (for example: "fix, from
   the branch name fix/login-timeout"), in one or two lines.

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
  conventional-commit form `<kind>: <summary>` in the imperative
  ("add", not "added"). No trailing period.
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
- Never invent the reason. Sections that state why the change was made (Why,
  Background, Motivation, 背景, 理由, 目的) must come from an issue, commit
  messages, code comments, or the user. What was broken can be read from the
  diff and tests; why the change was wanted cannot. If none of them say it,
  ask the user once in one line before creating the PR. In preview, leave
  `<!-- TODO: why -->` in that section and say so.

Before showing or creating the PR, reread the title and body once and cut
anything a reviewer would not miss.

## Preview (`omakase preview`)

Do steps 1-4 of "Open a PR", but do not push or create anything. Show the
kind, the reason, the title, the PR fields that would be set, and the filled
body.

## Never

- Never invent test results, issue numbers, screenshots, or reviewers.
- Never overwrite an existing template without asking.
- Never force-push, and never commit changes the user did not ask to commit.
