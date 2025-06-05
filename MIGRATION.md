# How to Update Your E2E Workflow

Here's how you can replace the 3-step process in your original workflow with the new `replace-comment` action.

## Before (your current workflow)

```yaml
- name: Find existing E2E results comment
  uses: peter-evans/find-comment@v3
  id: find-comment
  with:
    token: ${{ secrets.PRICERA_GITHUB_TOKEN }}
    repository: ${{ github.event.client_payload.repo-name || inputs.repo-name }}
    issue-number: ${{ inputs.pr-number || github.event.client_payload.pr-number }}
    comment-author: 'pricera'
    body-includes: '<!-- e2e-test-results -->'

- name: Delete existing E2E results comment
  if: steps.find-comment.outputs.comment-id
  run: |
    curl -L \
      -X DELETE \
      -H "Accept: application/vnd.github+json" \
      -H "Authorization: Bearer ${{ secrets.PRICERA_GITHUB_TOKEN }}" \
      -H "X-GitHub-Api-Version: 2022-11-28" \
      "https://api.github.com/repos/${{ github.event.client_payload.repo-name || inputs.repo-name }}/issues/comments/${{ steps.find-comment.outputs.comment-id }}"

- name: Comment PR with E2E Results
  uses: peter-evans/create-or-update-comment@v4
  with:
    token: ${{ secrets.PRICERA_GITHUB_TOKEN }}
    repository: ${{ github.event.client_payload.repo-name || inputs.repo-name }}
    issue-number: ${{ inputs.pr-number || github.event.client_payload.pr-number }}
    body: |
      <!-- e2e-test-results -->
      ## ${{ needs.run-tests.result == 'success' && '✅ E2E Tests Passed' || '❌ E2E Tests Failed' }}
      # ... rest of your content
```

## After (using replace-comment)

```yaml
- name: Comment PR with E2E Results
  uses: johanwulf/replace-comment@v1
  with:
    token: ${{ secrets.PRICERA_GITHUB_TOKEN }}
    repository: ${{ github.event.client_payload.repo-name || inputs.repo-name }}
    issue-number: ${{ inputs.pr-number || github.event.client_payload.pr-number }}
    comment-author: 'pricera'
    body-includes: '<!-- e2e-test-results -->'
    body: |
      <!-- e2e-test-results -->
      ## ${{ needs.run-tests.result == 'success' && '✅ E2E Tests Passed' || '❌ E2E Tests Failed' }}

      **Testing Branch:** `${{ github.event.client_payload.branch-name || inputs.branch-name || 'N/A' }}`
      **Workflow Run:** [View Details](${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }})

      ---

      <details>
      <summary>🎯 Microfrontend Versions</summary>
      ${{ needs.prepare-frontend.outputs.frontend-info || 'No version info available' }}
      </details>

      <details>
      <summary>🔧 Backend Versions</summary>
      ${{ needs.prepare-backend.outputs.backend-info || 'No version info available' }}
      </details>

      ---
      ### 🔧 Test Results
      #
      ${{ needs.run-tests.result == 'success' && '- **Status:** All tests passed' || needs.process-results.outputs.failed-tests }}
```

## Benefits

- ✅ **3 steps → 1 step** - Much cleaner workflow
- ✅ **Comments always at bottom** - Easy to find latest results  
- ✅ **Reuses Peter Evans' proven actions** - No reinventing the wheel
- ✅ **Same functionality** - Works exactly like your current setup
- ✅ **Better maintenance** - Centralized logic in one action

The comment will now appear at the bottom of the PR conversation every time, making it much easier to find the latest test results!
