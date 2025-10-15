# Testing Instructions for only-issue-types Fix

## Quick Start: GitHub Actions Testing (Recommended)

### Step 1: Create Test Workflow

Add this to your repository at `.github/workflows/test-stale-fix.yml`:

```yaml
name: Test Stale Fix

on:
  workflow_dispatch:

jobs:
  test-stale:
    runs-on: ubuntu-latest
    permissions:
      issues: write
      pull-requests: write
    
    steps:
      - uses: Bibo-Joshi/stale@copilot/analyze-only-issue-types-flaw
        with:
          only-issue-types: '❔ question'
          days-before-stale: 365
          days-before-close: 365
          stale-issue-message: 'TEST: This issue may be stale'
          stale-issue-label: 'test-stale'
          operations-per-run: 30
          debug-only: true  # Important: prevents actual changes
```

### Step 2: Run and Review

1. Go to Actions → "Test Stale Fix" → "Run workflow"
2. Check the logs for verbose output showing type matching

## What You'll See in Logs

### ✅ Matching Type (Will Process)
```
Option only-issue-types is set. Allowed types: ['❔ question']
Issue has issue_type: '❔ question' (normalized: '❔ question')
  Comparing normalized type '❔ question' === '❔ question': true
Continuing to process this issue because its type matches
```

### ❌ Non-Matching Type (Will Skip)
```
Option only-issue-types is set. Allowed types: ['❔ question']
Issue has issue_type: 'bug' (normalized: 'bug')
  Comparing normalized type 'bug' === '❔ question': false
Skipping this issue because its type ('bug') is not in onlyIssueTypes
```

### 🔧 Whitespace Handling (Fixed)
```
Option only-issue-types is set. Allowed types: ['question']
Issue has issue_type: ' question ' (normalized: 'question')
  Comparing normalized type 'question' === 'question': true
Continuing to process this issue because its type matches
```

## Local Testing Alternative

```bash
git clone https://github.com/Bibo-Joshi/stale.git
cd stale
git checkout copilot/analyze-only-issue-types-flaw
npm install
npm run build

# Set environment variables
export GITHUB_TOKEN=your_token
export GITHUB_REPOSITORY=python-telegram-bot/python-telegram-bot
export INPUT_ONLY-ISSUE-TYPES='❔ question'
export INPUT_DEBUG-ONLY=true

node dist/index.js 2>&1 | tee test.log
grep -A 5 "Allowed types:" test.log
```

## Expected Behavior

- ✅ Exact type match → Process
- ✅ Type with whitespace trimmed → Process  
- ✅ Different type → Skip
- ✅ No type (undefined) → Skip
- ✅ Case-insensitive matching
- ✅ Emojis preserved
