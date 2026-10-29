# PHP Conventions

**Detection triggers**: presence of `composer.json`. Version read from `composer.json` (`require.php`). A framework dependency in `composer.json` (`drupal/core*`, and later `symfony/*`, `laravel/framework`) also loads its own reference file on top of this one.

- Use named arguments and named parameters.
- Favor dependency injection.
- Type function parameters and, more generally, prefer strong typing.
- Favor ValueObjects where applicable.
- Avoid business logic in Controllers.
- Favor services.
- Services should stay lightweight and well isolated (one service per responsibility rather than one service per feature).
- Naming: PascalCase for classes, lowerCamelCase for methods and object properties.
