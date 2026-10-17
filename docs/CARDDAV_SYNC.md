# Synchronisation directe CardDAV — exigences et conception

## 1. Terminologie

Pour les contacts, le protocole à prendre en charge est **CardDAV** (RFC 6352), qui étend WebDAV pour accéder à des carnets de contacts basés sur vCard. **CalDAV** (RFC 4791) est le protocole de calendriers et n'est pas requis pour ce projet. La découverte des services CardDAV/CalDAV et les URI bien connues sont définies dans RFC 6764.

Références normatives :

- CardDAV : https://www.rfc-editor.org/rfc/rfc6352
- Découverte CardDAV/CalDAV : https://www.rfc-editor.org/rfc/rfc6764
- vCard 4.0 : https://www.rfc-editor.org/rfc/rfc6350
- WebDAV : https://www.rfc-editor.org/rfc/rfc4918
- WebDAV Sync Collection (si supportée par le serveur) : https://www.rfc-editor.org/rfc/rfc6578

## 2. Objectif produit

L'application doit pouvoir se connecter directement à plusieurs comptes CardDAV configurés par l'utilisateur. L'installation et l'exécution de DAVx⁵ ne doivent pas être une dépendance obligatoire. Une bibliothèque libre peut être envisagée ; si elle est retenue, sa licence, ses dépendances, sa disponibilité source et sa compatibilité avec F-Droid doivent être vérifiées.

## 3. Découverte et connexion

La conception doit couvrir, sans présumer que tous les serveurs sont identiques :

- serveur/URL fourni par l'utilisateur et support éventuel d'une découverte `.well-known/carddav` conforme à RFC 6764 ;
- gestion des redirections et des chemins finaux, sans envoyer des identifiants à un hôte non approuvé au cours d'une redirection ;
- découverte du principal courant, du carnet d'adresses de base et des collections accessibles par l'utilisateur ;
- authentification échouée, mauvais chemin, certificat invalide, serveur non compatible, compte verrouillé et perte réseau ;
- connexion TLS avec validation de certificat obligatoire en production ; pas d'option dangereuse « accepter tous les certificats » ;
- pas de transmission du mot de passe avant que la destination et le canal soient validés.

Le premier mode d'authentification doit être décidé après sélection des serveurs cibles. Ne pas promettre OAuth, Basic, Digest ou autre avant d'avoir vérifié les exigences réelles des serveurs. Le support d'une authentification doit être documenté par compte.

## 4. Plusieurs comptes et séparation

- Chaque compte conserve sa propre URL/serveur, identité d'authentification, principal, collection, identifiants distants, ETag/tokens et état de synchronisation.
- Les contacts de deux comptes ne doivent pas être envoyés au mauvais serveur.
- L'UI affiche le nom/serveur du compte et un état compréhensible : prêt, synchronisation, réussi, hors ligne, authentification nécessaire, conflit ou erreur.
- Permettre l'ajout, la validation, la synchronisation manuelle, la pause ou désactivation et la suppression d'un compte.
- Avant de supprimer un compte, expliquer si les contacts locaux du compte seront supprimés du fournisseur Android, conservés ou marqués pour suppression ; ne jamais déduire une politique silencieusement.
- Aucune synchronisation ne démarre pour un serveur simplement découvert ou suggéré avant la confirmation de l'utilisateur.

## 5. Synchronisation sûre

Avant de permettre l'édition bidirectionnelle, la conception doit expliquer et tester :

- comment sont identifiés les contacts distants (URL de ressource, UID vCard et identifiant stable) ;
- la conservation et l'utilisation des ETag et des éventuels `sync-token` ;
- la récupération initiale et incrémentale, les pages/réponses partielles, les réponses multi-status WebDAV, les redirections et les erreurs ;
- les opérations conditionnelles de mise à jour/suppression lorsque le serveur fournit une version (par exemple `If-Match`) ;
- la distinction entre absence confirmée d'un contact et échec de récupération du carnet ;
- la manière de traiter une même fiche modifiée à la fois localement et à distance ;
- l'encodage, les champs inconnus, les lignes repliées et les versions vCard prises en charge ;
- l'impact des modifications locales au Contacts Provider sur les indicateurs de synchronisation Android.

