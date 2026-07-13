# 🕵🏻 Get Node Version

This repository implements a very simple GitHub composite action that pulls the Node.js version from the _package.json_ file of the project and ensures that it is used for the current pipeline execution. The action relies on the Node.js version specified within the [engines](https://docs.npmjs.com/cli/v10/configuring-npm/package-json#engines) section of the _package.json_, e.g:

```
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

```
name: build

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v7

      - name: Select Node Version
        uses: svierk/get-node-version@v1.5.0
```

### Inputs

| Name              | Required | Default | Description                                                                |
| ----------------- | -------- | ------- | -------------------------------------------------------------------------- |
| `node-version`    | no       | –       | Explicit Node.js version spec (e.g. `24.x`) that takes precedence over the _package.json_ lookup. An empty value falls back to `engines.node`. |
| `path`            | no       | `./`    | Path where the _package.json_ file can be found.                           |
| `package-manager` | no       | `npm`   | Package manager used for dependency caching. Supported: `npm`, `yarn`, `pnpm`. |
| `step-summary`    | no       | `true`  | Write a result section to the GitHub Actions [job summary](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#adding-a-job-summary). Set to `false` to avoid collisions with a custom workflow summary. |

When using `pnpm`, make sure `pnpm` is available before this action runs (e.g. via [`pnpm/action-setup`](https://github.com/pnpm/action-setup)), as it is required for the dependency cache.

### Outputs

| Name           | Description                                                                          |
| -------------- | ------------------------------------------------------------------------------------ |
| `node-version` | The Node.js version spec resolved from the `node-version` input or the `engines.node` field. |
| `source`       | Where the version spec was resolved from: `node-version input` or `package.json`.    |

The action fails with a clear error if no `node-version` input is provided and the _package.json_ or its `engines.node` field is missing, so a misconfiguration surfaces immediately instead of silently falling back to an undefined version.

### Overriding the version via repository variable

The _package.json_ remains the recommended single source of truth, since it is the only place local developer tooling picks the version up from as well. However, sometimes you temporarily need to pin the pipeline to a fixed, known-good Node.js version - ideally without touching the workflow file at all. For this, wire the `node-version` input to a [repository variable](https://docs.github.com/en/actions/learn-github-actions/variables) (organization and environment variables work the same way):

```yaml
- name: Select Node Version
  uses: svierk/get-node-version@v1.5.0
  with:
    node-version: ${{ vars.NODE_VERSION_OVERRIDE }}
```

This wiring is safe to keep in place permanently: as long as the variable is unset or blank, the expression evaluates to an empty string and the action falls back to the `engines.node` field as usual. Only when the variable is populated does it take precedence - so a temporary override becomes a pure configuration change, and removing the variable's value restores the default behaviour. The job summary and the `source` output always show which of the two sources was used.

Note that when an explicit version is provided this way, a _package.json_ is no longer required for the version lookup itself - but the lock file matching the selected `package-manager` still needs to exist for dependency caching.

## References

The idea for the creation of this action and some further details are described in the following Medium post: [How to enforce a specific node version within a project and the associated CI/CD automations](https://medium.com/gitconnected/how-to-enforce-a-specific-node-version-within-a-project-and-the-associated-ci-cd-automations-aca9697fb9e)

## Releases

Latest release notes can be found on the [release page](https://github.com/svierk/get-node-version/releases).

## License

The scripts and documentation in this project are released under the [MIT License](https://github.com/svierk/get-node-version/blob/main/LICENSE).