# Architecture — proposition à approuver

> Le projet n'est pas encore implémenté. Ce document décrit une proposition de départ et des options, pas une architecture existante.

## 1. Recommandation en résumé

Pour une application qui doit jouer le rôle de gestionnaire de contacts Android, afficher les comptes existants et fournir à terme une synchronisation CardDAV native multi-comptes, privilégier **Android Contacts Provider** comme source des contacts, via `ContentResolver` et `ContactsContract`. Si cette approche est approuvée, envisager le mécanisme Android `AccountManager` + `AbstractThreadedSyncAdapter` pour enregistrer plusieurs comptes CardDAV et synchroniser les contacts vers le fournisseur système.

C'est une architecture plus complexe qu'une base privée, mais elle rend les contacts disponibles aux autres applications et utilise le modèle de comptes/synchronisation prévu par Android. La documentation Android indique que Contacts Provider réunit des contacts bruts provenant de comptes distincts, les agrège et prévoit des sync adapters pour échanger avec un service distant.

**Ne pas implémenter l'approche avant que le responsable du projet l'ait approuvée.** Le premier travail de conception doit comparer les options et rendre visibles leurs conséquences.

## 2. Options de stockage

### Option A — Contacts Provider (recommandée, à valider)

- Lecture/écriture par `ContactsContract` et `ContentResolver`.
- Demander `READ_CONTACTS`/`WRITE_CONTACTS` au moment nécessaire, expliquer le bénéfice et gérer le refus/la révocation.
- Respecter l'identité de chaque RawContact, de chaque compte et les règles d'agrégation de contacts d'Android.
- Éviter de présenter une entrée agrégée comme une donnée qui peut être modifiée/supprimée intégralement sans vérifier ses RawContacts et sources.
- Le CardDAV peut être représenté comme un type de compte Android avec son propre sync adapter ; chaque compte a sa propre identité, collection distante, état et credentials.
- Ne pas supposer que toutes les sources de contacts sont éditables ou synchronisables par l'app.

### Option B — base privée Room

- Stocker les fiches dans Room et exposer les flux par des repositories.
- La synchronisation serait contrôlée par l'application, probablement avec WorkManager ; gérer son propre modèle de conflits, UID distants, ETag, suppressions, identifiants et migrations.
- Les contacts resteraient dans l'app et ne seraient pas automatiquement disponibles dans les autres applications ou le composeur système.
- Éviter d'ajouter un second store en plus du Contacts Provider sans exigence métier nette ; la double écriture crée des problèmes de conflits, doublons et perte de données.

## 3. Structure logique proposée

À adapter selon l'option retenue et la taille réelle du code. Pas de multi-module ou couche de domaine obligatoire sans bénéfice démontré.

- `ui/contacts` : liste, recherche, filtres et états d'écran.
- `ui/contactdetail` : fiche, champ, photo et actions.
- `ui/contactedit` : ajout/modification et validation.
- `ui/settings` : préférences, comptes et options activées.
- `data/contacts` : adaptateur du Contacts Provider ou accès Room, selon la décision.
- `data/vcard` : import/export et mapping vCard lorsqu'implémenté.
- `data/carddav` : accès protocole, découverte, authentification, opérations distantes et modèles de réponse.
- `sync` : orchestration, état de synchronisation, comparaison locale/distante, conflits et planification si la fonctionnalité est activée.
- `core/designsystem` : uniquement les tokens et composants réutilisables effectivement nécessaires.

Flux recommandé : UI Compose → ViewModel/UI state → repository → source de données approuvée. Les composables ne doivent pas appeler directement `ContentResolver`, la base, le réseau ou manipuler les credentials.

## 4. Interface et état

- Kotlin + Jetpack Compose pour un nouveau projet.
- Architecture en couches raisonnable, flux unidirectionnel, état UI observable (`StateFlow` si approprié) et coroutines pour les opérations asynchrones.
- Ne pas bloquer le thread principal avec des requêtes de contacts, de fichiers, des images ou le réseau.
- Employer `LazyColumn`/chargement incrémental lorsque nécessaire ; mettre des clés stables et réduire les conversions/lectures répétées.
- Garder les composants adaptatifs et les chaînes localisées. Un module de design séparé ne doit être ajouté que s'il réduit réellement la complexité.

