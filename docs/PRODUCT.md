# Spécification fonctionnelle — Contacts

> Ces exigences décrivent ce que le produit doit offrir. Elles ne signifient pas que le code existe. Les décisions d'architecture non approuvées restent indiquées comme telles dans `PROJECT_PROFILE.md` et `ARCHITECTURE.md`.

## 1. Objectif

Créer une alternative Android libre, simple, réactive et personnalisable aux gestionnaires de contacts courants. L'application doit donner un accès rapide à l'information utile, éviter les fonctions décoratives superflues et protéger les données personnelles.

## 2. Principes non négociables

- Android uniquement ; Kotlin et Jetpack Compose par défaut.
- Licence du projet `GPL-3.0-only` ; dépendances et ressources tierces vérifiées.
- Pas de publicité, collecte cachée, analytique intrusive ou transfert de contacts vers un tiers non choisi.
- Accès aux contacts limité à la fonction demandée et expliqué à l'utilisateur.
- Recherche et liste réactives pour une grande base de contacts.
- Toutes les chaînes d'interface localisées à travers les ressources Android, avec gestion du pluriel, formats locaux, RTL et longues traductions.
- Soft Motion est un système visuel de référence ; ne retenir que les fonctionnalités utiles à l'application.

## 3. Portée proposée

### V1 — gestion des contacts et actions système

- Liste des contacts, état vide compréhensible et indicateur d'erreur si la source de données n'est pas disponible.
- Recherche rapide. La proposition initiale est de chercher dans les noms, numéros et adresses e-mail, sans exiger de correspondance exacte ; confirmer les champs et les règles de normalisation pendant la conception.
- Fiche contact détaillée avec les champs que la source de données sait représenter : nom, plusieurs numéros, plusieurs adresses e-mail, adresses postales, organisation/poste, notes et photo lorsque disponibles.
- Création, modification et suppression d'un contact. Une suppression doit préciser le contact/la source affectée et ne jamais déclencher une suppression en cascade cachée.
- Actions d'appel, de SMS et d'e-mail ouvrant l'application système correspondante (ex. `ACTION_DIAL`, `ACTION_SENDTO` avec schéma approprié). Aucun appel ou message ne doit être lancé/envoyé sans intervention explicite dans l'application cible.
- Affichage de la source ou du compte associé à un contact lorsque cette information existe. Les limites d'édition doivent être visibles.
- Thème, typographie, navigation et animations pertinents de Soft Motion, sans obligation d'offrir trois thèmes complets, un sélecteur de couleur, les fonds photo/dots ou des célébrations.

### Fonctions prévues, à prioriser après validation du modèle de données

- Import/export de plusieurs contacts au format `.vcf` avec le Storage Access Framework ; choix explicite des contacts à exporter lorsque c'est pertinent, traitement des erreurs et prévention des doublons sans fusion silencieuse.
- Photo de contact : sélection de l'image et respect des droits d'accès URI et des limites de mémoire. Éviter la lecture globale des médias si un sélecteur système suffit.
- Groupes ou étiquettes, avec règles de propriété et comportement pour les contacts issus de comptes différents.
- Synchronisation CardDAV directe avec plusieurs comptes personnels, hors dépendance obligatoire à DAVx⁵ ; statut de synchronisation par compte, action de synchronisation manuelle, gestion des erreurs et mode hors ligne.
- Synchronisation bidirectionnelle sûre après implémentation des mécanismes de versionnement, conflits et suppressions distantes.

## 4. Modèle de stockage : Contacts Provider Android retenu

L'architecture retient le **Contacts Provider Android** (`ContactsContract` et `ContentResolver`) comme source principale de données :

1. **Contacts Provider Android** (validé) : les contacts sont ceux du système, partagés avec les autres applications et organisés par comptes (`RawContacts` agrégés en `Contacts`). L'application utilise les permissions appropriées (`READ_CONTACTS`, `WRITE_CONTACTS`) et gère rigoureusement la propriété des fiches.
2. **Base privée Room** (écartée pour les contacts) : non introduite comme seconde base primaire de contacts afin d'éviter les doubles écritures et les doublons. Room pourra être utilisé ultérieurement uniquement pour un besoin documenté et justifié de métadonnées spécifiques à l'application.

Le périmètre initial couvre la gestion locale (consultation, recherche, affichage, création, édition, suppression, actions système). La synchronisation directe CardDAV multi-comptes constitue une phase ultérieure qui exigera une étude préalable d'`AccountManager`, `SyncAdapter`, de la propriété des fiches, des identifiants distants, des ETag, des conflits et de la gestion des suppressions.

