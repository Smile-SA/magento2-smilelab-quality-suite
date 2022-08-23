# Smile Lab Quality Suite tools

This library contains tools used by Smile Lab for quality control on Magento2 community projects.

## Install

```bash
composer require --dev smile/magento2-smilelab-quality-suite
```

Create three file at the root of you project directory : 
- `phpcs.xml.dist`
    ```xml
    <?xml version="1.0"?>
    <ruleset xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="vendor/squizlabs/php_codesniffer/phpcs.xsd">

        <arg name="basepath" value="."/>
        <arg name="extensions" value="php,phtml"/>
        <arg name="colors"/>
  
        <config name="php_version" value="[MIN_COMPATIBLE_PHP_VERSION]"/>

        <!-- Show progress of the run -->
        <arg value="p"/>

        <!-- Show sniff codes -->
        <arg value="s"/>

        <rule ref="SmileLab"/>      

        <exclude-pattern>vendor/*</exclude-pattern>
    </ruleset>
    ```
    You must [specify the minimum php version](https://github.com/squizlabs/PHP_CodeSniffer/wiki/Configuration-Options#setting-the-php-version) that your module require by setting the config php_version in your phpcs config file. For example, to set php 7.4 as the min version, specify the value `70400`.


- `phpmd.xml.dist`
    ```xml
    <?xml version='1.0' encoding="UTF-8"?>
    <ruleset xmlns="http://pmd.sf.net/ruleset/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://pmd.sf.net/ruleset/1.0.0 http://pmd.sf.net/ruleset_xml_schema.xsd" xsi:noNamespaceSchemaLocation="http://pmd.sf.net/ruleset_xml_schema.xsd">

        <rule ref="vendor/smile/magento2-smilelab-phpmd/ruleset.xml"/>

        <exclude-pattern>vendor/*</exclude-pattern>
    </ruleset>
    ```
  - `phpstan.neon.dist`
      ```neon
      parameters:
          level: 6
          phpVersion: [MIN_COMPATIBLE_PHP_VERSION]
          checkMissingIterableValueType: false
          excludePaths:
              - 'vendor/*'
      ```

      If you also install phpstan/extension-installer then you're all set!

      Otherwise, add the following configuration to this file:

      ```neon
      includes:
          - vendor/smile/magento2-smilelab-phpstan/extension.neon
      ```

      You must [specify the minimum php version](https://phpstan.org/config-reference#miscellaneous-parameters) that your module require by setting the config php_version in your phpcs config file. For example, to set php 7.4 as the min version, specify the value `70400`.

## Analyze your code

```bash
# Check registered vulnerabilities
composer audit

# Analyze php syntax
php vendor/bin/parallel-lint --exclude vendor [src path]

# Analyze code style
php vendor/bin/phpcs --standard=SmileLab [src path]

# Analyze code complexity
php vendor/bin/phpmd [src path] text phpmd.xml.dist

# Analyze code logic
php vendor/bin/phpstan analyse [src path]
```

## Fix your code

A lot of style errors can be fixed automatically, you can run this command to do it :

```bash
php vendor/bin/phpcbf -s --standard=SmileLab --extensions=php,phtml [src path]
```

## CI

You should consider validate the code of you project automatically, there some exemple of configuration file you can use on Gitlab or GitHub project.

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

Example of `.github/workflows/ci.yaml` file:

```yaml
name: 'CI'

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