## 5. Synchronisation CardDAV

La synchronisation est une fonctionnalité distincte, sensible et potentiellement destructive ; ne pas la développer à partir d'un simple test HTTP. Lire `CARDDAV_SYNC.md` et traiter ses exigences comme des critères de conception.

Si le Contacts Provider est retenu, examiner un compte Android + SyncAdapter afin de synchroniser les RawContacts du compte ciblé, préserver `SOURCE_ID`/identifiants distants et respecter les indicateurs de modification/synchronisation du fournisseur. Documenter le cycle de vie, l'authentification, le travail en arrière-plan et la gestion des erreurs. Ne pas faire fonctionner deux planificateurs concurrents pour une seule synchronisation sans raison documentée.

Si la base Room est retenue, employer des transactions, des migrations versionnées et une stratégie de synchronisation idempotente distincte. Ne jamais utiliser une liste de contacts vide obtenue après erreur réseau comme signal de suppression.

## 6. Persistance et préférences

- Ne créer une base Room que si les données sont détenues par l'application (option B ou métadonnées spécifiques réellement nécessaires).
- Préférences non sensibles : DataStore si utile.
- Secrets de comptes : choisir un mécanisme de stockage fondé sur les API Android actuelles et documenter sa revue de sécurité avant implémentation ; jamais en clair.
- Les photos ne doivent pas être chargées en pleine résolution pour chaque ligne. Charger des miniatures dimensionnées et conserver un cache borné.
- Les migrations ou modifications de schéma doivent être testées, jamais résolues en effaçant automatiquement les données existantes.

## 7. Actions système

- Appel : ouvrir le composeur avec le numéro sélectionné.
- SMS : ouvrir une application compatible avec une URI `smsto:`.
- E-mail : ouvrir une application compatible avec une URI `mailto:`.
- Vérifier qu'un gestionnaire compatible existe et offrir un message compréhensible si aucun n'est installé.
- Ne pas demander de permission directe de téléphonie si l'intent satisfait l'usage.

## 8. Dépendances et build

- Utiliser le Gradle Wrapper et un catalogue de versions si le projet s'y prête.
- Vérifier les licences et dépendances transitives avant tout choix de library CardDAV, vCard, UI ou d'injection.
- Candidate DAV à évaluer : `dav4jvm` (MPL-2.0 annoncé, WebDAV/CardDAV pour JVM/Kotlin, utilisant Ktor) ; ne pas ajouter sans vérifier les versions, publications, dépendances et compatibilité avec la construction F-Droid.
- Candidate vCard à évaluer : `ez-vcard` (BSD/FreeBSD déclaré dans les métadonnées, prend en charge vCard 2.1/3.0/4.0 ; vérifier la licence et les transitives de la version exacte).
- Aucune bibliothèque n'est approuvée uniquement par son nom ou par les références ci-dessus. Voir `THIRD_PARTY.md`.

## 9. Questions à trancher

1. Valider Contacts Provider ou Room privé.
2. Définir si l'application doit gérer les contacts de tous les comptes ou permettre un filtre par source.
3. Valider l'intégration de CardDAV comme compte système avec SyncAdapter.
4. Définir le socle Android minSdk sans exclure inutilement d'appareils ; choisir les SDK actuels par consultation des sources officielles.
5. Valider modèle de groupe/étiquette et son échange avec vCard/serveurs.
6. Décider des modes d'authentification et des serveurs de test initiaux.

## Références

- Contacts Provider : https://developer.android.com/identity/providers/contacts-provider
- Architecture Android : https://developer.android.com/topic/architecture/recommendations
- Compose lists : https://developer.android.com/develop/ui/compose/lists
- Room : https://developer.android.com/training/data-storage/room
- Gradle dependency verification : https://docs.gradle.org/current/userguide/dependency_verification.html
