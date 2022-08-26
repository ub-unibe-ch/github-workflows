# github-workflows

This repository contains shared github actions which can be used by different projects.

## Prerequisites

This software can't be run directly on a client machine. See the [internal documentation](https://it-docs.ub.unibe.ch) to see how to implement these workflows in the repositories.

## Installation

This software can't be installed on a client machine.

## Configuration

There is no configuration present in this software.

## Usage

Currently there these workflows which can be reused:

* phpstan
* php-cs-fixer
* phpunit

Following lists an example GitHub action which calls these workflows:

```yaml
name: "Build"

on:
  push:
    branches:
      - master
      - main
  pull_request:
    branches:
      - master
      - main

jobs:
  phpstan:
    uses: ub-unibe-ch/github-workflows/.github/workflows/phpstan.yaml@v1
    with: 
      php-version: "8.1"
      php-extensions: "intl, json, zip"
    secrets:
      packagist-token: ${{ secrets.PACKAGIST_TOKEN }}

  php-cs-fixer:
    uses: ub-unibe-ch/github-workflows/.github/workflows/php-cs-fixer.yaml@v1
    with: 
      php-version: "8.1"
      php-extensions: "intl, json, zip"
    secrets:
      packagist-token: ${{ secrets.PACKAGIST_TOKEN }}

  phpunit:
    needs: [ "phpstan", "php-cs-fixer" ]
    uses: ub-unibe-ch/github-workflows/.github/workflows/phpunit.yaml@v1
    with: 
      php-version: "8.1"
      php-extensions: "intl, json, zip"
      database-init: "schema" # possible values are "schema" and "migration"
      database-load-fixtures: false
    secrets:
      packagist-token: ${{ secrets.PACKAGIST_TOKEN }}
```

In this example all valid parameters are listed with their default value. For better explanation, look into the corresponding workflow file.

## Contributing

Simply change the corresponding workflow file and commit & push the changes. If you wan't to test the changes, you have to reference the workflow files by the branch name, like so:

```yaml

# ...
jobs:
  phpstan:
    uses: ub-unibe-ch/github-workflows/.github/workflows/phpstan.yaml@main # <-- here
  
# ...
  php-cs-fixer:
    uses: ub-unibe-ch/github-workflows/.github/workflows/php-cs-fixer.yaml@main # <-- here
# ...  
  phpunit:
    uses: ub-unibe-ch/github-workflows/.github/workflows/phpunit.yaml@main # <-- here
```

When you are finished with your changes, you have to move the git tag to the new commit:

```
git push origin :refs/tags/<tagname>
git tag -fa <tagname>
git push --tags
```

## References

* [Reusing Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
