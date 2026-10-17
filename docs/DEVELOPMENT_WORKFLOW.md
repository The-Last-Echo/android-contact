# Workflow de développement — Contacts

Ce document décrit le mode de travail destiné aux contributeurs humains et aux agents IA. Il complète `AGENTS.md`, qui reste local et exclu du dépôt public. Les règles de ce document ne prétendent pas activer elles-mêmes les protections GitHub : les paramètres de branches et les workflows doivent être configurés séparément.

## 1. Principes

- `main` est la branche stable : chaque commit intégré doit être testé et, une fois le projet compilable, réussir la CI requise.
- Ne jamais développer directement sur `main`.
- Une branche représente une tâche cohérente, pas une modification par fichier. Éviter les branches longues qui regroupent de nombreux objectifs sans relation.
- Ne fusionner une tâche qu'après vérification du résultat, des tests et du diff complet.
- Ne pas attendre la fin du projet pour fusionner toutes les branches en une seule fois.
- L'IA ne doit jamais déclarer un test réussi si elle ne l'a pas réellement exécuté et inspecté.
- Aucun changement utilisateur préexistant ne doit être perdu ou écrasé.

## 2. Noms des branches

Utiliser des noms simples et descriptifs :

- `feat/contact-search`
- `fix/contact-photo-loading`
- `test/vcard-import`
- `docs/release-process`
- `build/gradle-bootstrap`
- `security/release-signing`

Une branche doit contenir des commits suffisamment petits pour être relus, mais pas des commits artificiellement fragmentés. Chaque commit doit décrire son intention (`feat:`, `fix:`, `docs:`, `test:`, `build:`, `security:`).

## 3. Début d'une tâche

1. Examiner `git status`, la branche active, les derniers commits et le diff existant.
2. Confirmer que `main` est propre et à jour avant de créer une branche, sans supprimer les changements préexistants.
3. Lire les documents pertinents : au minimum `PROJECT_PROFILE.md`, `PRODUCT.md`, `ARCHITECTURE.md`, `TESTING.md` et `SECURITY.md` selon le sujet.
4. Vérifier les décisions déjà approuvées. Si la tâche nécessite un choix produit ou une modification potentiellement destructive, demander l'approbation du propriétaire avant de le traiter comme décidé.
5. Créer une branche pour cette tâche avant de modifier les fichiers.

## 4. Pendant le développement

- Garder le diff centré sur l'objectif ; éviter les refactorisations sans rapport.
- Ajouter ou mettre à jour les tests au même moment que le code.
- Respecter les couches définies dans `docs/ARCHITECTURE.md`. L'UI ne doit pas gérer directement le réseau, les identifiants ni les accès de bas niveau aux contacts.
- Mettre à jour `CHANGELOG.md` dans `Unreleased` pour les changements notables réellement apportés.
- Mettre à jour la documentation concernée lorsque le comportement, l'architecture, les dépendances, la sécurité, les permissions ou le processus de publication changent.
- Examiner les dépendances et ressources ajoutées selon `THIRD_PARTY.md`.
- Ne jamais introduire de secrets, données personnelles réelles ou contacts réels dans les tests et fixtures.
- Ne pas augmenter la version publique pour chaque commit ou pull request : le versionnement suit `docs/RELEASING.md` au moment de préparer une livraison.

## 5. Validation avant revue

Exécuter les commandes de validation réellement disponibles dans le projet. Après bootstrap, ces commandes seront inscrites dans `docs/TESTING.md`.

Avant de demander la fusion :

- [ ] Les tests pertinents ont été exécutés et leurs résultats contrôlés.
- [ ] Le build et Lint ont réussi si les tâches existent et sont requises.
- [ ] Le diff complet a été relu ; aucun fichier ni changement hors sujet n'est inclus.
- [ ] Aucun secret, token, contact réel, log personnel ou artefact local ne se trouve dans le diff.
- [ ] Les dépendances et licences ont été examinées.
- [ ] Le changelog et les documents affectés ont été mis à jour.
- [ ] Les limites et vérifications non exécutées sont explicitement signalées.

Si une vérification ne peut pas s'exécuter dans l'environnement disponible, expliquer pourquoi et ne pas la déclarer réussie.

## 6. Fusion

- Utiliser une pull request pour les changements soumis via GitHub, avec description, justification, risques et résultats de tests.
- Les contrôles CI doivent réussir avant la fusion lorsque le dépôt et les workflows les prennent en charge.
- Le propriétaire doit relire le diff et approuver toute fusion dans `main`. Aucun agent ne doit fusionner de lui-même sans autorisation explicite.
- Ne jamais forcer un push, déplacer un tag publié ou réécrire l'historique partagé.
- Après fusion, vérifier que `main` correspond au résultat attendu et reste propre.
- Supprimer les branches terminées seulement après avoir vérifié que leur contenu est bien intégré.

## 7. Automatisation GitHub à mettre en place après le bootstrap

Quand le projet est compilable :

1. Ajouter un workflow CI pour les pull requests et les pushs sur les branches pertinentes. Il utilise les tâches réelles vérifiées avec le Gradle Wrapper.
2. Activer les règles de protection de `main` : pull request obligatoire, contrôles CI obligatoires, interdiction du force-push et de la suppression de la branche.
3. Configurer le workflow release distinct décrit dans `docs/RELEASING.md`.
4. Restreindre les permissions des workflows ; aucun secret de publication dans les builds de pull requests non fiables.
5. Épingler les actions tierces à des SHA complets vérifiés lorsque possible.

Avant que ces étapes soient réellement réalisées et testées, la documentation doit dire qu'elles sont planifiées, pas qu'elles sont actives.

## Références

- GitHub — protection des branches : https://docs.github.com/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches
- GitHub Actions — sécurisation des workflows : https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions
- Gradle — Wrapper : https://docs.gradle.org/current/userguide/gradle_wrapper.html
