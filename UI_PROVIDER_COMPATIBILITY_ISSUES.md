# UI Provider Compatibility Issues

## Contexte

L'erreur suivante signifie que l'application enregistre un service provider UI qui n'existe pas dans la version de `velt/ui` installee :

```text
Provider class [Velt\Kernel\Ui\UiServiceProvider] does not exist.
```

ou :

```text
Provider class [Velt\Ui\Providers\UiServiceProvider] does not exist.
```

Le provider correct est :

```php
Velt\Ui\Providers\UiServiceProvider
```

Il est fourni par `velt/ui` depuis la version `0.1.0`.

## Issue 1 - Namespace obsolete dans le bootstrap

Symptome :

```php
use Velt\Kernel\Ui\UiServiceProvider;
```

Cause :

Le skeleton essaie de charger le provider UI depuis le namespace du kernel. Le kernel ne doit pas contenir l'integration UI.

Correction :

```php
use Velt\Ui\Providers\UiServiceProvider;
```

Fichiers a verifier :

- `bootstrap/app.php`
- templates de skeleton utilises par la CLI
- fixtures de test qui bootent une application complete

## Issue 2 - Version trop ancienne de velt/ui

Symptome :

Le bootstrap utilise le bon namespace, mais l'erreur reste :

```text
Provider class [Velt\Ui\Providers\UiServiceProvider] does not exist.
```

Cause :

Composer installe `velt/ui v0.0.2`, qui ne contient pas `src/Providers/UiServiceProvider.php`.

Correction dans le framework :

`veltphp-framework/composer.json` doit exiger :

```json
"velt/ui": "^0.1.0"
```

La matrice de compatibilite du framework doit aussi documenter `UI ^0.1.0`.

Correction dans un projet deja installe :

```bash
composer update velt/framework velt/ui --with-dependencies
composer dump-autoload
```

En developpement local, utiliser un repository `path` vers `../velt-ui` avec la version `0.1.0`.

## Issue 3 - Lock file stale dans le skeleton ou un projet test

Symptome :

`composer.json` semble correct, mais `composer.lock` garde :

```json
"name": "velt/ui",
"version": "v0.0.2"
```

Cause :

Le lock file a ete genere avant l'ajout du provider UI.

Correction :

```bash
composer update velt/framework velt/ui velt/kernel --with-dependencies
```

Verifier ensuite :

```bash
php -r "require 'vendor/autoload.php'; var_dump(class_exists('Velt\\Ui\\Providers\\UiServiceProvider'));"
```

Le resultat attendu est :

```text
bool(true)
```

## Issue 4 - Release Packagist incomplete

Symptome :

En local, le path repository fonctionne, mais une installation propre depuis Packagist reprend une combinaison incompatible.

Cause :

Le code local a ete corrige, mais les tags Composer publics ne sont pas alignes.

Correction release :

1. Tagger et publier `velt/ui` avec `UiServiceProvider` en `0.1.0`.
2. Tagger et publier `veltphp-framework` apres la contrainte `"velt/ui": "^0.1.0"`.
3. Regenerer le lock du skeleton apres publication.
4. Verifier une installation propre sans vendor existant.

## Verification minimale

Depuis un projet skeleton :

```bash
composer dump-autoload
php -r "require 'vendor/autoload.php'; var_dump(class_exists('Velt\\Ui\\Providers\\UiServiceProvider'));"
php public/index.php
vendor/bin/phpunit
```

Le front controller ne doit plus lever d'exception `Provider class ... does not exist`.
