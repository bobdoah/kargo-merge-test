# Kargo Merge Methods Example

This example demonstrates end-to-end testing of the `mergeMethod` configuration option
in the `git-merge-pr` promotion step. It showcases three different Git merge strategies
that can be used when automatically merging pull requests during Kargo promotions.

## Merge Methods

### 1. Merge Commit (`merge`)
- Creates a merge commit that combines all commits from the source branch
- Preserves the complete commit history
- Results in a non-linear history with merge commits
- Files: `kargo/stages-merge-commit.yaml`

### 2. Squash Merge (`squash`)
- Combines all commits from the source branch into a single commit
- Creates a cleaner, more condensed history
- Ideal for feature branches with many small commits
- Files: `kargo/stages-squash.yaml`

### 3. Rebase Merge (`rebase`)
- Re-applies commits on top of the target branch
- Maintains a linear commit history
- No merge commits are created
- Files: `kargo/stages-rebase.yaml`

## Directory Structure

```
merge-methods/
├── README.md
├── argocd/
│   ├── appproject.yaml      # ArgoCD project definition
│   └── applications.yaml    # ArgoCD applications for all stages
├── base/
│   ├── deployment.yaml      # Base deployment manifest
│   ├── kustomization.yaml   # Base kustomization
│   └── service.yaml         # Base service manifest
├── env/
│   ├── dev/
│   │   └── kustomization.yaml
│   ├── staging/
│   │   └── kustomization.yaml
│   └── prod/
│       └── kustomization.yaml
└── kargo/
    ├── project.yaml              # Kargo project definition
    ├── warehouse.yaml            # Freight source configuration
    ├── stages-merge-commit.yaml  # Stages using merge commit strategy
    ├── stages-squash.yaml        # Stages using squash merge strategy
    └── stages-rebase.yaml        # Stages using rebase merge strategy
```

## Prerequisites

- Kubernetes cluster with Kargo installed
- Argo CD installed and configured
- GitHub repository with appropriate permissions for PR operations
- Git credentials configured in Kargo

## Setup

1. **Create the Kargo project namespace:**
   ```bash
   kubectl create namespace kargo-merge-methods
   ```

2. **Apply ArgoCD resources:**
   ```bash
   kubectl apply -f argocd/appproject.yaml
   kubectl apply -f argocd/applications.yaml
   ```

3. **Apply Kargo resources:**
   ```bash
   kubectl apply -f kargo/project.yaml
   kubectl apply -f kargo/warehouse.yaml
   ```

4. **Choose and apply your preferred merge strategy:**
   ```bash
   # For merge commit strategy
   kubectl apply -f kargo/stages-merge-commit.yaml

   # For squash merge strategy
   kubectl apply -f kargo/stages-squash.yaml

   # For rebase merge strategy
   kubectl apply -f kargo/stages-rebase.yaml
   ```

## Testing the Merge Methods

Each stage pipeline follows this workflow:

1. **git-clone** - Clones the source and target branches
2. **git-clear** - Clears the output directory
3. **kustomize-set-image** - Updates the image tag from Freight
4. **kustomize-build** - Builds the Kustomize manifests
5. **git-commit** - Commits the changes
6. **git-push** - Pushes to a new branch
7. **git-open-pr** - Opens a pull request
8. **git-merge-pr** - Merges the PR using the specified `mergeMethod`
9. **argocd-update** - Triggers Argo CD sync

### Verifying Merge Method Behavior

After a promotion completes, examine the Git history to verify the merge method:

```bash
# Check the commit history on the target branch
git log --oneline --graph env/dev

# For merge commit: You'll see merge commits like "Merge pull request #X"
# For squash: You'll see a single commit per promotion
# For rebase: You'll see a linear history without merge commits
```

## Configuration Reference

The `mergeMethod` option in the `git-merge-pr` step accepts:

| Value    | Description                                      |
|----------|--------------------------------------------------|
| `merge`  | Create a merge commit (default)                  |
| `squash` | Squash all commits into one before merging       |
| `rebase` | Rebase commits onto the target branch            |

Example configuration:
```yaml
- uses: git-merge-pr
  config:
    repoURL: ${{ vars.gitRepo }}
    prNumber: ${{ outputs['open-pr'].prNumber }}
    mergeMethod: squash  # or 'merge' or 'rebase'
    wait: true
```

## GitHub Repository Requirements

Ensure your GitHub repository has the appropriate merge methods enabled:

1. Go to **Settings** > **General** > **Pull Requests**
2. Enable the merge methods you want to use:
   - [x] Allow merge commits
   - [x] Allow squash merging
   - [x] Allow rebase merging

## Notes

- The `wait: true` option ensures Kargo waits for the PR to be mergeable
  before attempting to merge (e.g., waiting for required checks to pass)
- Different merge methods may have different requirements based on branch
  protection rules
- Rebase merge may fail if there are conflicts that cannot be automatically resolved
