# GitHub Actions Release Workflow

This document describes how the automated release workflow updates Homebrew formulas when new versions are released.

## Overview

When a new tag (e.g., `v0.2.0`) is pushed to any tool repository (grep, find, gawk, sed), the release workflow:

1. Builds binaries for all platforms (macOS arm64, Linux amd64/arm64)
2. Creates both pure and GNU-fallback variants
3. Creates GitHub releases with tarball artifacts
4. Automatically updates the Homebrew formula repositories

## Required GitHub Secret

To enable automatic formula updates, you must configure a **Personal Access Token (PAT)**:

### Setting up HOMEBREW_TAP_TOKEN

1. Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Generate a new token with the following scopes:
   - `repo` (full control of private repositories)
   - `workflow` (update workflow files)
3. Copy the generated token
4. Go to each tool repository (grep, find, gawk, sed) → Settings → Secrets and variables → Actions
5. Click "New repository secret"
6. Name: `HOMEBREW_TAP_TOKEN`
7. Value: Paste your PAT
8. Click "Add secret"

**Note**: The token must have write access to both `e-jerk/homebrew-utils` and `e-jerk/homebrew-utils-gnu` repositories.

## Workflow Behavior

### Individual Tool Formulas

When a release is created, the workflow:

- Downloads release tarballs for all platforms
- Calculates SHA256 hashes
- Updates the formula version and hashes in:
  - `e-jerk/homebrew-utils/Formula/{tool}.rb` (pure version)
  - `e-jerk/homebrew-utils-gnu/Formula/{tool}.rb` (GNU-fallback version)
- Commits and pushes changes to both repositories

### Formula Update Process

```
Tag pushed to e-jerk/grep
    ↓
Release workflow triggered
    ↓
Build binaries (pure + GNU variants)
    ↓
Create GitHub release with artifacts
    ↓
Calculate SHA256 hashes
    ↓
Update homebrew-utils/Formula/grep.rb
Update homebrew-utils-gnu/Formula/grep.rb
    ↓
Commit and push to both taps
```

## Manual Formula Updates

If you need to manually update formulas (e.g., if automation fails):

```bash
# Clone the tap repository
git clone https://github.com/e-jerk/homebrew-utils
cd homebrew-utils

# Download the release tarball
curl -L -o tool.tar.gz https://github.com/e-jerk/grep/releases/download/v0.2.0/grep-macos-arm64-v0.2.0.tar.gz

# Calculate SHA256
sha256sum tool.tar.gz

# Update the formula
# Edit Formula/grep.rb to update version and sha256 values

# Commit and push
git add Formula/grep.rb
git commit -m "Update grep to v0.2.0"
git push origin main
```

## Troubleshooting

### "Resource not accessible by personal access token" Error

- Verify the PAT has `repo` scope
- Ensure the token owner has write access to homebrew-utils and homebrew-utils-gnu repos
- Check that the secret name matches exactly: `HOMEBREW_TAP_TOKEN`

### Formulas Not Updating

- Check the Actions logs in the tool repository
- Verify the release artifacts exist
- Ensure the Formula file exists in the tap repository
- Check that the tool name matches the formula filename

### Wrong SHA256 Values

The workflow handles both:
- Placeholder values: `PLACEHOLDER_SHA256_MACOS_ARM64`
- Existing hashes: `sha256 "abc123..." # macos-arm64`

Both will be replaced with the correct values from the release artifacts.
