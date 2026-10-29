# Drupal Conventions

**Detection triggers**: presence of a `drupal/core` or `drupal/core-recommended` dependency in `composer.json`. Applies in addition to `php.md`, and takes precedence over it on conflict. Version read from the `drupal/core*` constraint.

- Use object-oriented hooks (`#[Hook]` attribute, and `#[LegacyHook]` if needed) — available starting with Drupal 11.1 / the hook module; on earlier versions, fall back to classic procedural hooks (`hook_xxx()`) without flagging it as a defect.
- Avoid hooks as much as possible, favor the various plugin systems (Plugin API, Event Subscribers).
- Check the targeted Drupal version before requiring a recent API (e.g. PHP attributes for plugins are only available from Drupal 10.2/11 onward).
- Naming: snake_case for procedural functions and hooks. Drupal core mixes camelCase and snake_case for local variables: follow the surrounding file and flag neither.

## Performance and scalability

- Load entities in a single `loadMultiple()` rather than calling `load()` inside a loop.
- Bound entity queries with `->range()`; an unbounded query is fine on a dev dataset and fatal on a production one. Be explicit about `->accessCheck()` either way.
- Offload long-running work to the Queue API or the Batch API rather than running it inline in a request, or directly inside `hook_cron`.
- Set render-array `#cache` keys, contexts, tags, and max-age deliberately: a missing cache tag serves stale content, an over-broad context destroys the hit rate.
