# Contributing

Thanks for helping with Notes API. This page explains how we name branches, how we write commits, and what we expect from pull requests.

## Branch names
add 123
Create a branch from `main` for every change. Use this pattern:

```
<type>/<short-description>
```

- `<type>` is one of: `feature`, `fix`, `docs`, `chore`, `test`.
- `<short-description>` is lowercase, words separated by hyphens, 2 to 5 words.
- If the work has an issue, put its number first: `fix/12-ready-returns-503`.

Examples: `feature/note-search`, `docs/add-contributing`, `fix/12-ready-returns-503`.

Do not commit directly to `main`.

## Commit messages

We follow a simplified [Conventional Commits](https://www.conventionalcommits.org/) style:

```
<type>: <short summary>

<optional body: what changed and why>
```

- `<type>` is one of: `feat`, `fix`, `docs`, `chore`, `test`, `refactor`.
- The summary is in the imperative mood ("add", not "added"), starts with a lowercase letter, has no trailing period, and is at most 72 characters.
- Use the body to explain why the change was made, not only what changed.
- Keep each commit to one logical change.

Examples:

```
feat: add GET /notes/{id} endpoint
docs: describe the /ready endpoint
fix: return 404 for a missing note
```

## Pull requests

- Open a pull request against `main` from your branch.
- Keep it small and focused: one topic per pull request.
- Use a title in the same format as a commit summary, for example `docs: add CONTRIBUTING.md`.
- In the description, say what you changed and why, and link the issue (for example `Closes #12`).
- Before requesting review, make sure the checks pass locally:
  ```
  pytest
  ruff check .
  ```
- Ask at least one teammate to review. Do not merge your own pull request without a review.
- Answer review comments by pushing new commits or replying; resolve a conversation only after it is addressed.
- Delete the branch after it is merged.

## Branching strategy: proposal by Namute2

   **Proposal: GitHub Flow.** `main` is always deployable; every change is a short-lived branch merged by pull request after review.

   **Why:** it matches the rules we already wrote (branch per change, PR with review), it is easy to learn, and it works well with a pipeline that deploys from `main`.