Aucun échec réseau, 401/403, réponse incomplète, parsing invalide ou erreur temporaire ne peut déclencher une suppression massive locale. Les suppressions distantes doivent être reflétées seulement à partir d'une preuve de suppression confirmée, dans le compte concerné.

## 6. Politique de conflit

Cette politique est **à définir avant toute synchronisation bidirectionnelle**. Au minimum, proposer un comportement sûr : détecter le conflit, ne pas écraser silencieusement, préserver la dernière copie fiable et fournir une résolution compréhensible. Si une fusion automatique est proposée, documenter les champs fusionnables et ceux qui nécessitent un choix humain. Tester les modifications concurrentes, l'absence d'ETag, la suppression d'un côté et la modification de l'autre.

## 7. Travail en arrière-plan

Si le Contacts Provider est retenu, évaluer le framework Android SyncAdapter/AccountManager, qui a été conçu pour synchroniser les RawContacts d'un compte et prendre en charge plusieurs noms de compte pour un même type. Sinon, documenter pourquoi une orchestration interne est préférable. Le travail doit respecter la connectivité, les restrictions batterie, la pause/annulation, les reprises et l'idempotence.

Le bouton « Synchroniser maintenant » doit exposer le résultat réel. Aucun indicateur de réussite ne doit être affiché avant la confirmation du serveur et l'application des changements.

## 8. Sécurité et confidentialité

- Le serveur distant configuré est le seul destinataire autorisé des contacts de son compte.
- Les identifiants sont stockés dans un mécanisme protégé adapté aux API Android actuelles, pas en clair.
- Ne jamais écrire credentials, contenu vCard, noms, e-mails, numéros ou notes dans les journaux.
- Pas de télémétrie réseau. Les diagnostics doivent être locaux et expurgés.
- Limiter les tailles de réponses, le nombre de redirections, le temps d'attente, la profondeur de parsing et les ressources consommées par les réponses malformées.
- Vérifier les URI et changements d'hôte à chaque redirection.
- Ne pas adopter de certificate pinning par défaut sans modèle de menace précis et stratégie de rotation ; ne jamais désactiver la validation standard TLS.

## 9. Compatibilité et test

Établir une matrice de serveurs après décision (par exemple les serveurs personnels retenus par le propriétaire et au moins un serveur de test libre). Pour chaque serveur, tester découverte, première sync, ajout/modification/suppression des deux côtés, ETag, conflits, serveur hors ligne, mauvaises informations d'identification, certificats et fichiers vCard inhabituels. Utiliser des carnets de test jetables, jamais le carnet de production pendant les tests automatisés.

## 10. Pistes techniques à réévaluer

- `dav4jvm` : https://github.com/bitfireAT/dav4jvm — bibliothèque WebDAV/CardDAV JVM/Kotlin, licence MPL-2.0 annoncée, utilise Ktor. La dépendance directe à DAVx⁵ n'est pas requise par la bibliothèque ; vérifier la version, la publication, les transitives et l'intégration SyncAdapter avant décision.
- `ez-vcard` : https://github.com/mangstadt/ez-vcard — analyse et écrit vCard 2.1, 3.0, 4.0 ; version 0.12.2 publiée en février 2026 selon le dépôt consulté. Vérifier la licence exacte et les dépendances transitives de l'artefact choisi.

Ces références sont des candidates, pas des décisions d'intégration. Il faut privilégier un minimum de bibliothèques, mais ne pas réimplémenter à la main la gestion complexe de vCard/WebDAV si une bibliothèque testée et compatible est disponible.
