# Repository Setup Guide

Use this guide to quickly configure a new repository with `main` and `develop` branches and standard branch protection rules using the GitHub CLI (`gh`).

## Prerequisites

- [GitHub CLI](https://cli.github.com/) installed (`gh`).
- Authenticated via `gh auth login`.
- Powershell or a compatible shell.

## 1. Setup Branches

Ensure both `main` and `develop` branches exist and are pushed to the remote.

```powershell
# Initialize git if needed
git init

# Create and push main
git checkout -b main
git commit --allow-empty -m "root commit"
git push -u origin main

# Create and push develop
git checkout -b develop
git push -u origin develop
```

## 2. Configure Pull Request Settings

Apply standard merge settings:
- **Enable**: Squash merging (Default to PR Title)
- **Disable**: Merge commits, Rebase merging
- **Enable**: Auto-delete head branches

Run the following command (replace `owner/repo-name` with your repository):

```powershell
$REPO = "owner/repo-name" 

gh api -X PATCH "repos/$REPO" `
  -f allow_squash_merge=true `
  -f allow_merge_commit=false `
  -f allow_rebase_merge=false `
  -f delete_branch_on_merge=true `
  -f squash_merge_commit_title="COMMIT_OR_PR_TITLE" `
  -f squash_merge_commit_message="COMMIT_MESSAGES"
```

## 3. Configure Branch Protection

Protect `main` and `develop` to prevent force pushes and deletions.

A `protection-settings.json` file is included in this repository with the following configuration:

```json
{
  "required_status_checks": null,
  "enforce_admins": null,
  "required_pull_request_reviews": {
    "dismiss_stale_reviews": false,
    "require_code_owner_reviews": false,
    "required_approving_review_count": 1
  },
  "restrictions": null,
  "allow_force_pushes": false,
  "allow_deletions": false,
  "block_creations": false,
  "required_conversation_resolution": false,
  "lock_branch": false,
  "allow_fork_syncing": false,
  "required_linear_history": false
}
```

Apply these settings to both branches using the included file:

```powershell
# Apply to main
gh api -X PUT "repos/$REPO/branches/main/protection" --input protection-settings.json

# Apply to develop (if you want it protected)
gh api -X PUT "repos/$REPO/branches/develop/protection" --input protection-settings.json
```
