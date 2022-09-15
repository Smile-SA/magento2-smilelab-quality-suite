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

Create three file at the root of you project directory:

- phpcs.xml.dist ([example](https://github.com/Smile-SA/magento2-module-debug-toolbar/blob/master/phpcs.xml.dist))
- phpmd.xml.dist ([example](https://github.com/Smile-SA/magento2-module-debug-toolbar/blob/master/phpmd.xml.dist))
- phpstan.neon.dist ([example](https://github.com/Smile-SA/magento2-module-debug-toolbar/blob/master/phpstan.neon.dist))

## Analyse Your Code

```shell
# Check registered vulnerabilities
composer audit

# Analyse php syntax
php vendor/bin/parallel-lint --exclude vendor [src path]

# Analyse code style
php vendor/bin/phpcs --standard=SmileLab [src path]

# Analyse code complexity
php vendor/bin/phpmd [src path] text phpmd.xml.dist

# Analyse code logic
php vendor/bin/phpstan analyse [src path]
```

## Fix your code

A lot of style errors can be fixed automatically by running this command:

```shell
php vendor/bin/phpcbf -s --standard=SmileLab --extensions=php,phtml [src path]
```

## CI

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
        - 'vendor/bin/phpcs --standard=SmileLab --extensions=php,phtml [src path]'
        - 'vendor/bin/phpmd [src path] text phpmd.xml.dist'
        - 'vendor/bin/phpstan analyse [src path]'
    tags:
        - 'php81'
```

### GitHub Workflow

Example of `.github/workflows/static-analysis.yaml` file:

```yaml
name: 'Static Code Analysis'

on:
    pull_request: ~
    push:
        branches:
            - 'master'

jobs:
    tests:
        runs-on: 'ubuntu-latest'

        strategy:
            matrix:
                php-version:
                    - '8.1'

        steps:
            - name: 'Checkout'
              uses: 'actions/checkout@v3'

            - name: 'Install PHP'
              uses: 'shivammathur/setup-php@v2'
              with:
                  php-version: '${{ matrix.php-version }}'
                  coverage: 'none'
                  tools: 'composer:v2'

            - name: 'Get composer cache directory'
              id: 'composercache'
              run: 'echo "::set-output name=dir::$(composer config cache-files-dir)"'

            - name: 'Cache dependencies'
              uses: 'actions/cache@v3'
              with:
                  path: '${{ steps.composercache.outputs.dir }}'
                  key: ${{ runner.os }}-composer-${{ hashFiles('**/composer.json') }}
                  restore-keys: '${{ runner.os }}-composer-'

            - name: 'Prepare credentials'
              env:
                  MAGENTO_USERNAME: '${{ secrets.MAGENTO_USERNAME }}'
                  MAGENTO_PASSWORD: '${{ secrets.MAGENTO_PASSWORD }}'
              run: 'composer config -g http-basic.repo.magento.com "$MAGENTO_USERNAME" "$MAGENTO_PASSWORD"'

            - name: 'Install dependencies'
              run: 'composer install --prefer-dist'

            - name: 'Run composer audit'
              run: 'composer audit --format=plain'

            - name: 'Run Parallel Lint'
              run: 'vendor/bin/parallel-lint --exclude vendor [src path]'

            - name: 'Run PHP CodeSniffer'
              run: 'vendor/bin/phpcs --standard=SmileLab --extensions=php,phtml [src path]'

            - name: 'Run PHPMD'
              run: 'vendor/bin/phpmd [src path] xml phpmd.xml.dist'

            - name: 'Run PHPStan'
              run: 'vendor/bin/phpstan analyse [src path]' 
```
