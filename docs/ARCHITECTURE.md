# Architecture — proposition à approuver

> Le projet n'est pas encore implémenté. Ce document décrit des options, pas une architecture déjà présente dans le code.

## 1. Recommandation provisoire

Pour une application destinée à devenir un véritable gestionnaire de contacts Android et à prendre en charge à terme plusieurs comptes CardDAV, privilégier l'étude du **Contacts Provider Android** (`ContentResolver` et `ContactsContract`). Si cette approche est approuvée, évaluer `AccountManager` et l'architecture de synchronisation Android pour représenter chaque compte CardDAV.

Cette option expose les contacts aux autres applications et profite du modèle système des comptes, mais complexifie la propriété des fiches, l'agrégation, les permissions et la synchronisation. Le propriétaire doit approuver l'approche après la comparaison avec une base privée Room. Ne pas implémenter de synchronisation complète avant cette décision.

## 2. Options de stockage

### Option A — Contacts Provider (à approuver)

- Lecture/écriture via `ContactsContract` et `ContentResolver`.
- Demander `READ_CONTACTS` et `WRITE_CONTACTS` au moment pertinent, expliquer leur utilité et traiter le refus/la révocation.
- Distinguer les lignes agrégées (`Contacts`) des contacts bruts (`RawContacts`) et des données (`Data`).
- Toute édition/suppression doit porter sur le `RawContact` ciblé et vérifier la propriété, les permissions et les capacités effectives de la source. `IS_READ_ONLY` constitue une information utile mais ne garantit pas à lui seul qu'une écriture réussira.
- Ne pas supposer que tous les comptes externes sont modifiables ou synchronisables par l'application.
- Le compte CardDAV pourra être exposé au système si la conception et les tests confirment le choix d'un adaptateur de synchronisation natif.

### Option B — base privée Room

- Stocker les fiches dans Room et exposer les flux via des repositories.
- Contrôler la synchronisation au sein de l'application et concevoir son propre modèle d'identifiants distants, ETag, conflits, suppressions et migrations.
- Les contacts ne seront pas automatiquement visibles dans les autres applications Android.
- Ne pas ajouter un deuxième stockage en parallèle du Contacts Provider sans exigence produit démontrée : les doubles écritures créent un risque de conflit et de doublons.

## 3. Structure logique proposée

À adapter à la taille réelle du projet ; aucun multi-module ou use-case séparé n'est obligatoire sans bénéfice concret.

- `ui/contacts` : liste, recherche et états d'écran.
- `ui/contactdetail` : fiche, champs, photo et actions.
- `ui/contactedit` : création/modification et validation.
- `ui/settings` : préférences et comptes si activés.
- `data/contacts` : adaptateur Contacts Provider ou Room selon la décision.
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

## 5. Synchronisation CardDAV

CardDAV est une fonctionnalité distincte, sensible et potentiellement destructive. Lire `CARDDAV_SYNC.md`. Une simple requête HTTP réussie ne prouve pas qu'un cycle de synchronisation est sûr.

Si Contacts Provider est retenu, étudier le cycle de vie de chaque compte, les identifiants distants, les marqueurs de synchronisation, les ETag, la suppression et les conflits. Ne pas faire coexister deux planificateurs pour une même fonction sans raison démontrée.

Si Room est retenu, employer des transactions, migrations versionnées et un protocole idempotent. Une collection vide obtenue après erreur réseau ne doit jamais déclencher une suppression.

## 6. Persistance, secrets et données

- Créer une base Room uniquement si les données sont détenues par l'application ou si des métadonnées spécifiques la justifient.
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

## 9. Questions à trancher

1. Contacts Provider ou Room privé.
2. Filtres et sources de contacts visibles dans l'application.
3. Modèle d'édition des contacts agrégés ou issus de comptes externes.
4. `applicationId`, nom public et socle `minSdk`/SDK vérifiés.
5. Import/export vCard, groupes/étiquettes et ordre des phases.
6. Serveurs CardDAV et méthodes d'authentification à tester.
7. Politique de conflits et de suppression avant synchronisation bidirectionnelle.
8. Signature, stockage de la clé et préparation des canaux GitHub/F-Droid avant la première distribution.

## Références

- Contacts Provider : https://developer.android.com/identity/providers/contacts-provider
- Contrat `RawContacts` : https://developer.android.com/reference/android/provider/ContactsContract.RawContacts
- Architecture Android : https://developer.android.com/topic/architecture/recommendations
- Compose lists : https://developer.android.com/develop/ui/compose/lists
- Room : https://developer.android.com/training/data-storage/room
- Gradle dependency verification : https://docs.gradle.org/current/userguide/dependency_verification.html
