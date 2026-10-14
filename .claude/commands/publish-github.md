---
description: Security-scan the project, then push to GitHub, update README and repo About, and deploy GitHub Pages via Actions
argument-hint: <owner>/<repo> [branch=main] [about text]
---

Publish this project to GitHub. Arguments: `$ARGUMENTS`

Parse the arguments as `<owner>/<repo>`, an optional branch (default `main`) and an optional About description. If the owner/repo is missing, ask the user for it before doing anything else. Also ask whether the repo should be public or private if it does not exist yet. Remember that Pages on a public repo makes the site public.

Work through the steps in order. Stop and report if any step fails. Never skip step 1.

## 1. Security scan (before anything leaves this machine)

Scan every file that would be committed (tracked files plus untracked files that are not git-ignored). Use Grep and Glob, not guesses.

- **Secrets:** look for API keys, tokens, passwords, private keys, and connection strings. Patterns include `AKIA[0-9A-Z]{16}`, `ghp_`, `github_pat_`, `xox[baprs]-`, `sk-[A-Za-z0-9]{20,}`, `-----BEGIN .* PRIVATE KEY-----`, `password\s*[:=]`, `secret`, `token`, `api[_-]?key`, and `Authorization:`.
- **Personal data:** real email addresses, phone numbers, and internal hostnames or URLs that should not be public. Flag any email address embedded in code (for example a notification endpoint) and let the user decide whether to keep it, replace it with a placeholder, or exclude it.
- **Risky files:** `.env*`, `*.pem`, `*.key`, `*.pfx`, `id_rsa*`, `credentials*`, `*.sqlite`, `*.log`, and stray files such as `New Text Document.txt` that were never reviewed. Read any unknown file before including it.
- **Web-specific:** for client-side apps, check for inline scripts that send data to third-party hosts and list every external URL found.
- **Git history:** if a repo already exists, search the history for the same patterns (`git log -p -S<pattern>`) so a secret removed from the working tree is not still in a past commit.

Report findings as a short table (file, line, issue, severity). If anything is High severity, or if there is any doubt, **stop and ask the user** before continuing. Do not push until the user has approved the findings. Add anything that must stay local to `.gitignore`.

## 2. Prepare the repository

- If `git` is not on PATH, try `C:\Program Files\Git\cmd\git.exe` on Windows.
- If this folder is not a git repo, run `git init -b <branch>`.
- Set the remote with `git remote add origin https://github.com/<owner>/<repo>.git`, or `git remote set-url origin ...` if it exists.
- Set a repo-local commit identity if none is configured, using the GitHub no-reply address `<owner>@users.noreply.github.com`. Never put the user's private email in commits.
- Stage only the files cleared in step 1, by name. Do not use `git add .` or `git add -A`.

## 3. README

Create or update `README.md` from the actual project. Read the code and `CLAUDE.md` first. Include only facts you verified: what the project is, how to run it, the config the user must change, any hard constraints, and the live Pages URL `https://<owner>.github.io/<repo>/`. If a README already exists, edit it in place and keep content the user wrote. Do not invent features, badges, or screenshots.

## 4. GitHub Pages deployment via Actions

Create or update `.github/workflows/pages.yml`:

- Trigger on push to the branch and on `workflow_dispatch`.
- Permissions: `contents: read`, `pages: write`, `id-token: write`.
- Stage only the files that should be public into a `_site` folder (for a single-page app, copy `index.html` and any assets, never `CLAUDE.md`, `.claude/`, or the workflow files).
- Use `actions/checkout@v4`, `actions/configure-pages@v5`, `actions/upload-pages-artifact@v3`, and `actions/deploy-pages@v4` with the `github-pages` environment.

## 5. Commit and push

Show the user the exact list of files and the commit message, and **wait for a clear yes** before pushing. Then run `git commit` and `git push -u origin <branch>`.

Pushing needs the user's GitHub sign-in. Never ask for, type, or store a password or token. If the push fails with a prompt or authentication error, tell the user to run the push command in their own terminal so the browser sign-in can open, then re-check with `git ls-remote origin` and `git status -sb`.

## 5b. Repo About

Set the About description (from the arguments, or a one-line summary you propose) and topics.

- If the GitHub CLI is available and the user is signed in: `gh repo edit <owner>/<repo> --description "<text>" --add-topic <topic>` and `--homepage https://<owner>.github.io/<repo>/`.
- Otherwise, tell the user to open the repo page, click the gear icon beside **About**, and paste the description, website, and topics. Give them the exact text.

## 6. Enable Pages and verify

Pages cannot be enabled by the workflow's default token. Ask the user to open `https://github.com/<owner>/<repo>/settings/pages` and set **Build and deployment → Source** to **GitHub Actions**. Do not click the suggested "Configure" workflow buttons, which would add a duplicate workflow. Then have them re-run the failed run or use **Run workflow** at `https://github.com/<owner>/<repo>/actions/workflows/pages.yml`.

Finish by reporting: the security-scan result, the commit that was pushed, the README and About status, the workflow status, and the Pages URL `https://<owner>.github.io/<repo>/`. State plainly anything you could not do yourself or did not verify.
