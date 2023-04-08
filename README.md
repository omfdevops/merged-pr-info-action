# merged-pr-info-action

A composite GitHub Action to get information about a pull request merged into the default branch.

## Inputs
Please look into [action.yml](action.yml) to see the inputs.

- `github-token`: The GitHub token to access the GitHub API.

## Outputs
- `pull-request-number`: The number of the pull request.
- `pull-request-creator`: The creator of the pull request.

## Use cases

### Comment to the merged pull request if all jobs are passed
The examples show how to comment to the merged pull request if all jobs are passed.
We assume that the job1 and job2 are the jobs that are executed when the pull request is merged into the default branch.
Those can be something like to deploy a product.

```yaml
name: Comment to the merged pull request
on:
  push:
    branches:
      - main
jobs:
  job1:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Hello, world!"
  job2:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Hello, world!"

  comment:
    needs: [job1, job2]
    permissions:
      contents: read
      pull-requests: write
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: ubie-inc/merged-pr-info-action@v1
        id: merged-pr-info
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
      - name: "Comment to the merged pull request"
        uses: actions/github-script@v6
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          debug: true
          script: |
            const pr_number = "${{ steps.merged-pr-info.outputs.pull_request_number }}";
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: pr_number,
              body: "@" + "${{ steps.merged-pr-info.outputs.pull_request_creator }}" +  " The jobs are passed!"
            });
            console.log(`Commented to the pull request ${pr_number}`);
```
