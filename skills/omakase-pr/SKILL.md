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
the PR the user asked for.

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
   - If it has a single `.github/pull_request_template.md` (or the same file in
     `docs/` or the repo root), treat its sections as house style: keep them
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

## Open a PR

1. **Check the branch.** If there are uncommitted changes, ask whether to
   commit them first. If the branch has no upstream, push it with
   `git push -u origin HEAD`. Find the base branch: the user's choice, else
   the repo's default branch.
2. **Pick the kind,** stopping at the first that gives a clear answer:
   1. The kind the user named ("omakase fix").
   2. The branch name, matched against the prefixes.
   3. The commit subjects on this branch (`git log <base>..HEAD --format=%s`),
      matched against the prefixes. Use the kind most commits share.
   4. The diff itself: only docs touched means docs; only dependency, CI, or
      config files means chore; tests plus a small code change near a bug
      report means fix.
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
   - Link an issue only if its number appears in the branch name, commits, or
     the user's message.
   - Write in the language of the template. Write for a reviewer: short,
     concrete, file paths where useful. No marketing tone.
4. **Write the title** in the style of recent merged PR titles in this repo.
   If they use a prefix such as `fix:` or `[Fix]`, use it.
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

## Preview (`omakase preview`)

Do steps 1-4 of "Open a PR", but do not push or create anything. Show the
kind, the reason, the title, the PR fields that would be set, and the filled
body.

## Never

- Never invent test results, issue numbers, screenshots, or reviewers.
- Never overwrite an existing template without asking.
- Never force-push, and never commit changes the user did not ask to commit.
