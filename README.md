# Contacts — nom de travail

Application Android de gestion de contacts, conçue pour être rapide, claire, personnalisable et respectueuse de la vie privée.

> **État : cadrage initial.** Le nom définitif, l'identifiant Android, la version minimale d'Android et plusieurs décisions d'architecture restent à confirmer. Ce dépôt ne contient pas encore une application compilable.

## Objectifs

- Afficher, rechercher, consulter, créer, modifier et supprimer des contacts.
- Proposer des actions explicites pour appeler, envoyer un SMS ou rédiger un e-mail, sans déclencher l'action automatiquement.
- Gérer les photos de contact, les champs usuels et les importations/exportations vCard (`.vcf`).
- Prévoir une synchronisation CardDAV directe avec plusieurs comptes, sans rendre un service externe obligatoire.
- Fournir une interface Android moderne fondée sur Kotlin et Jetpack Compose, cohérente avec le système de design Soft Motion.
- Être traduisible dès le début, utilisable hors ligne pour les données déjà disponibles, sans publicité ni télémétrie cachée.

## État des fonctionnalités

Aucune fonctionnalité n'est déclarée implémentée à ce stade. Les spécifications de `docs/PRODUCT.md` décrivent les objectifs et phases envisagés, et non un état d'avancement du code.

## Distribution prévue

- Sources : dépôt public GitHub.
- Paquets Android signés : GitHub Releases.
- Distribution depuis les sources : candidature au dépôt principal F-Droid, sous réserve de conformité et d'acceptation.

Les APK provenant de GitHub et de F-Droid peuvent être signés par des clés différentes et ne pas pouvoir se mettre à jour l'un l'autre sans désinstallation. Lire `docs/RELEASING.md` avant de choisir le canal d'installation.

## Licence

Le projet vise la GNU General Public License version 3 **uniquement** (`GPL-3.0-only`). Voir `LICENSE`. Chaque bibliothèque, police, icône, illustration et ressource embarquée doit être vérifiée et consignée dans `THIRD_PARTY.md`.

## Développement

Les commandes de compilation ne sont pas encore renseignées, car les fichiers Gradle et les versions Android n'ont pas été choisis. Elles devront être testées sous Linux puis documentées ici après création du projet compilable.

## Documentation

- `docs/PROJECT_PROFILE.md` — faits du projet et décisions à prendre.
- `docs/PRODUCT.md` — fonctionnalités attendues et critères d'acceptation.
- `docs/ARCHITECTURE.md` — options et architecture à confirmer après comparaison.
- `docs/CARDDAV_SYNC.md` — portée et exigences de synchronisation des contacts.
- `docs/PRIVACY.md` — flux de données et garanties de confidentialité attendues.
- `DESIGN.md` — système visuel Soft Motion et configuration de cette application.
- `SECURITY.md` — règles de sécurité et signalement des vulnérabilités.
- `THIRD_PARTY.md` — inventaire des dépendances et ressources tierces.
- `docs/TESTING.md` — plan de validation.
- `docs/RELEASING.md` — publication GitHub et préparation F-Droid.
- `docs/TECHNICAL_RESEARCH.md` — références techniques ayant guidé le cadrage.

## Confidentialité

Les contacts ne doivent être transmis qu'aux comptes CardDAV que l'utilisateur configure explicitement et après une explication claire de la synchronisation. Aucun service tiers, suivi, publicité ou transfert caché n'est prévu. Ces engagements devront être vérifiés dans le manifeste, le code, les dépendances et les tests avant chaque publication.
