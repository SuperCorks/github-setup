# Repository Setup Guide

Use this guide to quickly configure a new repository with `main` and `develop` branches (setting `develop` as default) and standard branch protection using GitHub Rulesets via the GitHub CLI (`gh`).

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

## 2. Set Default Branch

Set `develop` as the default branch for the repository.

```powershell
gh repo edit $REPO --default-branch develop
```

## 3. Configure Pull Request Settings

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

## 4. Configure Branch Protection (Rulesets)

Protect `main` and `develop` to prevent force pushes and deletions, and require pull requests using GitHub Rulesets.

Two configuration files are included in this repository:
- `ruleset-main.json`: Protects the `main` branch.
- `ruleset-develop.json`: Protects the `develop` branch.

Both files use the following configuration structure:

```json
{
  "name": "Branch Protection",
  "target": "branch",
  "enforcement": "active",
  "bypass_actors": [
    {
      "actor_id": 5,
      "actor_type": "RepositoryRole",
      "bypass_mode": "always"
    }
  ],
  "conditions": {
    "ref_name": {
      "include": [
        "refs/heads/YOUR_BRANCH_NAME"
      ],
      "exclude": []
    }
  },
  "rules": [
    {
      "type": "deletion"
    },
    {
      "type": "non_fast_forward"
    },
    {
      "type": "pull_request",
      "parameters": {
        "required_approving_review_count": 1,
        "dismiss_stale_reviews_on_push": false,
        "require_code_owner_review": false,
        "require_last_push_approval": false,
        "required_review_thread_resolution": false
      }
    }
  ]
}
```

The `bypass_actors` section allows repository admins (role ID 5) to bypass the branch protection rules.

Apply these rulesets to the repository:

```powershell
# Apply main protection
gh api -X POST "repos/$REPO/rulesets" --input ruleset-main.json

# Apply develop protection
gh api -X POST "repos/$REPO/rulesets" --input ruleset-develop.json
```
