# Architecture

> Le projet n'est pas encore implémenté. Ce document décrit l'architecture retenue, pas une implémentation déjà présente dans le code.

## 1. Décision d'architecture validée

Le **Contacts Provider Android** (`ContentResolver` et `ContactsContract`) est officiellement retenu comme la source principale de données de contacts pour l'application.

Cette décision permet à l'application de s'intégrer comme gestionnaire de contacts natif du système, de rendre les contacts accessibles aux autres applications et de s'appuyer sur le modèle de comptes Android. Elle impose en contrepartie d'utiliser les permissions appropriées (`READ_CONTACTS`, `WRITE_CONTACTS`) et de préserver rigoureusement la distinction entre contacts agrégés (`Contacts`) et contacts bruts rattachés à un compte (`RawContacts`), ainsi que les lignes de données (`Data`).

Room n'est pas introduit comme seconde base primaire de contacts : maintenir un double stockage en parallèle créerait des risques majeurs de conflits, de désynchronisation et de doublons. Une base Room pourra être introduite ultérieurement uniquement pour un besoin documenté et justifié de métadonnées propres à l'application non prises en charge par le Contacts Provider.

L'implémentation est organisée en phases distinctes :
1. **Périmètre initial (bootstrap)** : consultation locale, recherche, affichage détaillé, création, édition, suppression de contacts et déclenchement d'actions système (appels, SMS, e-mails), sous réserve des permissions requises et des tests associés.
2. **Phase ultérieure** : synchronisation directe multi-comptes CardDAV, hors du périmètre du bootstrap initial. Avant d'implémenter CardDAV, une étude approfondie doit analyser `AccountManager`, `SyncAdapter`, la propriété des fiches, les identifiants distants, les ETag, les conflits, la gestion des suppressions, les reprises et la prévention des pertes de données.

## 2. Modèle de stockage et statut de Room

### Contacts Provider Android (source principale retenue)

- Lecture/écriture via `ContactsContract` et `ContentResolver`.
- Demander `READ_CONTACTS` et `WRITE_CONTACTS` au moment pertinent, expliquer leur utilité et traiter le refus/la révocation.
- Distinguer les lignes agrégées (`Contacts`) des contacts bruts (`RawContacts`) et des données (`Data`).
- Toute édition/suppression doit porter sur le `RawContact` ciblé et vérifier la propriété, les permissions et les capacités effectives de la source. `IS_READ_ONLY` constitue une information utile mais ne garantit pas à lui seul qu'une écriture réussira.
- Ne pas supposer que tous les comptes externes sont modifiables ou synchronisables par l'application.
- Le compte CardDAV pourra être exposé au système si la conception et les tests confirment le choix d'un adaptateur de synchronisation natif (`AccountManager` / `SyncAdapter`).

### Statut de Room (écarté pour les contacts, réservé aux métadonnées éventuelles)

- Room n'est pas utilisé comme stockage de contacts : ne pas ajouter un deuxième stockage en parallèle du Contacts Provider, car les doubles écritures créent un risque de conflit et de doublons.
- Les contacts restent dans l'annuaire Android afin d'être visibles et utilisables par les autres applications.
- Une base Room pourra être introduite ultérieurement uniquement pour un besoin documenté et justifié, par exemple pour des métadonnées spécifiques à l'application.

## 3. Structure logique proposée

À adapter à la taille réelle du projet ; aucun multi-module ou use-case séparé n'est obligatoire sans bénéfice concret.

- `ui/contacts` : liste, recherche et états d'écran.
- `ui/contactdetail` : fiche, champs, photo et actions.
- `ui/contactedit` : création/modification et validation.
- `ui/settings` : préférences et comptes si activés.
- `data/contacts` : adaptateur Contacts Provider (`ContentResolver` et `ContactsContract`).
- `data/vcard` : import/export et mapping vCard.
- `data/carddav` : découverte, authentification, requêtes, modèles/réponses.
- `sync` : orchestration, état, suivi des versions, conflits et planification lorsque cette fonction est activée.
- `core/designsystem` : tokens/composants effectivement réutilisables, sans module séparé forcé.

Flux recommandé : Compose UI → état UI/ViewModel → repository → source de données approuvée. Les composables ne doivent pas appeler directement le réseau, la base, `ContentResolver` ou manipuler des secrets.

## 4. Interface, état et performances

