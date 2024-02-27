# 🕵🏻 Get Node Version V1

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
        uses: svierk/get-node-version@v1
```

## References

The idea for the creation of this action and some further details are described in the following Medium post: [How to enforce a specific node version within a project and the associated CI/CD automations](https://medium.com/gitconnected/how-to-enforce-a-specific-node-version-within-a-project-and-the-associated-ci-cd-automations-aca9697fb9e)

## Releases

Latest release notes can be found on the [release page](https://github.com/svierk/get-node-version/releases).

## License

The scripts and documentation in this project are released under the [MIT License](https://github.com/svierk/get-node-version/blob/main/LICENSE).