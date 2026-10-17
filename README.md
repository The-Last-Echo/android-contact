# Contacts — nom de travail

Application Android libre de gestion de contacts, conçue pour être rapide, claire, accessible et respectueuse de la vie privée.

> **État du projet : cadrage initial.** Le dépôt ne contient pas encore une application Android compilable. Les fonctionnalités et processus décrits ici sont des exigences ou des plans, pas la preuve qu'ils sont déjà implémentés. Aucun APK n'est actuellement annoncé comme disponible.

## Objectifs

- Afficher, rechercher, consulter, créer, modifier et supprimer des contacts sans modifier accidentellement les données d'un autre compte.
- Proposer des actions explicites pour appeler, envoyer un SMS ou rédiger un e-mail via une application système, sans déclencher l'action automatiquement.
- Gérer les photos et les champs usuels, ainsi que l'import/export vCard (`.vcf`).
- Prévoir une synchronisation directe CardDAV multi-comptes, sans rendre DAVx⁵ obligatoire.
- Utiliser Kotlin et Jetpack Compose, avec le système visuel Soft Motion adapté aux besoins réels de l'application.
- Localiser l'interface dès le départ, prendre en charge l'accessibilité et rester utilisable hors ligne pour les données locales.
- N'inclure ni publicité ni télémétrie cachée. Aucun contact ne doit être transmis à un service tiers non explicitement choisi.

## État des fonctionnalités

Aucune fonctionnalité ne doit être considérée comme implémentée tant qu'elle n'existe pas dans le code et que ses critères d'acceptation ne sont pas vérifiés. Voir [`docs/PRODUCT.md`](docs/PRODUCT.md), [`docs/TESTING.md`](docs/TESTING.md) et [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

## Distribution prévue

- Sources : dépôt public GitHub.
- Versions téléchargeables : GitHub Releases, lorsque la compilation, les tests et la signature de publication seront opérationnels.
- Distribution depuis les sources : candidature au dépôt principal F-Droid, sous réserve de conformité et d'acceptation. La publication GitHub ne publie pas automatiquement l'application sur F-Droid.

**Aucun APK n'est disponible à ce stade.** Quand les releases seront activées, chaque release GitHub devra correspondre à un tag de version immuable `vX.Y.Z` ou à une préversion explicite. Les assets attendus et les contrôles sont définis dans [`docs/RELEASING.md`](docs/RELEASING.md).

Les builds GitHub et F-Droid peuvent être signés avec des certificats différents et ne pas pouvoir se mettre à jour mutuellement. Lire la procédure de distribution avant de changer de canal ; une désinstallation peut entraîner une perte de données locales.

## Développement

Les conventions de branches, de pull requests, de CI et de validation sont définies dans [`docs/DEVELOPMENT_WORKFLOW.md`](docs/DEVELOPMENT_WORKFLOW.md). Les commandes Gradle ne seront documentées qu'après création du projet et vérification des tâches réellement disponibles. Ne pas inventer des commandes de build.

## Licence

Le projet vise la GNU General Public License version 3 **uniquement** (`GPL-3.0-only`). Voir [`LICENSE`](LICENSE). Chaque bibliothèque, police, icône, illustration et ressource embarquée doit être vérifiée, licenciée pour son usage prévu et recensée dans [`THIRD_PARTY.md`](THIRD_PARTY.md).

## Confidentialité et sécurité

Les contacts restent dans la source de stockage choisie sur l'appareil, sauf export explicite ou synchronisation vers un compte CardDAV configuré par la personne qui utilise l'application. Les engagements de confidentialité doivent être vérifiés dans le manifeste fusionné, le code, les dépendances, les logs et le comportement réseau avant une release. Voir [`docs/PRIVACY.md`](docs/PRIVACY.md) et [`SECURITY.md`](SECURITY.md).

## Documentation du projet

- [`docs/PROJECT_PROFILE.md`](docs/PROJECT_PROFILE.md) — état du projet, décisions confirmées et décisions ouvertes.
- [`docs/PRODUCT.md`](docs/PRODUCT.md) — objectifs fonctionnels et critères d'acceptation.
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — modèle de contacts et architecture.
- [`docs/CARDDAV_SYNC.md`](docs/CARDDAV_SYNC.md) — exigences de synchronisation des contacts.
- [`docs/PRIVACY.md`](docs/PRIVACY.md) — flux de données et garanties attendues.
- [`DESIGN.md`](DESIGN.md) — système visuel Soft Motion et configuration de cette application.
- [`SECURITY.md`](SECURITY.md) — sécurité de l'application, des données et du processus de publication.
- [`THIRD_PARTY.md`](THIRD_PARTY.md) — inventaire et audit des dépendances et ressources tierces.
- [`docs/TESTING.md`](docs/TESTING.md) — stratégie de test et critères de livraison.
- [`docs/DEVELOPMENT_WORKFLOW.md`](docs/DEVELOPMENT_WORKFLOW.md) — branches, revues et CI.
- [`docs/RELEASING.md`](docs/RELEASING.md) — versionnement, tags, signatures, APK et GitHub Releases/F-Droid.
- [`docs/TECHNICAL_RESEARCH.md`](docs/TECHNICAL_RESEARCH.md) — notes techniques datées et sources à revérifier.
- [`CHANGELOG.md`](CHANGELOG.md) — historique des changements effectivement livrés.

## Licence et statut de publication

Les documents de conception ne remplacent ni les tests, ni une revue du code, ni l'approbation du propriétaire. Une fonctionnalité, un contrôle de sécurité ou une publication ne doit jamais être déclaré réussi sans preuve vérifiable.