- Kotlin + Jetpack Compose pour ce nouveau projet.
- Séparer l'UI de la couche de données ; utiliser `StateFlow`/coroutines lorsque cela simplifie l'observation et les opérations asynchrones.
- Ne jamais bloquer le thread principal avec les contacts, les fichiers, les images ou le réseau.
- Utiliser `LazyColumn`/pagination seulement quand le besoin le justifie, avec des clés stables et un cache de miniatures borné.
- Localiser les chaînes et prendre en charge RTL, grandes polices, accessibilité et fenêtres adaptatives selon le périmètre convenu.

## 5. Synchronisation CardDAV (phase ultérieure)

CardDAV est une fonctionnalité distincte, sensible et potentiellement destructive, planifiée dans une phase ultérieure et non dans le bootstrap initial. Lire `CARDDAV_SYNC.md`. Une simple requête HTTP réussie ne prouve pas qu'un cycle de synchronisation est sûr.

Avec le choix du Contacts Provider comme source principale, la phase CardDAV nécessite d'étudier préalablement `AccountManager`, `SyncAdapter`, le cycle de vie de chaque compte, la propriété des `RawContacts`, les identifiants distants, les ETag, les marqueurs de synchronisation, la résolution des conflits, la gestion des suppressions, l'idempotence, les reprises et la prévention des pertes de données. Ne pas faire coexister deux planificateurs pour une même fonction sans raison démontrée.

Une erreur réseau, une réponse partielle ou une collection vide obtenue après incident ne doit jamais déclencher une suppression de contacts, qu'elle soit locale ou distante.

## 6. Persistance, secrets et données

- Ne pas créer de base Room pour les contacts primaires ; une base Room ne pourra être introduite ultérieurement que si un besoin documenté de métadonnées spécifiques à l'application le justifie.
- Préférences non sensibles : DataStore si utile.
- Secrets de compte : choisir une stratégie fondée sur des API Android actuelles, la documenter et la vérifier avant implémentation ; jamais en clair.
- Les photos de liste doivent être dimensionnées ; borner cache et mémoire.
- Toute évolution de schéma doit être versionnée et testée. Ne jamais résoudre automatiquement un problème de migration en supprimant les données utilisateur.

## 7. Actions système

- Appel : ouvrir le composeur avec le numéro choisi.
- SMS : ouvrir une application compatible avec une URI `smsto:`.
- E-mail : ouvrir une application compatible avec une URI `mailto:`.
- Vérifier qu'un gestionnaire existe et afficher une erreur compréhensible sinon.
- Ne pas demander de permission directe de téléphonie si les intents satisfont le besoin.

## 8. Build, dépendances et livraison

- Utiliser le Gradle Wrapper, un catalogue de versions si utile et des versions mutuellement compatibles vérifiées à la date du bootstrap.
- Vérifier les licences, transitives et origines de toutes les dépendances selon `THIRD_PARTY.md`.
- Les candidates `dav4jvm` et `ez-vcard` ne sont pas approuvées par leur seule présence dans la documentation.
- Les branches, revues et CI sont décrites dans `DEVELOPMENT_WORKFLOW.md` ; la version, le tag, les APK et GitHub/F-Droid sont décrits dans `RELEASING.md`.
- Ne pas créer de workflow présenté comme fonctionnel tant que les tâches Gradle réelles ne sont pas créées et testées.

## 9. Questions restantes à trancher

1. Filtres et sources de contacts visibles dans l'application (tous comptes système, contacts locaux uniquement, filtres par source).
2. Modèle d'édition des contacts agrégés ou issus de comptes externes non modifiables.
3. `applicationId`, nom public et socle `minSdk`/SDK vérifiés.
4. Import/export vCard, groupes/étiquettes et ordonnancement fin des phases suivantes.
5. Serveurs CardDAV et méthodes d'authentification à tester lors de la phase CardDAV.
6. Politique de conflits et de suppression avant synchronisation bidirectionnelle CardDAV.
7. Signature, stockage de la clé et préparation des canaux GitHub/F-Droid avant la première distribution.

## Références

- Contacts Provider : https://developer.android.com/identity/providers/contacts-provider
- Contrat `RawContacts` : https://developer.android.com/reference/android/provider/ContactsContract.RawContacts
- Architecture Android : https://developer.android.com/topic/architecture/recommendations
- Compose lists : https://developer.android.com/develop/ui/compose/lists
- Room : https://developer.android.com/training/data-storage/room
- Gradle dependency verification : https://docs.gradle.org/current/userguide/dependency_verification.html
