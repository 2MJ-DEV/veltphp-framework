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

## Repartition par repository

Ces issues doivent etre creees dans des repositories separes.

| Issue | Repository cible | Objectif |
| --- | --- | --- |
| Issue 1 | `Velt-PHP/veltphp-skeleton` | Corriger le namespace du provider UI dans le bootstrap applicatif. |
| Issue 2 | `Velt-PHP/veltphp-framework` | Verrouiller la compatibilite framework vers `velt/ui ^0.1.0`. |
| Issue 3 | `Velt-PHP/veltphp-skeleton` | Regenerer le lock file du skeleton avec les versions compatibles. |
| Issue 4 | `Velt-PHP/velt-ui` | Publier/tagger une version contenant `UiServiceProvider`. |
| Issue 5 | `Velt-PHP/veltphp-cli` | Verifier que `velt new` genere un projet avec le skeleton corrige. |

## Issue 1 - Corriger le namespace UI dans le skeleton

Repository cible : `Velt-PHP/veltphp-skeleton`

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

Verification :

```bash
php -l bootstrap/app.php
php public/index.php
```

## Issue 2 - Verrouiller velt/ui ^0.1.0 dans le framework

Repository cible : `Velt-PHP/veltphp-framework`

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

Fichiers a verifier :

- `composer.json`
- `README.md`
- documentation de release/compatibilite

Correction dans un projet deja installe :

```bash
composer update velt/framework velt/ui --with-dependencies
composer dump-autoload
```

En developpement local, utiliser un repository `path` vers `../velt-ui` avec la version `0.1.0`.

Verification :

```bash
composer validate
```

## Issue 3 - Regenerer le lock file du skeleton

Repository cible : `Velt-PHP/veltphp-skeleton`

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

Fichiers a verifier :

- `composer.json`
- `composer.lock`
- `vendor/` seulement en local, jamais comme source de verite du repo

Verification :

```bash
composer install
php -r "require 'vendor/autoload.php'; var_dump(class_exists('Velt\\Ui\\Providers\\UiServiceProvider'));"
vendor/bin/phpunit
```

## Issue 4 - Publier velt/ui avec UiServiceProvider

Repository cible : `Velt-PHP/velt-ui`

Symptome :

En local, le path repository fonctionne, mais une installation propre depuis Packagist reprend une combinaison incompatible.

Cause :

Le code local a ete corrige, mais les tags Composer publics ne sont pas alignes.

Correction release :

1. Verifier que `src/Providers/UiServiceProvider.php` existe.
2. Verifier que `composer.json` expose `"Velt\\Ui\\": "src/"`.
3. Tagger et publier `velt/ui` avec `UiServiceProvider` en `0.1.0`.

Fichiers a verifier :

- `src/Providers/UiServiceProvider.php`
- `composer.json`
- tests du provider UI

Verification :

```bash
composer test
php -r "require 'vendor/autoload.php'; var_dump(class_exists('Velt\\Ui\\Providers\\UiServiceProvider'));"
```

## Issue 5 - Verifier la generation CLI

Repository cible : `Velt-PHP/veltphp-cli`

Symptome :

Une application creee avec `velt new` ou une commande equivalente installe encore un skeleton qui pointe vers une mauvaise combinaison de packages.

Cause :

La CLI peut copier un template, telecharger un skeleton tagge, ou embarquer une reference qui n'a pas ete mise a jour apres la correction framework/skeleton.

Correction :

1. Identifier la source exacte utilisee par `velt new`.
2. Verifier que le skeleton genere contient `Velt\Ui\Providers\UiServiceProvider`.
3. Verifier que le projet genere installe `velt/ui >= 0.1.0`.
4. Ajouter ou mettre a jour un test CLI qui cree un projet temporaire et verifie l'autoload du provider.

Fichiers a verifier :

- commandes `new`, `make`, ou templates skeleton
- tests CLI de generation projet
- documentation d'installation

Verification :

```bash
php bin/velt new demo-app
cd demo-app
composer install
php -r "require 'vendor/autoload.php'; var_dump(class_exists('Velt\\Ui\\Providers\\UiServiceProvider'));"
php public/index.php
```

## Verification minimale

Depuis un projet skeleton :

```bash
composer dump-autoload
php -r "require 'vendor/autoload.php'; var_dump(class_exists('Velt\\Ui\\Providers\\UiServiceProvider'));"
php public/index.php
vendor/bin/phpunit
```

Le front controller ne doit plus lever d'exception `Provider class ... does not exist`.
