# Smile Lab Quality Suite (Magento)

This library provides coding standards / rulesets that can be used on Magento projects.

It includes the following packages:

- [Smile Lab PHPCS Coding Standard](https://github.com/Smile-SA/magento2-smilelab-phpcs)
- [Smile Lab PHPMD Ruleset](https://github.com/Smile-SA/magento2-smilelab-phpmd)
- [SmileLab PHPStan Extension](https://github.com/Smile-SA/magento2-smilelab-phpstan)

## Installation

```shell
composer require --dev smile/magento2-smilelab-quality-suite
```

## Configuration

Create three files at the root of your project directory:

- phpcs.xml.dist ([example](https://github.com/Smile-SA/magento2-module-debug-toolbar/blob/master/phpcs.xml.dist))
- phpmd.xml.dist ([example](https://github.com/Smile-SA/magento2-module-debug-toolbar/blob/master/phpmd.xml.dist))
- phpstan.neon.dist ([example](https://github.com/Smile-SA/magento2-module-debug-toolbar/blob/master/phpstan.neon.dist))

## Usage

```shell
# Check registered vulnerabilities
composer audit

# Analyse php syntax
vendor/bin/parallel-lint --exclude vendor [src path]

# Analyse code style
vendor/bin/phpcs

# Analyse code complexity
vendor/bin/phpmd [src path] text phpmd.xml.dist

# Analyse code logic
vendor/bin/phpstan analyse
```

## Fix your code

A lot of style errors can be fixed automatically by running this command:

```shell
vendor/bin/phpcbf --extensions=php,phtml
```

## CI

### GitHub Workflow

Example of `.github/workflows/static-analysis.yaml` file:

```yaml
name: 'Static Analysis'

on:
    pull_request: ~
    push:
        branches:
            - 'master'

jobs:
    tests:
        runs-on: 'ubuntu-latest'

        steps:
            - name: 'Checkout'
              uses: 'actions/checkout@v3'

            - name: 'Install PHP'
              uses: 'shivammathur/setup-php@v2'
              with:
                  php-version: '8.1'
                  coverage: 'none'
                  tools: 'composer:v2'
              env:
                  COMPOSER_AUTH_JSON: |
                      {
                          "http-basic": {
                              "repo.magento.com": {
                                  "username": "${{ secrets.MAGENTO_USERNAME }}",
                                  "password": "${{ secrets.MAGENTO_PASSWORD }}"
                              }
                          }
                      }

            - name: 'Get composer cache directory'
              id: 'composer-cache'
              run: 'echo "dir=$(composer config cache-files-dir)" >> $GITHUB_OUTPUT'

            - name: 'Cache dependencies'
              uses: 'actions/cache@v3'
              with:
                  path: '${{ steps.composer-cache.outputs.dir }}'
                  key: ${{ runner.os }}-composer-${{ hashFiles('**/composer.lock') }}
                  restore-keys: '${{ runner.os }}-composer-'

            - name: 'Install dependencies'
              run: 'composer install --prefer-dist'

            - name: 'Run composer audit'
              run: 'composer audit --format=plain'

            - name: 'Run Parallel Lint'
              run: 'vendor/bin/parallel-lint --exclude vendor [src path]'

            - name: 'Run PHP CodeSniffer'
              run: 'vendor/bin/phpcs --extensions=php,phtml'

            - name: 'Run PHPMD'
              run: 'vendor/bin/phpmd [src path] xml phpmd.xml.dist'

            - name: 'Run PHPStan'
              run: 'vendor/bin/phpstan analyse' 
```

### GitLab Runner

Example of `.gitlab-ci.yml` file:

```yaml
before_script:
  - 'composer install'

sniffers:
    variables:
        COMPOSER_AUTH: $COMPOSER_AUTH
    script:
        - 'composer audit --format=plain'
        - 'vendor/bin/parallel-lint --exclude vendor [src path]'
        - 'vendor/bin/phpcs --extensions=php,phtml'
        - 'vendor/bin/phpmd [src path] text phpmd.xml.dist'
        - 'vendor/bin/phpstan analyse'
    tags:
        - 'php81'
```