## 5. Confidentialité et flux de données attendus

- Aucun serveur n'est configuré par défaut.
- Les contacts restent locaux jusqu'à une action explicite d'export ou la configuration d'un compte CardDAV.
- Avant d'ajouter un compte, présenter le serveur et les données de contact qu'il pourra recevoir.
- Une synchronisation ne doit cibler que les comptes explicitement configurés. Aucun analytics, tracker, publicitaire ou endpoint tiers caché.
- Les erreurs peuvent inclure le type de problème, jamais des données de contact ou des identifiants.
- Voir `PRIVACY.md` et `SECURITY.md`.

## 6. Exigences UX

- Recherche atteignable rapidement depuis la liste.
- États distincts : chargement, aucun contact, aucun résultat, permission refusée, source indisponible, erreur de lecture/écriture et erreur de synchronisation.
- Préserver les modifications à travers rotation, recréation d'activité et reprise de l'app.
- Dialogues de confirmation pour les actions irréversibles ; annulation claire lorsqu'elle est possible.
- Respecter l'agrandissement du texte, TalkBack, focus clavier/D-pad, RTL et contrastes.
- Ne pas utiliser la couleur seule pour différencier compte, état ou type de champ.

## 7. Non-objectifs initiaux

- Pas de remplacement du composeur téléphonique, de l'application SMS ou du client e-mail.
- Pas d'envoi automatique d'appel, SMS ou e-mail.
- Pas de calendrier : CalDAV est un protocole de calendriers, alors que CardDAV est celui des contacts. CalDAV reste hors périmètre sauf demande distincte.
- Pas de service cloud géré par le projet, compte propriétaire central, publicité, profilage ou télémétrie.
- Pas de gestion des appels, de journal d'appels, d'enregistrement audio ni permission associée.
- Pas de support de protocoles de synchronisation supplémentaires tant que leur besoin n'est pas explicité.

## 8. Critères d'acceptation — base initiale

| ID | Exigence testable | Priorité | Statut |
|---|---|---|---|
| C-001 | Afficher les contacts accessibles depuis la source retenue et distinguer clairement les états vide/erreur | Must | À implémenter |
| C-002 | La recherche filtre sans bloquer le thread UI et retourne les résultats attendus dans un corpus de test connu | Must | À implémenter |
| C-003 | Afficher les numéros et e-mails disponibles et ouvrir l'action système choisie après sélection utilisateur | Must | À implémenter |
| C-004 | Créer, modifier et supprimer un contact sans modifier involontairement d'autres contacts/comptes | Must | À implémenter |
| C-005 | Les permissions refusées/révoquées n'entraînent ni plantage ni message trompeur | Must | À implémenter |
| C-006 | Aucune donnée de contact/identifiant n'apparaît dans les journaux en production | Must | À vérifier |
| C-007 | Chaque chaîne visible passe par les ressources traduisibles ; le français et l'anglais sont vérifiés avant une version publique | Must | À implémenter |
| C-008 | Les listes et images restent raisonnables en mémoire avec un corpus de grande taille défini dans TESTING.md | Must | À mesurer |
| C-009 | L'import/export vCard conserve les champs pris en charge et signale les champs non pris en charge | Should | Phase à décider |
| C-010 | La synchronisation distante respecte les comptes séparés et ne traite jamais une erreur réseau comme une suppression générale | Must avant sync publique | À concevoir |
| C-011 | Plusieurs comptes CardDAV peuvent être ajoutés sans mélanger leurs identifiants ni leur état de synchronisation | Must avant sync publique | À concevoir |

## 9. Décisions ouvertes

Les éléments suivants doivent être discutés et approuvés avant d'être transformés en contraintes d'implémentation :

- Quels contacts doivent être visibles : tous les comptes système, contacts locaux uniquement, ou filtres par source ?
- Comportement des contacts Google/autres comptes non modifiables par l'app.
- Nom définitif, identifiant de package, `minSdk`, et langues pour la première version.
- Priorité entre import/export vCard, groupes et synchronisation directe CardDAV.
- Serveurs CardDAV et méthodes d'authentification à supporter d'abord.
- Stratégie de résolution des conflits, suppression distante et restauration après une erreur.
- Mode de signature et avertissement pour les installations provenant de GitHub vs F-Droid.
