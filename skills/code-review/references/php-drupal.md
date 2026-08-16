# PHP / Drupal Conventions

**Detection triggers**: presence of `composer.json` (PHP), with a `drupal/core` or `drupal/core-recommended` dependency (Drupal). PHP version read from `composer.json` (`require.php`); Drupal version read from the `drupal/core*` constraint.

## PHP (general)

- Use named arguments and named parameters.
- Favor dependency injection.
- Type function parameters and, more generally, prefer strong typing.
- Favor ValueObjects where applicable.
- Avoid business logic in Controllers.
- Favor services.
- Services should stay lightweight and well isolated (one service per responsibility rather than one service per feature).

## Drupal

- Use object-oriented hooks (`#[Hook]` attribute, and `#[LegacyHook]` if needed) — available starting with Drupal 11.1 / the hook module; on earlier versions, fall back to classic procedural hooks (`hook_xxx()`) without flagging it as a defect.
- Avoid hooks as much as possible, favor the various plugin systems (Plugin API, Event Subscribers).
- Check the targeted Drupal version before requiring a recent API (e.g. PHP attributes for plugins are only available from Drupal 10.2/11 onward).

## Performance and scalability

- Load entities in a single `loadMultiple()` rather than calling `load()` inside a loop.
- Bound entity queries with `->range()`; an unbounded query is fine on a dev dataset and fatal on a production one. Be explicit about `->accessCheck()` either way.
- Offload long-running work to the Queue API or the Batch API rather than running it inline in a request, or directly inside `hook_cron`.
- Set render-array `#cache` keys, contexts, tags, and max-age deliberately: a missing cache tag serves stale content, an over-broad context destroys the hit rate.
