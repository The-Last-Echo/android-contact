# Fiche projet — Contacts

> État : cadrage initial. Les éléments marqués « proposé » ne sont pas des faits d'implémentation. Mettre à jour ce document lorsqu'une décision est approuvée et relier la décision à son détail.

## Identité et contraintes

| Champ | Valeur | État |
|---|---|---|
| Nom du projet | Contacts — nom de travail | Nom définitif à choisir |
| `applicationId` | À définir avant la première version distribuée | Bloquant avant publication ; difficile à changer ensuite |
| Plateforme | Android uniquement | Confirmé |
| Langage/UI | Kotlin + Jetpack Compose par défaut | Confirmé |
| Système visuel | Soft Motion, sous-ensemble choisi pour l'application | Confirmé ; sélection des composants à finaliser pendant la conception UI |
| Licence | `GPL-3.0-only` | Confirmé |
| Distribution | GitHub Releases et candidature F-Droid | Objectif confirmé ; acceptation F-Droid non garantie |
| Environnement | Linux | Confirmé |
| Publicité/télémétrie | Aucune publicité ni télémétrie cachée | Confirmé |
| Langues initiales | Anglais source/fallback et français traduit | Proposition à valider au bootstrap |
| `minSdk` | À rechercher et proposer | À approuver avant bootstrap |
| `compileSdk` / `targetSdk` | Versions stables vérifiées dans les sources officielles au bootstrap | Ne pas recopier une ancienne valeur ni présenter une recommandation comme un fait permanent |
| Kotlin/AGP/Gradle/JDK | À sélectionner ensemble, puis verrouiller | À approuver ; choisir une matrice compatible et actuelle |
| Canal privé de sécurité | À configurer et tester | Bloquant avant annonce publique d'une procédure de signalement |

## Produit

Gestionnaire de contacts Android léger, agréable, accessible et personnalisable ; gestion locale des contacts, actions système, import/export vCard, et synchronisation directe CardDAV multi-comptes à terme.

## Modèle de données et intégrations

- Source de contacts : Contacts Provider Android ou base Room privée ; décision d'architecture encore à approuver.
- Synchronisation : CardDAV direct, plusieurs comptes, activation explicite par l'utilisateur.
- Import/export : `.vcf` via Storage Access Framework, sans permission générale de stockage si elle n'est pas nécessaire.
- Appels/SMS/e-mails : intents système, pas d'envoi automatique.
- Réseau : aucune requête sans une fonction explicitement activée et une destination connue.
- Les permissions, les capacités de modification réelles et l'appartenance aux comptes doivent être vérifiées avant d'éditer ou de supprimer un RawContact ; un seul indicateur de lecture seule ne garantit pas à lui seul le succès d'une écriture.

## Versionnement et distribution — exigences du propriétaire

| Sujet | Règle | État |
|---|---|---|
| Version publique | Semantic Versioning `X.Y.Z`; tags `vX.Y.Z` | Exigence documentaire à appliquer dès la première version |
| Préversions | Tags explicites comme `v0.1.0-alpha.1`, `v0.1.0-beta.1`, `v1.0.0-rc.1` | Autorisées quand une préversion est réellement distribuée |
| `versionName` | Correspond à `X.Y.Z` sans le `v` du tag | À vérifier automatiquement dans le build |
| `versionCode` | Entier monotone strictement croissant pour chaque nouvelle version distribuée de l'application | À décider et appliquer dans le build ; une vérification doit bloquer les régressions |
| Release GitHub | À partir d'un tag immuable, après tests et approbation | Exigence ; workflow non implémenté actuellement |
| APK release | Fichier de téléchargement officiel, optimisé et signé par la clé GitHub Release stable | Exigence pour les releases GitHub ; clé et secrets à configurer en sécurité |
| APK debug | Reconstruit à chaque tag et joint à chaque GitHub Release comme fichier exclusivement destiné aux tests, avec nom et avertissement explicites | Demande du propriétaire ; suffixe `applicationId` `.debug` recommandé pour éviter les conflits avec la release |
| Contrôles de release | Tests, lint, vérifications d'identité/version, signature, sommes SHA-256 et état de CI | Exigence ; commandes exactes à établir après création du build |
| Clé de signature | Jamais dans Git ; accès limité au processus de release, sauvegarde hors ligne protégée et procédure de récupération | À mettre en place avant la première release signée |
| F-Droid | Construction depuis les sources et chaîne séparée de GitHub ; signatures potentiellement différentes | Objectif ; conformité et acceptation à démontrer |

La génération des APK et de la GitHub Release n'est pas encore automatique. Ne jamais annoncer le contraire avant d'avoir créé, testé et audité les workflows réels.

## Organisation Git et agents IA

- `main` doit rester compilable et vérifiée dès que le projet possède un build.
- Une branche par tâche cohérente ; pas de travail direct sur `main`.
- Toute fusion doit suivre les contrôles automatiques disponibles et une revue du propriétaire.
- L'IA ne doit pas pousser de force, réécrire un historique partagé, créer/pousser un tag de release ou fusionner dans `main` sans autorisation explicite.
- Les fichiers `AGENTS.md` et `.github/copilot-instructions.md` sont locaux et doivent rester hors du suivi Git via `.git/info/exclude`.
- Les conventions publiques destinées aux contributeurs sont dans `docs/DEVELOPMENT_WORKFLOW.md`.

## Décisions d'architecture à approuver

| Sujet | État | Proposition actuelle | Pourquoi c'est important |
|---|---|---|---|
| Modèle contacts | À approuver | Étudier en premier le Contacts Provider | Visibilité système, permissions, propriété et risque de suppressions |
| Synchronisation | À approuver | Compte Android + mécanisme de synchronisation dédié si viable | Multi-comptes et intégration Android, mais davantage de complexité |
| Contacts de tiers | À décider | Montrer la source et n'éditer que les enregistrements dont la modification est autorisée | Éviter d'altérer les comptes externes |
| Import/export vCard | Prévu | Prioriser après choix du modèle de données | Portabilité et sauvegarde |
| Groupes/étiquettes | Phase ultérieure probable | Définir la correspondance avec groupes Android et `CATEGORIES` vCard | Les groupes peuvent dépendre du compte ou serveur |
| Authentification CardDAV | À rechercher | Choisir après sélection des serveurs cibles | Interopérabilité et sécurité des identifiants |
| Conflits CardDAV | À définir avant sync bidirectionnelle | Préserver les données et éviter tout écrasement silencieux | Prévention de pertes |
| Identifiant et nom public | À décider | Stabiliser avant la première release | Identité durable du paquet et de l'application |

## Documentation de référence

- `README.md` — présentation, statut, distribution et index documentaire.
- `docs/PRODUCT.md` — exigences produit.
- `docs/ARCHITECTURE.md` — modèle de contacts et architecture.
- `docs/CARDDAV_SYNC.md` — protocole et sécurité de synchronisation.
- `docs/PRIVACY.md` — flux de données.
- `DESIGN.md` — système visuel.
- `SECURITY.md` — sécurité de l'app et de la release.
- `THIRD_PARTY.md` — dépendances et ressources.
- `docs/TESTING.md` — tests et critères de qualité.
- `docs/DEVELOPMENT_WORKFLOW.md` — branches, revues et CI.
- `docs/RELEASING.md` — SemVer, tags, APK et GitHub Releases/F-Droid.
- `docs/TECHNICAL_RESEARCH.md` — résultats de recherche datés.
- `CHANGELOG.md` — changements livrés.
