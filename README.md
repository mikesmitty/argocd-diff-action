# ArgoCD Diff GitHub Action

This action generates a diff between the PR and the state of the cluster for any Applications that are sourced from the repo that the action is running in.

**Note**: This includes any changes between the head branch (i.e., the feature branch) and the base branch (i.e., the trunk, e.g., `main`), as well as ways in which the cluster is out-of-sync. Essentially, any diff from an Application that is sourced from the repo.

> This is forked from [`quizlet/argocd-diff-action`](https://github.com/quizlet/argocd-diff-action) which has not had a change since [2021-12-14](https://github.com/quizlet/argocd-diff-action/commit/4267297e35307515075a87fc65310bb6fbdba6df).

## Usage

Example GH action:
```yaml
# .github/workflows/checks.yml
name: Checks

on:
  pull_request:
    branches: [master, main]

jobs:
  argocd-diff:
    name: Generate ArgoCD Diff
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: argocd-diff-action/argocd-diff-action@v0
        with:
          argocd-server-fqdn: argocd.example.com
          argocd-token: ${{ secrets.ARGOCD_TOKEN }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          argocd-extra-cli-args: --grpc-web
          argocd-exclude-paths: "path/to/exclude,"
```

<!-- Action inputs and outputs are auto-generated by the release workflow. -->
### Inputs
<!--(inputs-start)-->

| Name  | Required | Default | Description |
| :---: | :------: | :-----: | ----------- |
| `argocd-server-fqdn` | true |  | ArgoCD server FQDN (i.e., without the protocol) |
| `argocd-token` | true |  | ArgoCD token for a local or project-scoped user https://argoproj.github.io/argo-cd/operator-manual/user-management/#local-usersaccounts-v15 |
| `argocd-version` | false |  | `argocd` command version to install. Defaults to the server version. |
| `github-token` | true |  | Github Token |
| `argocd-extra-cli-args` | false | --grpc-web | Extra arguments to pass to the argocd CLI |
| `argocd-exclude-paths` | false |  | ArgoCD app apths to exclude in comma separated list |

<!--(inputs-end)-->

### Outputs
<!--(outputs-start)-->

| Name  | Description |
| :---: | ----------- |
|  |

<!--(outputs-end)-->

## Compatibility
- Requires `argocd-version >= v2.1.0` to support use of [the `--exit-code=false` option on `app diff`](https://github.com/argoproj/argo-cd/commit/2faa08e710b6da3fdfa88eb1491de0648d004a19).

### Migrating from [the `quizlet` Fork](https://github.com/quizlet/argocd-diff-action)
- The `argocd-server-url` input has been refactored to `argocd-server-fqdn`.

## How it works

1. Downloads the ArgoCD binary, makes it executable and authenticates the Server.
2. Fetches all of the Applications from the ArgoCD API using the `argocd-token`.
3. Filters the Applications to the ones that are sourced from the current repo (using the context of the action), targeting the trunk of the repo, and are not in an excluded path.
4. Runs `argocd app diff` for each Application.
5. If in the Application diff there is an Application with a change to its `targetRevision`, get the diff for it using `--revision`.
    - Note that this won't include any other changes to the App of App (e.g., Helm value changes).
6. Posts the diff output as a comment on the PR (updating the same comment if it already exists).

## Releases & Publishing
Releases are automated using `semantic-release` via [the `release.yml` workflow](https://github.com/argocd-diff-action/argocd-diff-action/blob/master/.github/workflows/release.yml) and [the `.releaserc` config file](https://github.com/argocd-diff-action/argocd-diff-action/blob/master/.releaserc). 

Each release will create a semver tag (e.g., `0.2.0`) and update the floating major version tag (e.g., `v0`) associated with it.

The action is in early development (hence [the `0` major version](https://github.com/argocd-diff-action/argocd-diff-action/releases/tag/v0)), which means there may be breaking changes introduced at any time. It's suggested to pin to the semver release tag until the release of `v1`.

## Contributing
All commits must conform to the Angular commit convention and be done through a PR. Given this, the use of `Rebase and merge` is preferred to capture each semantic commit in a PR in the changelog.

Check [`commit-analyzer` plugin configuration](https://github.com/argocd-diff-action/argocd-diff-action/blob/master/.releaserc#L6) to see which types cause what releases. 

### Local Development
Use Node 16.

Install the dependencies without updating the lock file.
```
$ npm ci
```

Running tests locally:
```
$ npm run test
```
> Note these are unit tests that, currently, cover a small porition of the code base. This will also run on every PR.


## Creating the `release` workflow PAT
- Created `morey-tech-bot` GitHub account to act as a service account.
- Invited the user a as member of the `argocd-diff-action` org.
- From the `morey-tech-bot` account,
  - https://github.com/settings/tokens/new
    - Forced to set Expiry on fine-grained tokens. Using classic instead.
  - Set Note to `argocd-diff-action`
  - Set Expiration to `No expiration`
  - Select the `repo` scope.
  - Click Generate Token
- Invite `morey-tech-bot` to repo as Maintain role.
- As a repo owner user, create the Action secret `GH_MOREY_TECH_BOT_TOKEN` and paste the token from the bot account.
  - https://github.com/argocd-diff-action/argocd-diff-action/settings/secrets/actions/new
  - Set Name
  - Paste in Secret
  - Click Add secret
