# Notes de recherche technique

> Document de synthèse évolutif. Les versions de SDK, outils, bibliothèques et politiques citées par date ne sont pas des vérités permanentes. Les versions stables et la documentation officielle doivent être revérifiées au moment du bootstrap et avant toute mise à jour majeure.

## 1. Contacts Provider Android

La documentation officielle décrit les contacts agrégés construits à partir de RawContacts provenant de différentes sources et comptes, ainsi que l'intégration via les mécanismes de synchronisation Android. Cette approche peut convenir à une application qui veut être un gestionnaire de contacts du système, mais introduit des responsabilités de propriété, d'agrégation, de permissions et de synchronisation.

Ne pas utiliser le seul identifiant du contact agrégé pour déterminer la ressource distante unique. Ne pas supposer qu'un indicateur de lecture seule égal à zéro garantit que toute modification réussira : permissions, propriétaire, capacités de source et résultat réel de l'écriture restent à contrôler.

Source : https://developer.android.com/identity/providers/contacts-provider

## 2. CardDAV et vCard

CardDAV (RFC 6352) sert aux carnets de contacts basés sur vCard. CalDAV (RFC 4791) concerne les calendriers. RFC 6764 définit des règles de découverte. Les identifiants distants, ETag et jetons de synchronisation éventuels doivent être étudiés afin d'éviter les écrasements et suppressions injustifiés.

- CardDAV : https://www.rfc-editor.org/rfc/rfc6352
- CalDAV : https://www.rfc-editor.org/rfc/rfc4791
- Découverte : https://www.rfc-editor.org/rfc/rfc6764
- vCard : https://www.rfc-editor.org/rfc/rfc6350
- WebDAV Sync Collection : https://www.rfc-editor.org/rfc/rfc6578

## 3. Architecture, Compose et performance

La documentation Android recommande des couches de données et UI séparées, des repositories et des flux unidirectionnels. Un domaine/use-case séparé se justifie quand il simplifie la complexité, mais n'est pas obligatoire pour toutes les applications. Les listes longues doivent être conçues avec des listes paresseuses, des identifiants stables et des chargements d'images maîtrisés.

- Architecture : https://developer.android.com/topic/architecture/recommendations
- Listes Compose : https://developer.android.com/develop/ui/compose/lists
- Room : https://developer.android.com/training/data-storage/room

## 4. Versions Android et toolchain

Ne pas fixer les SDK Android ou versions Kotlin/AGP/Gradle/JDK d'après un ancien rapport ou la date de ce document. Au moment du bootstrap :

1. consulter les pages officielles des versions Android et les notes de version de l'Android Gradle Plugin, Gradle, Kotlin et JDK ;
2. vérifier une matrice de versions mutuellement compatibles ;
3. choisir le `minSdk` en fonction de la compatibilité utilisateur et des API réellement nécessaires ;
4. utiliser des versions stables, documenter les choix et verrouiller le Gradle Wrapper ;
5. revérifier les exigences et recommandations en vigueur avant une release majeure.

- Versions Android : https://developer.android.com/about/versions
- Releases du plugin Android Gradle : https://developer.android.com/build/releases/gradle-plugin
- Wrapper Gradle : https://docs.gradle.org/current/userguide/gradle_wrapper.html

## 5. Dépendances et provenance

La présence d'une bibliothèque candidate dans ce document n'est pas son approbation. Examiner la licence source de la version exacte, les POM/métadonnées, les transitives, l'origine des artefacts, la maintenance, les besoins réseau et l'impact sur la compilation depuis les sources. Toute exclusion transitive doit être suivie de tests prouvant que les fonctions retenues continuent de fonctionner.

- Vérification des dépendances Gradle : https://docs.gradle.org/current/userguide/dependency_verification.html
- Inventaire du projet : `THIRD_PARTY.md`

## 6. F-Droid et builds reproductibles

Une licence libre est nécessaire mais ne garantit pas l'inclusion. Il faut vérifier toutes les dépendances et ressources, la disponibilité des sources, l'absence de SDK de suivi/publicité interdits, la capacité du serveur de build à construire le projet et, autant que possible, la reproductibilité.

- Politique d'inclusion : https://f-droid.org/docs/Inclusion_Policy/
- Builds reproductibles : https://f-droid.org/docs/Reproducible_Builds/
- Traitement des mises à jour : https://f-droid.org/en/docs/Update_Processing/

## 7. Versionnement et GitHub Releases

Le projet doit suivre Semantic Versioning pour les versions visibles, documenter les changements dans `CHANGELOG.md` et lier chaque release à un tag immuable. Le `versionName` Android correspond à la version du tag sans son préfixe `v`, tandis que `versionCode` est géré comme un entier strictement croissant. La stratégie exacte est définie dans `docs/RELEASING.md`.

- Semantic Versioning : https://semver.org/lang/fr/
- Keep a Changelog : https://keepachangelog.com/fr/1.1.0/
- Android versioning : https://developer.android.com/studio/publish/versioning
- GitHub Releases : https://docs.github.com/repositories/releasing-projects-on-github/about-releases
- Sécurité GitHub Actions : https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions

## 8. Bibliothèques candidates — non approuvées

### `dav4jvm`

Projet amont : https://github.com/bitfireAT/dav4jvm

Candidate WebDAV/CardDAV Java/Kotlin. Vérifier au moment du choix la version exacte, la licence, les releases et l'origine des artefacts, Ktor et ses moteurs, les transitives et la possibilité d'intégrer la source dans le build F-Droid. Ne jamais supposer qu'un artefact est sur Maven Central sans vérifier ses coordonnées et le dépôt qui le sert.

### `ez-vcard`

Projet amont : https://github.com/mangstadt/ez-vcard

Candidate vCard. Vérifier le fichier de licence et les en-têtes de la version exacte, le POM et les transitives réelles. N'exclure ni `jsoup`, ni Jackson, ni FreeMarker sur la seule base d'un audit antérieur : examiner les fonctionnalités nécessaires et exécuter les tests de parsing/écriture après chaque exclusion.

## Limites

Cette synthèse aide au cadrage mais ne remplace pas l'examen des sources primaires au moment de l'implémentation. Aucun choix d'architecture ou de dépendance ne doit être considéré comme approuvé uniquement parce qu'il apparaît ici.
