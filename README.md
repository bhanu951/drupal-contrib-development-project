# Drupal Contrib Development Project

A minimal project build for Drupal Contrib Modules, Themes and Profiles Development and Testing.

## Contents

* [Getting started](#getting-started)
* [Common tasks](#common-tasks)
* [Sample workflows](#sample-workflows)

## Getting started

### Requirements

This project requires [DDEV](https://ddev.readthedocs.io/en/latest/users/install/) to be installed before you begin.

### Drupal Contrib Development Using DDEV

1. Clone this repository by using `git clone --branch=master https://github.com/bhanu951//drupal-contrib-development-project.git`
2. cd drupal-contrib-development-project
3. ddev start
4. ddev composer install (from project root)
5. ddev auth ssh
6. Now you are ready to install Drupal and test modules run `ddev install`. This command will install Drupal 10 plus the following useful modules: `devel`, `config_inspector`, `admin_toolbar`, and `admin_toolbar_tools`. You can login with `admin / admin` at https://drupal-contrib-development-project.ddev.site/user.

#### Note on NFS

If using NFS to speed up the container, see these steps.

* `ddev debug nfsmount` (optional)
  * See the [macOS NFS Setup section](https://ddev.readthedocs.io/en/latest/users/install/performance/#nfs) of the DDEV documentation.

### Drush

`drush 12` is installed by default and can be run with `ddev drush COMMAND`. You can use `ddev drush site:install` if you want to customize the install.

### Composer

The use of `ddev composer require` is assumed to be used for maintenance of the core framework, not adding modules for testing.

You can use `ddev composer require` to add modules that you need -- and to check dependencies. Do not commit those additions to the project.

## Common tasks

We have automated common commands as ddev commands to reduce dependencies. The following commands are available.

### ddev install

**Command:** `ddev install`

Installs the default drupal site with `devel`, `config_inspector`, `admin_toolbar`, and `admin_toolbar_tools` modules installed.

### ddev phpcs

**Command:** `ddev phpcs PATH_TO_MODULE`

**Example:** `ddev phpcs web/modules/contrib/admin_toolbar`

The `phpcs` command will run PHPCS against the selected module.

### ddev phpcbf

**Command:** `ddev phpcbf PATH_TO_MODULE`

**Example:** `ddev phpcbf web/modules/contrib/admin_toolbar`

The `phpcbf` command will run PHPCS against the selected module.

### ddev dck

**Command:** `ddev dck PATH_TO_MODULE`

**Example:** `ddev dck web/modules/contrib/admin_toolbar`

The `dck` command will run code reviews using drupal-check against the selected module.

### ddev phpversion

**Command:** `ddev phpversion PATH_TO_MODULE [-v VERSION]`

**Example:** `ddev phpversion web/modules/contrib/admin_toolbar`

**Example:** `ddev phpversion admin_toolbar -v 7.4`

The `phpversion` command will run PHPCS against the selected module using the `PHPCompatibility` coding standard.

Use the `-v` flag to specify a PHP version to test. By default, the version is `8.2`.

### ddev phpstan

**Command:** `ddev phpstan PATH_TO_MODULE`

**Example:** `ddev phpstan web/modules/contrib/admin_toolbar`

**Example:** `ddev phpstan admin_toolbar -l 8`

The `stan` command will run code reviews using PHPStan against the selected module.

This command defaults to use [PHPStan level 2](https://phpstan.org/user-guide/rule-levels). You can pass a preferred level (`0-9`, or `max`) using the `-l` flag.

### ddev sniff

**Command:** `ddev sniff PATH_TO_MODULE`

**Example:** `ddev sniff web/modules/contrib/admin_toolbar`

The `sniff` command will run code reviews using PHPCBF, PHPCS, PHPStan, drupal-check and rector dry-run against the selected module. PHPCBF may change the module's code as part of this action.

These commands can also be run individually as `ddev phpcs`, `ddev phpcbf`, `ddev dck`,`ddev phpstan`, `ddev phpversion` and `ddev rector`

Note that PHPStan is pinned to [PHPStan level 2](https://phpstan.org/user-guide/rule-levels) in this command. Use [`ddev phpstan`](#ddev-stan) to override the level.

Note that compatibility phpversion checks against is PHP 8.2 in this command. Use [`ddev phpversion`](#ddev-stan) to check against specific PHP version.

### ddev test

**Command:** `ddev test MODULE_NAME`

**Example:** `ddev test admin_toolbar`

**Example:** `ddev test admin_toolbar Functional AdminToolbarAdminMenuTest`

The `ddev test` command runs all tests defined by a module according to Drupal's testing standards.

If you pass the type of test (`Functional, FuntionalJavascript, Kernel, Unit`) and a test class, only tests in that class will be run.

This command uses `testdox` output, which is easy to read but does not provide debugging output.

You can run `testb` with the same parameters to get browser output.

### ddev tests-cleanup

**Command:** `ddev tests-cleanup`

Removes test files generated by `ddev test`.

### ddev clone

**Command:** `ddev clone MODULE_NAME [-b BRANCH_NAME | -t TAG_NAME] [-c]`

**Example:** `ddev clone admin_toolbar`

**Example:** `ddev clone workbench_access -c`

**Example:** `ddev clone admin_toolbar -b 8.x-1.x`

**Example:** `ddev clone my_module -o git@github.com:palantirnet`

Use `ddev clone` rather than `ddev composer require` if you want to checkout a git project for development.

The `clone` command will checkout the default branch of the selected module to the `web/modules/contrib` directory.

Use the `-b` or `-t` flags to specify a branch or tag. The flags cannot be used together.

Adding the `-c` flag indicates you are a `project contributor`, not a `project maintainer`. The command will use https rather than ssh to checkout the project. This flag is ignored when using `-o`.

Use the `-o` flag to specify a different origin than drupal.org. The argument is a path to the account that owns the repository (see the example above).

Note that this command will `delete` any existing copy of the module.

### ddev rector

**Command:** `ddev rector PATH_TO_MODULE` or `ddev rector PATH_TO_MODULE -d` or `ddev rector PATH_TO_MODULE --dry-run`

**Example:** `ddev rector web/modules/contrib/admin_toolbar`

**Example:** `ddev rector web/modules/contrib/admin_toolbar -d`

The `rector` command will run Drupal Rector updates against the selected module, potentially rewriting the module's code. Using the `-d` or `--dry-run` flag will not perform the changes, but instead show the suggested changes.

### ddev remove

**Command:** `ddev remove MODULE_NAME`

**Example:** `ddev remove admin_toolbar`

The `ddev remove` command will uninstall a module and delete it from the `web/modules/contrib` directory.

## Sample workflows

Let's assume you want to write a patch for the `workbench_tabs` module. Follow these steps:

### Creating a patch

* `ddev start`
* `composer install`
* `ddev install`
* `ddev clone workbench_tabs`
  * If you are a maintainer, use `ddev clone workbench_tabs -y`
* `ddev test workbench_tabs`
  * Run this step to ensure that existing tests pass.
  * Then run `ddev cleanup` to remove test files.
* `ddev drush en workbench_tabs`
* Now develop your new feature or bugfix in the module
  * Do not check your changes into git before creating the patch.
  * Create a patch
    * `cd web/modules/workbench_tabs`
    * `git diff > ISSUE#-PATH_TO_MODULE-summary-COMMENT.patch`
  * Now you can upload the patch to the Drupal.org issue.

### Testing a patch

* `ddev start`
* `composer install`
* `ddev install`
* `ddev clone workbench_tabs`
  * If you are a maintainer, use `ddev clone workbench_tabs -y`
* `ddev test workbench_tabs`
  * Run this step to ensure that existing tests pass.
  * Then run `ddev cleanup` to remove test files.
* `ddev drush en workbench_tabs`
* Download the patch to `web/modules/workbench_tabs`:
  * `cd web/modules/workbench_tabs`
  * `wget https://drupal.org/PATH-TO-FILE/NAME-OF-FILE`
  * `patch -p1 < NAME-OF-FILE`
  * `ddev drush cr`
* Now you can test the patch in the site.
