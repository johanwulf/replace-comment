# replace-comment

A GitHub Action that finds, deletes, and recreates a comment to move it to the bottom of the conversation. This is perfect for CI/CD status updates, bot notifications, and any content you want to keep highly visible.

## Why Replace Instead of Update?

When you **update** a comment, it stays in its original chronological position. When you **delete and recreate**, the new comment appears at the bottom of the conversation - much more visible!

**Perfect for:**
- ✅ CI/CD test results
- ✅ Build status updates  
- ✅ Bot notifications
- ✅ Status reports
- ✅ Any content that should stay at the bottom for visibility

## Usage

### Basic Example

```yaml
- name: Replace E2E Test Results Comment
  uses: johanwulf/replace-comment@v1.0.0
  with:
    issue-number: ${{ github.event.pull_request.number }}
    body-includes: '<!-- e2e-test-results -->'
    comment-author: 'github-actions[bot]'
    body: |
      <!-- e2e-test-results -->
      ## 🧪 E2E Test Results
      
      ✅ All tests passed!
      
      - **Total tests:** 42
      - **Duration:** 3m 24s
      - **Browser:** Chrome 120
```

### Advanced Example with File Body

```yaml
- name: Replace Build Report
  uses: johanwulf/replace-comment@v1.0.0
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    repository: ${{ github.repository }}
    issue-number: ${{ github.event.pull_request.number }}
    body-regex: '<!-- build-report-\\d+ -->'
    comment-author: 'build-bot[bot]'
    body-path: './build-report.md'
    reactions: 'rocket,eyes'
```

## Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `token` | GITHUB_TOKEN or a repo scoped PAT | No | `${{ github.token }}` |
| `repository` | The full name of the repository | No | `${{ github.repository }}` |
| `issue-number` | The number of the issue or pull request | **Yes** | |
| `body-includes` | A string to search for in comment bodies | No* | |
| `body-regex` | A regular expression to search for in comment bodies | No* | |
| `comment-author` | The GitHub username of the comment author | No | |
| `direction` | Search direction: `first` or `last` | No | `first` |
| `nth` | 0-indexed number specifying which comment to replace if multiple found | No | `0` |
| `body` | The comment body content | No** | |
| `body-path` | Path to a file containing the comment body | No** | |
| `reactions` | Comma-separated reactions to add (+1, -1, laugh, confused, heart, hooray, rocket, eyes) | No | |

\* Either `body-includes` or `body-regex` must be provided  
\** Either `body` or `body-path` must be provided

## Outputs

| Name | Description |
|------|-------------|
| `comment-id` | The ID of the created comment |
| `comment-url` | The URL of the created comment |
| `comment-body` | The body of the created comment |
| `found-comment-id` | The ID of the comment that was found and deleted (empty if none found) |

## How It Works

1. **🔍 Find**: Uses [peter-evans/find-comment](https://github.com/peter-evans/find-comment) to search for existing comments
2. **🗑️ Delete**: Removes the found comment using GitHub's REST API (if any)
3. **✨ Create**: Uses [peter-evans/create-or-update-comment](https://github.com/peter-evans/create-or-update-comment) to create a new comment at the bottom

This action is a **composite action** that combines the excellent work of Peter Evans into a simple "find-delete-create" workflow.

## Examples

### CI/CD Test Results

```yaml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run tests
        id: tests
        run: |
          # Your test commands here
          echo "result=✅ All tests passed!" >> $GITHUB_OUTPUT
      
      - name: Update test results comment
        if: github.event_name == 'pull_request'
        uses: johanwulf/replace-comment@v1.0.0
        with:
          issue-number: ${{ github.event.pull_request.number }}
          body-includes: '<!-- test-results -->'
          comment-author: 'github-actions[bot]'
          body: |
            <!-- test-results -->
            ## 🧪 Test Results
            
            ${{ steps.tests.outputs.result }}
            
            **Commit:** ${{ github.sha }}
            **Workflow:** [${{ github.run_id }}](${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }})
```

### Deployment Status

```yaml
- name: Update deployment status
  uses: johanwulf/replace-comment@v1.0.0
  with:
    issue-number: ${{ github.event.pull_request.number }}
    body-regex: '<!-- deploy-status-.*? -->'
    body: |
      <!-- deploy-status-${{ github.run_id }} -->
      ## 🚀 Deployment Status
      
      **Environment:** staging
      **Status:** ${{ job.status }}
      **URL:** https://staging-${{ github.event.pull_request.number }}.example.com
      **Deploy time:** $(date)
    reactions: 'rocket'
```

### Build Artifacts Report

```yaml
- name: Generate build report
  run: |
    echo "## 📦 Build Artifacts" > build-report.md
    echo "" >> build-report.md
    echo "| File | Size |" >> build-report.md
    echo "|------|------|" >> build-report.md
    ls -la dist/ | awk '{print "| " $9 " | " $5 " |"}' >> build-report.md

- name: Post build report
  uses: johanwulf/replace-comment@v1.0.0
  with:
    issue-number: ${{ github.event.pull_request.number }}
    body-includes: '<!-- build-artifacts -->'
    body-path: './build-report.md'
```

## Tips

### Unique Comment Identification

Use HTML comments to uniquely identify your comments:

```yaml
body: |
  <!-- my-unique-identifier -->
  Your visible content here
```

### Multiple Bot Comments

Use different identifiers for different types of comments:

```yaml
# Test results
body-includes: '<!-- test-results -->'

# Build status  
body-includes: '<!-- build-status -->'

# Deploy status
body-includes: '<!-- deploy-status -->'
```

### Conditional Comments

Only create comments when needed:

```yaml
- name: Replace comment
  if: github.event_name == 'pull_request' && failure()
  uses: johanwulf/replace-comment@v1.0.0
  with:
    # ... your config
```

## Comparison with Other Actions

| Action | Find | Update | Delete+Create | Use Case |
|--------|------|--------|---------------|----------|
| `peter-evans/create-or-update-comment` | ✅ | ✅ | ❌ | Update existing content |
| `edumserrano/find-create-or-update-comment` | ✅ | ✅ | ❌ | Simplified find+update |
| `replace-comment` (this action) | ✅ | ❌ | ✅ | Keep comments at bottom |

## Development

This is a composite GitHub Action that leverages:
- [peter-evans/find-comment](https://github.com/peter-evans/find-comment) for finding comments
- [peter-evans/create-or-update-comment](https://github.com/peter-evans/create-or-update-comment) for creating comments
- GitHub REST API for deleting comments

No build step required! The action runs directly from the `action.yml` file.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Inspired by the excellent work of [Peter Evans](https://github.com/peter-evans) on GitHub Actions for comments
- Built to fill the gap for delete+create comment workflows
