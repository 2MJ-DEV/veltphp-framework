# Velt Framework

Official Velt framework meta-package and integration layer.

## Role

This repository assembles the official Velt components without replacing them. It exists to document and enforce compatible versions between:

- `velt/kernel`
- `velt/http`
- `velt/ui`
- `velt/database`
- `velt/orm`
- `velt/cli`
- `velt/preview`
- `velt/skeleton`

## Skeleton vs Framework

`veltphp/skeleton` is the application template copied or installed by developers.

`veltphp/framework` is the Composer-level assembly package that defines which Velt components work together.

## Module 3 Goal

The first Module 3 task is to create the framework meta-package, document local development with Composer path repositories, and maintain a compatibility matrix for multi-repo work.

## Compatibility Matrix

| Velt Framework | PHP | Kernel | HTTP | UI | Database | ORM | CLI | Preview | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `^0.1.0` | `^8.2` | `^0.1.0` | `^0.1.0` | `^0.0.2` | `^0.1.0` | `^0.1.0` | `^0.2.0` | `^0.1.0` | active development |

## Package Policy

The framework package should not replace component packages. It only declares which released component versions are expected to work together.

For Packagist releases, dependencies must use version constraints such as `^0.1.0`. Local `path` repositories belong in application-level development setups, not in the published framework package.
