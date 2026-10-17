# Stratégie de tests et critères de qualité

> Le projet est en cadrage et n'a pas encore de build Android. Les commandes exactes seront ajoutées après création du projet. Ne pas inventer des tâches Gradle : vérifier leur existence avec le Wrapper (`./gradlew tasks --all`).

## 1. Tests par domaine

### Contacts et données

- Liste vide, contacts sans prénom/nom, accents, scripts différents, plusieurs numéros/e-mails, champs manquants et entrées malformées.
- Recherche selon les champs et la normalisation réellement décidés : Unicode/accents, espaces, ponctuation de numéros et casse des e-mails.
- Affichage de la source et des contacts agrégés/multiples RawContacts si Contacts Provider est retenu.
- Création, modification et suppression strictement limitées aux enregistrements autorisés ; refus/révocation de permissions ; comptes non modifiables ; échec d'écriture correctement présenté.
- Photos absentes, grandes images, URI inaccessible, rotation et pression mémoire.
- Aucune modification ou suppression involontaire d'un autre RawContact du même contact agrégé.

### Actions système

- Aucun gestionnaire de téléphone/SMS/e-mail installé.
- Construction correcte des URI `tel:`, `smsto:` et `mailto:` et échappement des caractères.
- L'application système cible s'ouvre avec les informations choisies ; pas d'envoi automatique.

### vCard import/export

- Plusieurs fiches dans un `.vcf`, UTF-8, lignes repliées, CRLF, champs optionnels, propriétés inconnues et versions supportées.
- Champs inconnus conservés lorsque possible ou signalés plutôt que supprimés silencieusement.
- Fichier vide, trop grand, malformé ou contenant une photo volumineuse ; aucun import partiel non signalé.
- Round-trip des champs principaux à travers export puis import.

### CardDAV — avant toute version qui l'active

- Découverte, redirections, collection vide, pagination/multi-status, ETag/sync-token, première synchronisation et synchronisations suivantes.
- Plusieurs comptes isolés, mauvaise authentification, certificat invalide, hors ligne, réponses invalides, délais expirés, codes 401/403/404/409/412/5xx.
- Modifications concurrentes, suppressions locales/distantes, échec en cours de sync et répétition sans doublons.
- Une erreur réseau ou une réponse incomplète ne doit jamais effacer le carnet local.
- Serveur simulé ou carnet de test jetable ; aucune donnée de contact réelle dans les fixtures ou logs.

### Interface et accessibilité

- Tests Compose UI et sémantiques de la recherche, de la navigation, des formulaires, de la confirmation de suppression et des erreurs.
- TalkBack, ordre de focus, clavier/D-pad si pertinent, grande taille de texte, thème clair/sombre, RTL et chaînes longues.
- Largeurs compactes, téléphones et fenêtres étendues si ces configurations sont prises en charge.
- Chaque chaîne visible vient des ressources localisées ; vérifier les langues réellement livrées.

## 2. Performance

- Utiliser des listes paresseuses pour les longues listes ; éviter de charger les fiches détaillées et photos en pleine résolution pour chaque ligne.
- Préparer un corpus synthétique de 1 000 puis 10 000 contacts, sans données personnelles réelles.
- Mesurer temps de première liste, filtrage, défilement, mémoire/cache des miniatures et retour de la fiche.
- Définir les seuils après avoir sélectionné et documenté une configuration de référence ; ne pas inventer des métriques prétendument mesurées.
- Utiliser les outils de profilage Android et envisager Macrobenchmark/Baseline Profiles uniquement si les mesures les justifient.

## 3. Validation automatisée du code

Quand Gradle est présent, vérifier d'abord les tâches disponibles puis documenter les commandes exactes pour :

- tests unitaires ;
- tests instrumentés/UI si configurés ;
- Lint et analyse statique ;
- compilation debug et release ;
- vérification des dépendances, inventaire des licences et recherche de secrets ;
- génération des rapports de test et APK.

Chaque tâche doit s'exécuter sous Linux dans un environnement propre. Tout contrôle non exécuté doit être déclaré « non exécuté », jamais « réussi » par supposition.

## 4. CI et pull requests

Après le bootstrap, la CI requise pour les branches et pull requests doit au minimum :

- compiler les variantes nécessaires ;
- exécuter les tests unitaires et Lint configurés ;
- publier les rapports de tests comme artefacts du run lorsqu'ils ne contiennent pas de données sensibles ;
- vérifier que le diff et les fichiers générés ne contiennent pas de secret ni de données personnelles ;
- vérifier les dépendances et licences avec des outils configurés ;
- échouer en cas de contrôle bloquant non réussi.

Les tests de pull request ne doivent pas avoir accès à la clé de signature release. Les protections de branche doivent exiger les contrôles CI pertinents une fois ceux-ci stables.

## 5. Vérifications d'une release

Pour chaque tag de version, suivre `docs/RELEASING.md`. La release doit être bloquée si l'un des contrôles obligatoires échoue :

- tag unique, format correct et commit attendu ;
- correspondance tag/`versionName`, `versionCode` strictement progressif ;
- tests/lint/analyse et audits requis réussis ;
- debug et release construits depuis le même commit tagué ;
- identifiant du paquet et métadonnées de version corrects pour chaque variante ;
- APK release non debuggable et signature vérifiée avec l'identité de publication attendue ;
- debug clairement identifié comme test-only ;
- manifeste fusionné et permissions examinés ;
- dépendances/références/licences à jour dans `THIRD_PARTY.md` ;
- SHA-256 générées pour les artefacts finaux ;
- changelog et notes de release cohérents avec les fonctionnalités vraiment présentes ;
- aucun secret, identifiant, contact réel ou artefact local dans les fichiers publiés.

Un hash SHA-256 seul n'atteste ni la signature ni la provenance. Conserver le commit, le tag, le run CI, le statut des tests et la vérification de signature avec les métadonnées de publication.

## 6. Validation F-Droid distincte

Avant une soumission F-Droid, vérifier la politique d'inclusion actuelle, les licences de l'ensemble des dépendances et la construction depuis les sources dans un environnement propre. Tester la recette F-Droid réelle ; ne pas déduire la compatibilité ou la reproductibilité d'un build GitHub réussi. Voir `docs/RELEASING.md`.

## 7. Critères de sortie

- Aucun bug critique de perte de contacts connu non résolu.
- Les permissions sont minimales et le manifeste est revu.
- Aucune fuite de données personnelles dans les logs ou le réseau non autorisé.
- Les licences/transitives/ressources sont recensées.
- Les tests adaptés aux fonctionnalités livrées sont réussis.
- Les imports/exports et la synchronisation sont validés avant d'activer ces fonctionnalités.
- Le diff est relu, le changelog et la documentation sont à jour.
