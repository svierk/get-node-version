# 🕵🏻 Get Node Version

This repository implements a very simple GitHub composite action that pulls the Node.js version from the _package.json_ file of the project and ensures that it is used for the current pipeline execution. The action relies on the Node.js version specified within the [engines](https://docs.npmjs.com/cli/v10/configuring-npm/package-json#engines) section of the _package.json_, e.g:

```
{
  "engines": {
    "node": "20.x.x",
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
        uses: actions/checkout@v4

      - name: Select Node Version
        uses: svierk/get-node-version@v1.0.1
```

### Inputs

| Name              | Required | Default | Description                                                                |
| ----------------- | -------- | ------- | -------------------------------------------------------------------------- |
| `path`            | no       | `./`    | Path where the _package.json_ file can be found.                           |
| `package-manager` | no       | `npm`   | Package manager used for dependency caching. Supported: `npm`, `yarn`, `pnpm`. |
| `step-summary`    | no       | `true`  | Write a result section to the GitHub Actions [job summary](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#adding-a-job-summary). Set to `false` to avoid collisions with a custom workflow summary. |

When using `pnpm`, make sure `pnpm` is available before this action runs (e.g. via [`pnpm/action-setup`](https://github.com/pnpm/action-setup)), as it is required for the dependency cache.

### Outputs

| Name           | Description                                                       |
| -------------- | ----------------------------------------------------------------- |
| `node-version` | The Node.js version resolved from the `engines.node` field.       |

The action fails with a clear error if the _package.json_ or its `engines.node` field is missing, so a misconfiguration surfaces immediately instead of silently falling back to an undefined version.

## References

The idea for the creation of this action and some further details are described in the following Medium post: [How to enforce a specific node version within a project and the associated CI/CD automations](https://medium.com/gitconnected/how-to-enforce-a-specific-node-version-within-a-project-and-the-associated-ci-cd-automations-aca9697fb9e)

## Releases

Latest release notes can be found on the [release page](https://github.com/svierk/get-node-version/releases).

## License

The scripts and documentation in this project are released under the [MIT License](https://github.com/svierk/get-node-version/blob/main/LICENSE).