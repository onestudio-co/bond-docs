# FAQ

- How do I switch environments?
  - Use `--dart-define-from-file` per flavor and keep separate Firebase configs.

- How do I add a new feature?
  - Create a feature module and a Service Provider, register APIs/controllers, and add routes.

- How do I cache API results?
  - Use BondFire `cacheThenNetwork` or Cache `remember` helper.
