# replace-comment

GitHub Action that finds, deletes, and recreates comments to keep them at the bottom of PR conversations. Perfect for CI/CD status updates and bot notifications.

## Usage

```yaml
- name: Replace Comment
  uses: johanwulf/replace-comment@v1.0.0
  with:
    issue-number: ${{ github.event.pull_request.number }}
    body-includes: '<!-- test-results -->'
    body: |
      <!-- test-results -->
      ## Test Results
      ✅ All tests passed!
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

1. Find existing comment using search criteria
2. Delete the found comment (if any)
3. Create new comment at the bottom

## License

MIT License - see [LICENSE](LICENSE) file for details.
