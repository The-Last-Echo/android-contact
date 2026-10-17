# Versionnement, APK, GitHub Releases et F-Droid

> **État actuel : procédure spécifiée, non automatisée.** Le dépôt est encore au stade du cadrage : aucun projet Gradle compilable, aucune tâche de build vérifiée, aucun identifiant Android définitif, aucune clé de publication configurée et aucun workflow de release actif. Ne pas annoncer une automatisation tant que les workflows n'existent pas et n'ont pas été testés de bout en bout.

## 1. Objectifs

Chaque version distribuée doit être traçable vers un commit source immuable. La release doit utiliser un seul tag et un seul commit pour ses notes, ses APK, ses sommes de contrôle et ses résultats CI. Un échec critique doit empêcher la publication des artefacts.

Les canaux prévus sont :

- **GitHub Releases** : APK préparés par le projet et signés avec la clé de publication GitHub.
- **F-Droid** : construction indépendante depuis les sources accessibles, suivant les recettes, contrôles et signatures F-Droid. Une publication GitHub n'entraîne pas automatiquement une publication F-Droid.

Le projet ne vise pas Google Play pour le moment.

## 2. Politique de versionnement

Suivre [Semantic Versioning 2.0.0](https://semver.org/lang/fr/) avec des tags Git préfixés par `v` :

- version stable : `vMAJOR.MINOR.PATCH`, par exemple `v1.2.3` ;
- préversions : `v0.2.0-alpha.1`, `v0.2.0-beta.1`, `v1.0.0-rc.1` ;
- le préfixe `v` appartient au tag et n'est pas inclus dans `versionName`.

### Quand augmenter la version

- **PATCH** (`x.y.Z`) : correction de bug, correctif de sécurité compatible, correction de régression ou changement interne qui ne modifie pas le contrat public.
- **MINOR** (`x.Y.0`) : fonctionnalité nouvelle compatible avec le comportement existant.
- **MAJOR** (`X.0.0`) : changement incompatible avec les engagements de compatibilité d'une version stable, ou migration majeure annoncée. Avant `1.0.0`, l'API publique est considérée comme en développement ; les ruptures doivent quand même être expliquées dans le changelog et la migration des données doit rester sûre.
- **Préversion** : utiliser un suffixe alpha/beta/rc quand le candidat est distribué pour essais avant la release stable. Ne pas présenter une préversion comme stable.

L'agent doit analyser le dernier tag, `CHANGELOG.md`, les modifications depuis la dernière version, la compatibilité, les changements de données et les fonctionnalités livrées, puis **proposer** le numéro suivant avec une justification. Le propriétaire approuve le numéro et décide si la livraison est prête. Ne pas augmenter la version de release à chaque commit ou fusion.

### Règles immuables des tags

- Chaque release a un tag unique qui correspond à la version de l'application.
- Un tag publié n'est jamais réutilisé, déplacé ni forcé. Toute correction après publication utilise un nouveau numéro.
- Le tag de release doit désigner le commit exact revu, validé et destiné à la release, normalement déjà intégré dans `main`.
- Utiliser un tag annoté ; signer les tags si une méthode de signature a été configurée et testée. Ne pas prétendre qu'un tag est signé lorsqu'il ne l'est pas.
- Les tags alpha, beta et release candidate doivent être identifiés comme préversions sur GitHub lorsqu'ils sont publiés.

## 3. `versionName` et `versionCode` Android

- `versionName` doit correspondre exactement à la version sans `v`, par exemple tag `v0.3.0` → `versionName = "0.3.0"`.
- `versionCode` est un entier qui augmente strictement à chaque nouvelle version distribuée de l'application concernée, y compris les préversions distribuées. Ne jamais le réduire ni le réutiliser pour une nouvelle publication de la même application.
- Ne pas calculer un `versionCode` en concaténant librement les parties SemVer sans en vérifier les limites et les règles de préversion. Définir une stratégie numérique explicite dès le bootstrap et la documenter dans la configuration du build.
- Le workflow release doit extraire la version du tag, comparer `versionName` et vérifier que `versionCode` est supérieur à celui de la dernière release applicable. En cas de tag malformé, divergence ou régression, le workflow échoue sans publier.
- Si la version utilise des flavors, les valeurs et les identifiants des variantes doivent être documentés ; chaque canal d'application doit garder une séquence de versions cohérente.

Référence Android : https://developer.android.com/studio/publish/versioning

## 4. Contenu de chaque GitHub Release

À la demande du propriétaire, chaque tag de version doit déclencher la construction des deux variantes et inclure les deux APK comme assets de GitHub Release une fois les vérifications réussies :

1. **APK release** — artefact officiel destiné aux utilisateurs : optimisé, `debuggable=false`, signé avec la clé permanente de publication du projet.
2. **APK debug** — artefact de test seulement, identifié sans ambiguïté par son nom et sa description. Il ne doit pas être présenté comme le téléchargement normal recommandé.
3. **`SHA256SUMS.txt`** — sommes SHA-256 calculées sur les fichiers finaux après leur création/signature.
4. **Notes de version** — résumant les rubriques réellement livrées de `CHANGELOG.md`, avec les limites connues et la procédure de mise à jour.

Noms recommandés, à adapter aux noms de variantes effectivement créés :

- `Contacts-vX.Y.Z-release.apk`
- `Contacts-vX.Y.Z-debug-testing-only.apk`
- `SHA256SUMS.txt`

Les notes doivent indiquer clairement : version, tag, commit source, niveau stable/préversion, changements majeurs, précautions de migration et canal de téléchargement recommandé. Ne jamais annoncer une fonctionnalité non livrée.

### Signature et variantes debug/release

- La release doit être signée avec le certificat de publication stable réservé à ce projet. Ne pas utiliser le certificat debug comme clé de release.
- Recommander un `applicationIdSuffix = ".debug"` pour la variante debug afin qu'elle s'installe à côté de l'application normale au lieu de la remplacer. Confirmer ce choix lors du bootstrap Gradle.
- Une build debug est destinée à l'essai et peut contenir des informations de diagnostic ou être signée par une clé de développement. Elle doit porter un avertissement « tests uniquement » et ne doit pas être prise pour un artefact de confiance équivalent à la release.
- Si le suffixe debug est utilisé, l'identifiant de paquet, le manifeste et les permissions de la variante debug restent vérifiés ; la variante ne doit pas transporter de contacts réels dans des logs de diagnostic.
- Ne jamais distribuer un artefact dont le statut de signature, la version ou la provenance n'a pas été vérifié.
- Une somme SHA-256 permet de détecter une altération par comparaison à une valeur fiable ; à elle seule, elle ne prouve pas l'identité de l'auteur ni l'authenticité de la source.

## 5. Workflow automatisé attendu

Les workflows réels ne doivent être ajoutés qu'après la création du projet Gradle. Avant de les activer, vérifier chaque tâche avec `./gradlew tasks --all`, tester les variantes en Linux propre et épingler les actions utilisées à des SHA complets vérifiés.

Le processus attendu pour un tag valide est :

1. Le propriétaire approuve la version, le changelog et l'état de préparation.
2. Le code est intégré à `main` par le processus de revue ; le commit de release est propre et validé.
3. Le tag annoté immuable `vX.Y.Z` est créé/poussé avec autorisation du propriétaire.
4. Un workflow déclenché par le tag extrait le code source correspondant exactement au commit tagué.
5. Il exécute les contrôles pertinents : tests, lint, analyse statique, vérification des licences/provenance de dépendances, détection de secrets, vérification du manifeste et cohérence des métadonnées de version.
6. Il construit debug et release à partir du même commit et vérifie leur existence, leur identifiant de paquet, leur `versionName`, leur `versionCode`, les variantes, le champ `debuggable` et la signature attendue.
7. Il vérifie la signature release avec `apksigner` (ou l'outil Android approprié), génère les sommes SHA-256 et prépare les notes de version.
8. Seulement si tous les contrôles requis réussissent, le job de publication crée la GitHub Release liée au tag et téléverse les deux APK ainsi que `SHA256SUMS.txt`.
9. La release est publiée comme préversion pour alpha/beta/rc ; les releases stables ne sont pas marquées préversions.
10. Le rapport final conserve les liens du run CI, du commit, du tag, des assets et des vérifications réellement exécutées.

Un échec critique arrête le processus et ne doit pas produire une GitHub Release présentée comme valide. Ne pas créer une release publique vide ou partiellement vérifiée par automatisme. Les tests et builds de pull request doivent utiliser un `GITHUB_TOKEN` limité en lecture et ne doivent pas avoir accès aux secrets de signature. Le job de publication dispose uniquement des permissions d'écriture nécessaires, idéalement après validation d'un environnement de publication protégé.

Ne pas copier une action GitHub tierce avec un SHA arbitraire : épingler uniquement un SHA complet vérifié sur le dépôt amont officiel. Garder les versions des actions et leurs mises à jour dans la revue de sécurité.

Sources :
- GitHub Releases : https://docs.github.com/repositories/releasing-projects-on-github/about-releases
- Déclenchement sur tag : https://docs.github.com/actions/using-workflows/events-that-trigger-workflows#push
- Durcissement Actions : https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions

## 6. Préparer une release : checklist

- [ ] La tâche à publier est intégrée à `main`, la revue est faite et la CI obligatoire réussit.
- [ ] Le propriétaire a approuvé la version à publier et son niveau (stable/préversion).
- [ ] `CHANGELOG.md` est à jour ; les éléments non terminés restent dans `Unreleased`.
- [ ] Les documents concernés et l'inventaire `THIRD_PARTY.md` sont à jour.
- [ ] Les tests adaptés à la portée de la release ont réussi ; chaque test non exécuté est signalé.
- [ ] Aucun secret, contact réel, artefact local ni donnée personnelle n'est ajouté au commit.
- [ ] Les licences, ressources, transitives et provenances sont vérifiées.
- [ ] La clé release est disponible au processus autorisé et sa sauvegarde hors ligne est vérifiée sans exposer la clé.
- [ ] Le tag est unique, pointe vers le commit attendu et correspond à `versionName`.
- [ ] `versionCode` progresse correctement par rapport à la dernière version distribuée.
- [ ] Les APK debug et release ont été générés depuis ce commit et leurs identifiants/versions sont corrects.
- [ ] La release APK n'est pas debuggable et sa signature correspond au certificat attendu.
- [ ] `SHA256SUMS.txt` porte sur les fichiers finaux qui seront téléversés.
- [ ] Les notes de version expliquent les changements, les limites, les changements incompatibles et les risques de migration.
- [ ] La GitHub Release n'est publiée qu'après réussite des contrôles de CI.

## 7. Clé de signature et récupération

La clé de publication ne doit jamais être committée, stockée dans les artefacts, affichée dans les logs ou partagée dans une conversation. Utiliser un stockage de secrets adapté pour le job de release et restreindre ce job à des tags/commits approuvés. Garder une sauvegarde chiffrée hors ligne, des instructions de récupération accessibles au propriétaire et une procédure testée sans publier de secret.

La perte de la clé peut empêcher les mises à jour en place pour les personnes qui ont installé un APK GitHub signé avec cette clé. Ne pas créer une nouvelle clé pour chaque release. Ne jamais changer de clé sans plan de migration spécifique et information des utilisateurs.

## 8. GitHub Releases et F-Droid : canaux distincts

La fiche F-Droid est un processus séparé. Il faudra préparer les métadonnées/recette de build selon la version actuelle de `fdroiddata`, soumettre le projet et répondre à la revue. La CI GitHub ne met pas automatiquement à jour F-Droid.

- Les sources de la version doivent être publiées, accessibles et construire sans dépendances binaires opaques.
- Vérifier la politique d'inclusion et la reproductibilité depuis les sources au moment de la soumission et lors des changements de toolchain.
- Les APK GitHub et F-Droid peuvent porter des signatures différentes. Android refuse généralement une mise à jour en place entre signatures incompatibles, même lorsque l'`applicationId` est identique ; un changement de canal peut demander une désinstallation et présenter un risque pour les données locales.
- Le README et les notes de release doivent expliquer les différences de canaux et indiquer le chemin d'installation à privilégier.
- Ne jamais annoncer « disponible sur F-Droid » avant que l'entrée officielle soit acceptée et visible dans le dépôt.

Sources :
- Politique F-Droid : https://f-droid.org/docs/Inclusion_Policy/
- Builds reproductibles : https://f-droid.org/docs/Reproducible_Builds/
- Traitement des mises à jour : https://f-droid.org/en/docs/Update_Processing/
- Signature Android : https://developer.android.com/studio/publish/app-signing
- Versionnement Android : https://developer.android.com/studio/publish/versioning

## 9. État actuel et décisions encore ouvertes

- `applicationId` et nom de paquet : à définir avant le premier build distribué.
- `minSdk`, `compileSdk`, `targetSdk`, Kotlin, AGP, Gradle et JDK : à sélectionner selon la documentation officielle actuelle et la compatibilité réelle de la toolchain au moment du bootstrap.
- Tâches Gradle exactes de tests/lint/build : non disponibles tant que le projet Android n'existe pas.
- Clé release, emplacement des secrets CI et sauvegarde de récupération : à configurer avant la première release signée.
- Workflow CI et workflow release : spécifiés ici, mais non encore implémentés ni vérifiés.
- Intégration F-Droid : future soumission, sans garantie d'acceptation.
