# Preview Changes

Runs a dry-run command and posts the diff/plan as a pull request comment. Shows what changes would be applied (Helm diff for charts, Terraform plan for services).

## Inputs

| Input                | Description                           | Required | Default |
| -------------------- | ------------------------------------- | -------- | ------- |
| `version`            | Release version to preview            | Yes      | -       |
| `environment`        | Environment name                      | Yes      | -       |
| `installation`       | Installation name                     | Yes      | -       |
| `ryvn_client_id`     | Ryvn Client ID for authentication     | Yes      | -       |
| `ryvn_client_secret` | Ryvn Client Secret for authentication | Yes      | -       |
| `github_token`       | GitHub token for posting comments     | Yes      | -       |
| `timeout`            | Timeout for watching task             | No       | `"10m"` |

## Usage

### Basic Example

```yaml
- name: Preview Changes
  uses: ./.github/actions/preview-deployment
  with:
    version: "v1.2.3"
    environment: "staging"
    installation: "my-app"
    ryvn_client_id: ${{ secrets.RYVN_CLIENT_ID }}
    ryvn_client_secret: ${{ secrets.RYVN_CLIENT_SECRET }}
    github_token: ${{ secrets.GITHUB_TOKEN }}
```

### After Create Release

```yaml
on:
  push:
    tags:
      - "v*"

jobs:
  release-and-preview:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      contents: read

    steps:
      - uses: actions/checkout@v4

      # Your build/push steps here
      # ...

      # Create Ryvn release
      - name: Install Ryvn CLI
        uses: ryvn-technologies/install-ryvn-cli@v1.0.0

      - name: Create Release
        env:
          RYVN_CLIENT_ID: ${{ secrets.RYVN_CLIENT_ID }}
          RYVN_CLIENT_SECRET: ${{ secrets.RYVN_CLIENT_SECRET }}
        run: |
          VERSION=${{ github.ref_name }}
          ryvn create release my-service $VERSION

      # Preview changes
      - name: Preview Changes
        uses: ./.github/actions/preview-deployment
        with:
          version: ${{ github.ref_name }}
          environment: staging
          installation: my-app
          ryvn_client_id: ${{ secrets.RYVN_CLIENT_ID }}
          ryvn_client_secret: ${{ secrets.RYVN_CLIENT_SECRET }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

## Required Permissions

```yaml
permissions:
  pull-requests: write # Create comments
  contents: read # Access repository
```

## Output

Creates a new comment on the PR showing changes with colored diff output:

````
✅ Preview changes for installation `my-app` in environment `staging` using version `v1.2.3` (5s)

```ansi
observability, alloy, ClusterRole (rbac.authorization.k8s.io) has changed:
...
````

<sub>Preview for commit abc1234</sub>

```

## Notes

- A new comment is created each time the action runs
- Large outputs (>60KB) are automatically truncated with a link to the full output in the Ryvn Dashboard
- ANSI color codes are preserved in the diff output for better readability
```
