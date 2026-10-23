# 🕵🏻 Get Node Version

This repository implements a very simple GitHub composite action that pulls the Node.js version from the _package.json_ file of the project and ensures that it is used for the current pipeline execution. The action relies on the Node.js version specified within the [engines](https://docs.npmjs.com/cli/v10/configuring-npm/package-json#engines) section of the _package.json_, e.g:

```json
{
  "engines": {
    "node": "24.x.x",
  },
}
```

## Usage

### Why?

What you will often notice in Node.js projects is that you quickly get to the point where the correct version relevant to the project is defined and needs to be maintained in different places, e.g. different pipeline / workflow definitions. Ideally, however, you would like to maintain the Node.js version only in one place and thus at the same time ensure that all developers use the same version for their local development, which is also used by the CI/CD automations of the project. Exactly this is what this action enables you to do!

### How?

In a GitHub workflow, the use of the action after the initial checkout step would look like this:

```yaml
name: build

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

# Least-privilege token: this workflow only reads the repository.
permissions:
  contents: read

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v7.0.1
        with:
          persist-credentials: false # the GITHUB_TOKEN is not needed after the checkout - don't leave it in .git/config

      - name: Select Node Version
        uses: svierk/get-node-version@v1.6.0
```

Every `uses:` reference above is pinned to an exact release tag rather than a floating major tag, so adopting a new version is always an explicit, reviewable change of that pin.

### Inputs

| Name              | Required | Default | Description                                                                |
| ----------------- | -------- | ------- | -------------------------------------------------------------------------- |
| `node-version`    | no       | –       | Explicit Node.js version spec (e.g. `24.x`) that takes precedence over the _package.json_ lookup. An empty value falls back to `engines.node`. |
| `path`            | no       | `./`    | Path where the _package.json_ file can be found.                           |
| `package-manager` | no       | `auto`  | Package manager used for dependency caching. Supported: `auto`, `npm`, `yarn`, `pnpm`. See [Package manager and dependency cache](#package-manager-and-dependency-cache). |
| `step-summary`    | no       | `true`  | Write a result section to the GitHub Actions [job summary](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#adding-a-job-summary). Set to `false` to avoid collisions with a custom workflow summary. |

### Outputs

| Name           | Description                                                                          |
| -------------- | ------------------------------------------------------------------------------------ |
| `node-version` | The Node.js version spec resolved from the `node-version` input or the `engines.node` field. |
| `source`       | Where the version spec was resolved from: `node-version input` or `package.json`.    |
| `package-manager` | The package manager used for the dependency cache, either set explicitly or detected with `auto`. Falls back to `npm` if nothing is detected. |

The action fails with a clear error if no `node-version` input is provided and the _package.json_ or its `engines.node` field is missing, so a misconfiguration surfaces immediately instead of silently falling back to an undefined version.

### Package manager and dependency cache

By default (`package-manager: auto`), the action detects the package manager of the project and enables the matching [dependency cache](https://github.com/actions/setup-node#caching-global-packages-data) of `actions/setup-node`:

1. The [`packageManager`](https://nodejs.org/api/packages.html#packagemanager) field in the _package.json_, e.g. `"packageManager": "yarn@4.18.1"`.
2. Otherwise the lock file: _package-lock.json_ or _npm-shrinkwrap.json_ → `npm`, _yarn.lock_ → `yarn`, _pnpm-lock.yaml_ → `pnpm`.
3. If no lock file is found, Node.js is set up without a dependency cache and the action only emits a notice. This keeps projects that do not commit a lock file working.

For `yarn` and `pnpm`, the action runs `corepack enable` before setting up Node.js, so the version pinned in the `packageManager` field is used and `pnpm` does not have to be installed separately. A `pnpm` installed beforehand, e.g. via [`pnpm/action-setup`](https://github.com/pnpm/action-setup), keeps working as well. The resolved value is available as the `package-manager` output for subsequent steps:

```yaml
- name: Select Node Version
  id: node
  uses: svierk/get-node-version@v1.6.0

- name: Install dependencies
  run: ${{ steps.node.outputs.package-manager }} install
```

Setting `package-manager` explicitly to `npm`, `yarn` or `pnpm` skips the detection. In this case the matching lock file is required and the action fails if it is missing.

### Overriding the version via repository variable

The _package.json_ remains the recommended single source of truth, since it is the only place local developer tooling picks the version up from as well. However, sometimes you temporarily need to pin the pipeline to a fixed, known-good Node.js version - ideally without touching the workflow file at all. For this, wire the `node-version` input to a [repository variable](https://docs.github.com/en/actions/learn-github-actions/variables) (organization and environment variables work the same way):

```yaml
- name: Select Node Version
  uses: svierk/get-node-version@v1.6.0
  with:
    node-version: ${{ vars.NODE_VERSION_OVERRIDE }}
```

This wiring is safe to keep in place permanently: as long as the variable is unset or blank, the expression evaluates to an empty string and the action falls back to the `engines.node` field as usual. Only when the variable is populated does it take precedence - so a temporary override becomes a pure configuration change, and removing the variable's value restores the default behaviour. The job summary and the `source` output always show which of the two sources was used.

Note that when an explicit version is provided this way, a _package.json_ is no longer required for the version lookup itself. With `package-manager: auto`, the package manager is then detected from the lock file alone.

## References

The idea for the creation of this action and some further details are described in the following Medium post: [How to enforce a specific node version within a project and the associated CI/CD automations](https://medium.com/gitconnected/how-to-enforce-a-specific-node-version-within-a-project-and-the-associated-ci-cd-automations-aca9697fb9e)

## Releases

Latest release notes can be found on the [release page](https://github.com/svierk/get-node-version/releases).

## License

The scripts and documentation in this project are released under the [MIT License](https://github.com/svierk/get-node-version/blob/main/LICENSE).