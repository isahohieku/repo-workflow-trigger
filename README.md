# Triggering One GitHub Actions Workflow from Another

This repository demonstrates how to configure and trigger one GitHub Actions workflow from another using the repository_dispatch or workflow_dispatch events. This can be useful for creating modular workflows or triggering dependent processes across repositories.

## Features

- **Trigger Workflows**: Automatically initiate another workflow upon the completion of a source workflow.
- **Event-Driven Design**: Utilizes workflow_dispatch or repository_dispatch for flexible triggering.
- **Cross-Repository Support**: Trigger workflows in the same or different repositories.

## Workflow Setup

### 1. Triggered Workflow

This is the workflow you want to run after being triggered by the source workflow.

Example file: `.github/workflows/triggered-workflow.yml`

```yaml
name: Triggered Workflow

on:
  workflow_dispatch:
  repository_dispatch:
    types:
      - custom-event

jobs:
  triggered-job:
    runs-on: ubuntu-latest
    steps:
      - name: Run Triggered Workflow
        run: echo "This is the triggered workflow!"
```

- `workflow_dispatch`: Allows manual or programmatic triggering.
- `repository_dispatch`: Listens for custom events, such as custom-event.

### 2. Source Workflow

This is the workflow that triggers the second workflow.

Example file: `.github/workflows/source-workflow.yml`

```yaml
name: Source Workflow

on:
  push:
    branches:
      - main

jobs:
  trigger-other:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger Another Workflow
        env:
          GITHUB_TOKEN: ${{ secrets.PAT_TOKEN }}
        run: |
          curl -X POST \
            -H "Accept: application/vnd.github+json" \
            -H "Authorization: token $GITHUB_TOKEN" \
            https://api.github.com/repos/<owner>/<repo>/dispatches \
            -d '{"event_type":"custom-event"}'
```

Replace `<owner>` and `<repo>` with the owner and repository name of the triggered workflow.

### Configuration

#### Personal Access Token (PAT)

1. Generate a Personal Access Token (PAT):
   - Go to GitHub Settings -> Developer settings -> Personal Access Tokens.
   - Create a new token with repo and workflow scopes.
2. Save the PAT as a secret in your repository:
   - Go to your repository -> Settings -> Secrets and Variables -> Actions.
   - Add a new secret (e.g., PAT_TOKEN).

### How It Works

1. **Trigger Source Workflow**:
   - The source workflow runs on a push event (or any specified trigger).
2. **Send API Request**:
   - The source workflow sends a repository_dispatch or workflow_dispatch event to the target repository using curl.
3. **Start Triggered Workflow**:
   - The target workflow listens for the event and starts automatically.

### Notes

- **Cross-Repository Triggering**: Ensure the PAT has permissions for the target repository if triggering across repos.
- **GITHUB_TOKEN Limitation**: The built-in GITHUB_TOKEN can only trigger workflows in the same repository.

### References

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub REST API - Repository Dispatch](https://docs.github.com/en/rest/repos/repos#create-a-repository-dispatch-event)
- [GitHub REST API - Workflow Dispatch](https://docs.github.com/en/rest/actions/workflows#create-a-workflow-dispatch-event)

Feel free to customize the README to match your specific project or use case!
