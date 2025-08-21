# What Is Bond

Bond is a modular Flutter toolkit to ship production apps faster.

## Why

- Consistent architecture with Service Providers
- Typed data flows for networking, cache, forms
- Production scaffolding out of the box

## TL;DR

- Create a project with Bond CLI
- Configure env and Firebase
- Build features as modules with their own providers

## Steps

1) Install CLI and create a project
2) Configure environments and Firebase
3) Register feature providers

## Example

```bash
dart pub global activate bond_cli
bond create project
```

## Deep Dive

- See Philosophy for design choices
- See Core Concepts for providers, modules, and configuration

## Pitfalls

- Mixing feature responsibilities in a single provider
- Skipping env separation between flavors

## Next Steps

- Read Philosophy
- Continue with Getting Started
