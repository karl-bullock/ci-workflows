# ci-workflows

Shared CI checks for the Link Robins, BlizzNux and karl-bullock repos:

- **ShellCheck:** only NEW warnings fail, compared with the base commit.
- **Gitleaks:** secrets in the commits being added.
- **Psalm taint analysis** (Flarum extensions, `v2` and later): follows user input to SQL, HTML and shell sinks in the extension's PHP. CodeQL does not read PHP, so this is the backend's security scan.

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

To change a check for every repo at once, move that major tag (`v1`, `v2`) to the new commit. A new kind of check gets a new major tag, so existing callers never change underneath.

## Psalm taint analysis

For a Flarum extension with backend PHP. Add `.github/workflows/psalm.yml`:

```yaml
name: Psalm
on:
  pull_request:
  push:
    branches: [main]
permissions:
  contents: read
  security-events: write
jobs:
  taint:
    uses: karl-bullock/ci-workflows/.github/workflows/psalm.yml@v2
```

It scans `src/` and `extend.php`; the repo needs no Psalm config. Out of the box Psalm only treats `$_GET` and friends as user input, which Flarum never reads, so the workflow adds a stub that marks PSR-7 request input and the JSON:API context as sources and Laravel's raw-SQL methods as sinks. Without it a scan finds nothing and looks clean.

It is report-only for now: findings appear on the repo's code scanning page under the `psalm-taint` category, next to CodeQL's, and never fail the build. A Psalm crash does fail it, so a broken scan cannot pass for a clean one.
