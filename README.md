# ci-workflows

Shared CI checks for the Link Robins, BlizzNux and karl-bullock repos:

- **ShellCheck:** only NEW warnings fail, compared with the base commit.
- **Gitleaks:** secrets in the commits being added.

## Use it in a repo

Add `.github/workflows/shared-checks.yml`:

```yaml
name: Shared checks
on:
  pull_request:
  push:
    branches: [main]
jobs:
  shared:
    uses: karl-bullock/ci-workflows/.github/workflows/checks.yml@v1
    # Private karl-bullock repos run on their KVM4 runner:
    # with: { runs-on: '["self-hosted","kvm4"]' }
```

Public repos use GitHub's hosted runners, which are free for them. A private repo needs a self-hosted runner. Repos without one are still covered weekly by the dev sweep (devsweep-stack), which runs ShellCheck and Gitleaks over every repo in ~/dev.

To change a check for every repo at once, move the `v1` tag to the new commit.
