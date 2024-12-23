It would be nice if GitHub pull requests (PRs) could be created in draft mode by default.
If this was the case, when the PR was ready for review and converted to "regular" mode, more elaborate tests and automations could be triggered off of the `ready_for_review` event.

This flow would allow editing and rebasing of the pull request while it was in draft mode.
Once out of draft mode and the `ready_for_review` tests have passed, the formal review involving a human can be requested with the knowledge that some degree of reworking, testing, and verification have been performed.
This reduces "review churn", saves computing resources, and saves everyone time.

In the absense of such functionality natively, here is a GitHub Actions workflow that converts any new pull request into a draft automatically.
Re-opened pull requests are not converted, neither is it converted when there are new Git commits pushed to it ("synchronize" events).

```yaml
name: Draft pull requests

on:
  pull_request:
    types: [opened]

jobs:
  convert:
    name: Convert to draft
    if: github.event.pull_request.draft == false
    runs-on: ubuntu-latest
    steps:
      - name: Repository checkout
        uses: actions/checkout@v3
      - name: Convert to draft
        env:
           GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
           PR_NUMBER: ${{ github.event.number }}
        run: |
          gh pr ready "$PR_NUMBER" --undo
```

## References

* [GitHub contexts][github-context]
* [GitHub events][github-events] 
* [Using the GitHub CLI in workflows][gh-in-workflows]

[github-context]: https://docs.github.com/en/actions/learn-github-actions/contexts#github-context
[github-events]: https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#pull_request
[gh-in-workflows]: https://docs.github.com/en/actions/using-workflows/using-github-cli-in-workflows
