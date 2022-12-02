# github-workflows

This repository contains shared github actions which can be used by different projects.

## Prerequisites

This software can't be run directly on a client machine. See the [internal documentation](https://it-docs.ub.unibe.ch) to see how to implement these workflows in the repositories.

## Installation

This software can't be installed on a client machine.

## Configuration

There is no configuration present in this software.

## Usage

Currently there are following workflows which can be reused:

* phpstan
* php-cs-fixer
* phpunit

Following shows an example GitHub action which calls these workflows:

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
      database-init: "schema" # possible values are "schema", "migration" and "none"
      database-load-fixtures: false
      start-services: true
    secrets:
      packagist-token: ${{ secrets.PACKAGIST_TOKEN }}
      node-token: ${{ secrets.NODE_AUTH_TOKEN }}
```

In this example all valid parameters are listed with their default value. For better explanation, look into the corresponding workflow file.

Please note that for phpunit, the `node-token` is used to download the npm packages from the ub-unibe-ch repository, while the `packagist-token` is used to download the composer packages.

### PHPUnit HealthCheck

The `phpunit` workflow starts the required services by invoking `docker compose up --wait` to ensure that the service is started before the tests are executed. Because most of the published resources have no healthcheck defined, you have to define one in the `docker-compose.yml`. 

#### PostgresSQL

```yaml
services:
  # ...
  database:
    # ...
    healthcheck:
      test: [ "CMD", "pg_isready" ]
      timeout: 5s
      interval: 5s
      retries: 10
    # ...
```

#### MariaDB

```yaml
services:
  # ...
  database:
    # ...
    environment:
      # ...
      MARIADB_MYSQL_LOCALHOST_USER: 1
    healthcheck:
      test: [ "CMD-SHELL", "/usr/local/bin/healthcheck.sh --su-mysql --innodb_initialized || exit 1" ]
      retries: 10
      interval: 5s
      timeout: 5s
    # ...
```

### Panther tests

Panther tests require a special setup. The problem is, that PHPUnit will instantiate the tests with the environment from docker. These tests would normally start the panther environment, which doesn't have the same environment. As an example, while the tests write in the docker provided database, panther would have the database defined in `.env` (or `.env.panther`).

To fix the problem, start the panther environment explicit with the same variables as the test. Following snippet shows how to do to it:

```php
<?php

namespace App\Tests\Controller;

use Symfony\Component\Panther\PantherTestCase;

class ControllerTest extends PantherTestCase
{
    public function testSomething(): void
    {
        $variables = ['DATABASE_URL', 'MAILER_DSN']; // Define here all services which are defined in docker
                                                     // You can invoke `symfony var:export --debug` to detect all docker services
        $env = [];
        foreach ($variables as $variable) {
            if (isset($_ENV[$variable])) {
                $env[$variable] = $_ENV[$variable];
            }
        }

        $client = self::createPantherClient([
            'env' => $env,
        ]);
        
        // ...
    }
}
```

You probably want to save this snippet as abstract class in your project somewhere to instantiate the panther client.

Also ensure that in the `config/packages/doctrine.yaml` the panther environment also gets the `_test` suffix:

```yaml

when@panther:
    doctrine:
        dbal:
            # "TEST_TOKEN" is typically set by ParaTest
            dbname_suffix: '_test%env(default::TEST_TOKEN)%'
```

## Contributing

Simply change the corresponding workflow file and commit & push the changes. If you want to test the changes, you have to reference the workflow files by the branch name, like so:

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
