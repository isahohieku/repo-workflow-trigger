# GitHub Workflow Trigger Example

This repository demonstrates how to trigger one GitHub Actions workflow (`triggered-workflow.yml`) from another workflow (`trigger-workflow.yml`) using the GitHub REST API.

---

## Workflows Overview

### 1. **Source Workflow**: `trigger-workflow.yml`

This workflow is triggered when there is a **push** event on the `trigger` branch. It uses the GitHub REST API to dispatch another workflow (`triggered-workflow.yml`) in the target repository.

#### **Trigger Condition**:

- **Event**: `push`
- **Branch**: `trigger`

#### **Action**:

It sends a POST request to the GitHub API using `curl` to trigger the **Triggered Workflow**.

#### **Workflow Code**:

```yaml
name: Source Workflow

on:
  push:
    branches:
      - trigger

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
            -H "X-GitHub-Api-Version: 2022-11-28" \
            https://api.github.com/repos/isahohieku/repo-trigger/actions/workflows/triggered-workflow.yml/dispatches \
            -d '{"ref":"main", "inputs":{"message":"Hello from the source workflow!"}}'
```

### 2. **Triggered Workflow**: `triggered-workflow.yml`

This workflow is triggered using the `workflow_dispatch` event. It accepts an input `message` and prints it in the workflow logs.

#### **Trigger Condition**:

- **Event**: `workflow_dispatch`
- **Inputs**:
  - `message`: A custom message passed from the **Source Workflow**.

#### **Workflow Code**:

```yaml
name: Triggered Workflow

on:
  workflow_dispatch:
    inputs:
      message:
        description: "Message to display"
        required: true

jobs:
  triggered-job:
    runs-on: ubuntu-latest
    steps:
      - name: Run Triggered Workflow
        run: |
          echo "This is the triggered workflow!"
          echo 'This is the message: "${{ inputs.message }}"'
```

### **Workflow Behavior**

1. **Triggering the Source Workflow**:

   - Push a change to the `trigger` branch.
   - This will run the `trigger-workflow.yml`.

2. **Triggering the Target Workflow**:

   - `trigger-workflow.yml` sends a dispatch request to the GitHub API to trigger `triggered-workflow.yml` on the `main` branch.
   - The input `message` is passed as `"Hello from the source workflow!"`.

3. **Running the Triggered Workflow**:
   - `triggered-workflow.yml` receives the dispatch event and prints the input `message` to the workflow logs.

### **Prerequisites**

1. **Personal Access Token (PAT)**:

   - A GitHub Personal Access Token (PAT) with the `repo` and `workflow` permissions is required.
   - Store this token in your repository secrets as `PAT_TOKEN`.

2. **Repository Setup**:

   - Ensure that the `triggered-workflow.yml` file exists in the repository (`isahohieku/repo-trigger`).

3. **Branch Setup**:
   - The `trigger-workflow.yml` workflow runs on the `trigger` branch. Ensure this branch exists.

### **Example Output**

After successfully running the workflows:

- The `Source Workflow` logs:

```
Triggering the Triggered Workflow...
```

- The `Triggered Workflow` logs:

```
This is the triggered workflow!
This is the message: "Hello from the source workflow!"
```

### **Use Case**

This setup is useful for:

- Chaining workflows across repositories.
- Passing dynamic inputs from one workflow to another.
- Modularizing CI/CD processes.
