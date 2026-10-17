# Notes de recherche technique — 17 octobre 2026

Ce document rassemble les résultats pertinents pour la conception initiale. Les pages officielles et leurs versions en vigueur doivent être relues au moment de l'implémentation, car les API, les versions et les politiques évoluent.

## Conclusions utiles

### Android Contacts Provider

La documentation officielle explique que le Contacts Provider maintient les contacts agrégés à partir de RawContacts provenant de différentes sources et comptes. Il est prévu pour des données de contacts locales et synchronisées et expose un mécanisme SyncAdapter pour intégrer les données d'un service distant. Un type de compte peut avoir plusieurs noms de compte. Cette option convient au besoin d'une app de contacts Android et d'une synchronisation CardDAV qui ne dépend pas de l'installation d'un autre gestionnaire.

Conséquence : modèle source-compte, permissions, RawContacts, agrégation et synchronisation sont des sujets centraux. Ne pas traiter Contact ID comme l'identifiant d'une fiche distante unique et ne pas écrire/supprimer sans vérifier la source.

Source : https://developer.android.com/identity/providers/contacts-provider

### CardDAV et vCard

CardDAV (RFC 6352) est le protocole pour les carnets de contacts basés sur vCard. CalDAV (RFC 4791) s'applique aux calendriers. RFC 6764 documente la découverte des endpoints et les URI `.well-known/carddav`/`.well-known/caldav`. Les identifiants distants et les versions de ressources doivent être suivis afin d'éviter d'écraser une modification concurrente.

Sources :
- https://www.rfc-editor.org/rfc/rfc6352
- https://www.rfc-editor.org/rfc/rfc4791
- https://www.rfc-editor.org/rfc/rfc6764
- https://www.rfc-editor.org/rfc/rfc6350
- https://www.rfc-editor.org/rfc/rfc6578

### Listes et performances Compose

La documentation Compose recommande des listes paresseuses pour les collections longues, car une `Column` qui compose tous les éléments peut provoquer des problèmes de performance. Paging est une option si le besoin le justifie. Les photos doivent aussi être chargées en miniatures bornées et les accès I/O/réseau ne doivent pas bloquer le thread UI.

Source : https://developer.android.com/develop/ui/compose/lists

### Architecture

Android recommande une couche de données clairement définie, des repositories, une couche UI distincte, des flux unidirectionnels et coroutines/flows ; un domaine avec use cases est recommandé surtout quand il simplifie la complexité, pas comme obligation pour toutes les apps.

Source : https://developer.android.com/topic/architecture/recommendations

### Dépendances

La vérification Gradle peut contrôler les sommes de contrôle et signatures des artefacts et détecter un remplacement inattendu. Le bootstrap est sensible : il ne faut pas faire confiance automatiquement à la première résolution si la provenance n'a pas été vérifiée. Les versions et dépendances doivent être verrouillées et examinées.

Source : https://docs.gradle.org/current/userguide/dependency_verification.html

### F-Droid

La politique actuelle demande que les composants distribués soient libres, que les sources soient disponibles et construites, et interdit les SDK propriétaires de publicité/suivi. La reproductibilité et la disponibilité de l'ensemble de la chaîne de build comptent. Une licence GPL seule ne garantit pas l'acceptation.

Sources :
- https://f-droid.org/docs/Inclusion_Policy/
- https://f-droid.org/docs/Reproducible_Builds/
- https://f-droid.org/en/docs/Update_Processing/

## Candidats explorés — aucune dépendance encore choisie

### `dav4jvm`

Le dépôt amont le décrit comme bibliothèque WebDAV/CalDAV/CardDAV pour Java/Kotlin, développée à l'origine pour DAVx⁵ et sous licence MPL-2.0. Il utilise Ktor et documente les opérations asynchrones/suspendues. La réutilisation de la bibliothèque ne signifie pas dépendre de l'application DAVx⁵, mais il reste nécessaire d'examiner la version publiée, la provenance des artefacts, les versions transitives, la politique de redirect/authentification et le build F-Droid.

- Dépôt : https://github.com/bitfireAT/dav4jvm
- Licence annoncée : MPL-2.0
- Dernière publication visible lors de la recherche : version 4.1.0, page de versions consultée le 17 octobre 2026 ; vérifier à nouveau à l'implémentation.

### `ez-vcard`

Le dépôt décrit un parseur/écrivain Java pour vCard 2.1, 3.0 et 4.0, avec prise en charge Android annoncée. La version 0.12.2 a été annoncée le 15 février 2026 ; les métadonnées Maven déclarent la FreeBSD/BSD license. Examiner sa licence exacte, les transitives (dont le parseur) et l'adéquation aux données CardDAV à utiliser avant adoption.

- Dépôt : https://github.com/mangstadt/ez-vcard
- Métadonnées Maven : https://central.sonatype.com/artifact/com.googlecode.ez-vcard/ez-vcard

## Limites de la recherche

Les documents officiels définissent les bases, mais ne décident pas du meilleur stockage pour ce produit. La sélection du Contacts Provider ou d'une base Room privée nécessite un choix produit explicite. La compatibilité concrète avec chaque serveur CardDAV doit être démontrée par des tests d'interopérabilité, pas inférée de la seule conformité théorique au RFC.
