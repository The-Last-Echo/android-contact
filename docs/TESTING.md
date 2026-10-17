# Stratégie de tests et critères de qualité

> Les tâches Gradle et versions exactes seront ajoutées après création du projet. Ne pas inventer des tâches qui n'existent pas ; vérifier les tâches avec le Wrapper.

## 1. Tests obligatoires par zone

### Contacts et données

- Liste vide, contacts sans prénom/nom, accents, alphabets différents, plusieurs numéros/e-mails, champs manquants et entrées malformées.
- Recherche selon les champs décidés ; normalisation Unicode/accents et espaces, ponctuation des numéros, casse des e-mails selon la stratégie retenue.
- Affichage des sources, de contacts agrégés et de plusieurs RawContacts si Contacts Provider est utilisé.
- Création, mise à jour et suppression ; refus/révocation des permissions ; contact d'un compte non modifiable.
- Photos absentes, images très grandes, URI inaccessible et rotation/pression mémoire.

### Actions système

- Aucun gestionnaire de téléphonie/SMS/e-mail.
- URI `tel:`, `smsto:` et `mailto:` construites correctement et caractères échappés.
- Le système cible s'ouvre avec les données attendues, sans déclencher d'envoi automatique.

### vCard import/export

- Plusieurs fiches dans un même `.vcf`, encodage UTF-8, lignes repliées, retours chariot, champs optionnels et versions prises en charge.
- Champs inconnus conservés lorsque possible ou signalés plutôt que supprimés silencieusement.
- Fichier vide, excessivement grand, malformé ou contenant une image volumineuse ; l'import ne doit pas laisser une demi-importation non signalée.
- Round-trip des champs principaux à travers export puis import.

### CardDAV (avant toute version qui l'active)

- Découverte, redirects, collection vide, pagination/multi-status, ETag/sync-token, premières synchronisations et suivantes.
- Plusieurs comptes isolés, mauvaise authentification, certificat non valide, serveur hors ligne, réponse invalide, timeout, 401/403, 404, 409, 412 et 5xx.
- Modification locale et distante concurrentes, suppressions de chaque côté, erreur en milieu de sync et répétition sans doublons.
- Un échec réseau ou une réponse incomplète ne doit jamais effacer le carnet local.
- Serveur simulé ou carnet de test jetable ; pas de contacts réels dans les fixtures ou journaux.

### Interface et accessibilité

- Tests Compose UI et sémantiques pour recherche, navigation, formulaire, confirmation de suppression et erreurs.
- TalkBack, ordre de focus, clavier/D-pad lorsque pertinent, taille de texte élevée, thème clair/sombre, RTL et chaînes longues.
- Écrans compacts et largeurs adaptées si la tablette/les fenêtres étendues sont prises en charge.
- Chaque chaîne de l'interface vient des ressources localisées ; vérifier les langues réellement livrées.

## 2. Performance

- Utiliser des listes paresseuses pour les longues listes ; éviter de composer et charger toutes les photos/fiche détaillées à chaque affichage.
- Préparer un jeu de test synthétique de 1 000 et 10 000 contacts, sans données personnelles réelles.
- Mesurer le temps de première liste, le filtrage, le défilement, la mémoire des miniatures et le retour depuis une fiche ; enregistrer les résultats de référence avant les optimisations.
- Ne pas fixer de chiffres arbitraires avant d'avoir un appareil et une configuration de référence. Définir ensuite des seuils adaptés, puis les maintenir.
- Utiliser les outils de profilage Android et envisager Macrobenchmark/Baseline Profiles si les mesures indiquent que le chemin critique le justifie.

## 3. Validation automatisée

Après création du projet, vérifier `./gradlew tasks --all`, puis remplir les commandes réelles correspondant à :

- tests unitaires ;
- tests instrumentés/UI si configurés ;
- lint et analyse statique ;
- compilation de debug ;
- compilation release ;
- vérification Gradle des dépendances et inventaire/licences ;
- génération de l'APK de version.

Chaque version candidate doit compiler sous Linux en environnement propre. Toutes les vérifications non exécutées doivent être signalées comme telles.

## 4. Critères avant livraison

- Aucun blocage critique ou perte de contact reproductible.
- Permissions minimales et manifeste revu.
- Pas de fuite de données personnelles dans les logs ou le réseau.
- Licences/transitives et ressources documentées.
- Tests adaptés aux fonctionnalités incluses.
- Scénarios d'import/export/synchronisation validés avant d'activer ces fonctions.
- Git diff propre, documentation et changelog à jour.
