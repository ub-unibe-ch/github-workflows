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
* test

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
    uses: ub-unibe-ch/github-workflows/.github/workflows/phpstan.yaml@main
    with: 
      php-version: "8.1"
      php-extensions: "intl, json, zip"
    secrets:
      packagist-token: ${{ secrets.PACKAGIST_TOKEN }}

  php-cs-fixer:
    uses: ub-unibe-ch/github-workflows/.github/workflows/php-cs-fixer.yaml@main
    with: 
      php-version: "8.1"
      php-extensions: "intl, json, zip"
    secrets:
      packagist-token: ${{ secrets.PACKAGIST_TOKEN }}

  phpunit:
    needs: [ "phpstan", "php-cs-fixer" ]
    uses: ub-unibe-ch/github-workflows/.github/workflows/phpunit.yaml@main
    with: 
      php-version: "8.1"
      php-extensions: "intl, json, zip"
      database-init: "schema"
      database-load-fixtures: false
    secrets:
      packagist-token: ${{ secrets.PACKAGIST_TOKEN }}

```

In this example all valid parameters are listed with their default value. For better explanation, look into the corresponding workflow file.

### phpstan

# Contributing

!!! info
    What you have to do to if you want to extend the code

# References

!!! info
    Useful links
