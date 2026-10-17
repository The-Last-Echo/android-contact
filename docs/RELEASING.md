# Compilation, GitHub Releases et F-Droid

## État

Aucun workflow de publication n'est ajouté à ce stade. L'application n'a pas encore de fichiers Gradle, d'identifiant définitif, de tâche de release ou de clé de signature configurée. Le workflow doit être créé après validation du build et des commandes de test réelles.

## GitHub Releases

À mettre en place une fois le projet compilable :

1. Utiliser des tags versionnés et un `versionCode` strictement croissant.
2. Sur tag signé/validé, exécuter les tests, lint, vérification des dépendances et compilation release sous Linux.
3. Publier un APK identifié, avec SHA-256, notes de version, commit source et résultat des vérifications.
4. Limiter les permissions du workflow au strict nécessaire ; épingler les actions de marketplace à un SHA commit validé ; ne jamais exécuter la publication depuis une pull request non fiable.
5. Les secrets de signature et mots de passe doivent rester dans un magasin de secrets approprié de la CI ou sur la machine du responsable, jamais dans Git, les logs ni les artefacts intermédiaires.
6. Ne publier aucun APK si la compilation ou une vérification critique échoue.

La signature de publication et sa récupération doivent être documentées et testées ; la perte de la clé peut empêcher les mises à jour en place pour les personnes utilisant les APK signés par cette clé.

## Attention aux signatures GitHub et F-Droid

La distribution officielle F-Droid construit généralement depuis les sources et signe l'APK de son côté. Les APK GitHub signés avec la clé du mainteneur auront donc normalement une signature différente. Android refuse la mise à jour en place entre deux APK ayant le même identifiant mais des certificats de signature incompatibles ; l'utilisateur pourrait devoir désinstaller avant de changer de canal, avec risque de perte des données locales.

Avant la première version publique :

- choisir la stratégie de signature côté GitHub ;
- expliquer clairement dans la documentation que les deux canaux peuvent ne pas être interchangeables ;
- tester installation, mise à jour dans chaque canal, puis expliquer les limites du changement de canal ;
- ne jamais suggérer que GitHub et F-Droid partagent automatiquement la même signature.

## F-Droid

- Confirmer que le projet, toutes ses dépendances transitives, ses ressources et sa toolchain sont conformes à la politique en vigueur.
- Privilégier une construction depuis les sources publiques, sans SDK propriétaire, clé API privée obligatoire, dépendance à Google Play Services ou téléchargement opaque d'exécutables.
- Vérifier le caractère reproductible du build et documenter les différences si nécessaire.
- Préparer les métadonnées pour le dépôt de recettes F-Droid (généralement `fdroiddata` pour le dépôt principal) et suivre la procédure de contribution/validation actuelle.
- Ne pas annoncer « disponible sur F-Droid » avant l'acceptation et la présence effective de la fiche.
- Une publication GitHub automatique ne met pas automatiquement à jour la fiche ou la construction F-Droid ; prévoir séparément les métadonnées et l'actualisation des recettes.

## Décisions à renseigner

- Identifiant Android permanent : à définir avant la première version publique.
- Clé de signature GitHub : responsable, sauvegarde hors ligne et procédure de récupération à définir.
- Tâches Gradle de test/lint/release : découvrir et tester.
- Mode exact de signature/automatisation : à choisir.
- Dépôt/issue de métadonnées F-Droid et stratégie de maintien : à définir après la première version construite.
- Notes de version et fichiers de somme de contrôle : à automatiser.

## Sources

- Politique F-Droid : https://f-droid.org/docs/Inclusion_Policy/
- Reproductibilité F-Droid : https://f-droid.org/docs/Reproducible_Builds/
- Métadonnées et mises à jour F-Droid : https://f-droid.org/en/docs/Update_Processing/
- GitHub Actions : https://docs.github.com/en/actions
- Durcissement des workflows : https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions
