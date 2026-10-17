# Fiche projet — Contacts

> État : cadrage initial. Les éléments marqués « proposé » ne sont pas encore des faits d'implémentation. Mettre ce document à jour lorsqu'une décision est approuvée.

## Identité

| Champ | Valeur | État |
|---|---|---|
| Nom du projet | Contacts — nom de travail | Nom définitif à choisir |
| `applicationId` | À définir avant la création de la première version distribuée | Bloquant avant publication ; difficile à changer ensuite |
| Plateforme | Android uniquement | Confirmé |
| Langage | Kotlin | Confirmé |
| UI | Jetpack Compose par défaut | Convention confirmée |
| Système visuel | Soft Motion, sous-ensemble choisi pour une app de contacts | Confirmé, sélection des composants à finir |
| Licence | `GPL-3.0-only` | Confirmé |
| Distribution | GitHub Releases et demande d'inclusion F-Droid | Confirmé comme objectif, éligibilité F-Droid non garantie |
| Environnement de développement | Linux | Confirmé |
| Publicité / télémétrie | Aucune publicité ni télémétrie cachée | Confirmé |
| Langues initiales envisagées | Anglais source/fallback et français traduit | Proposition alignée sur `DESIGN.md`, à valider au moment de créer les ressources |
| Version minimale Android | À rechercher et proposer | Ne pas inventer |
| Compile/target SDK | Valeurs stables en vigueur à vérifier au moment du bootstrap | Ne pas recopier une vieille version |
| Version Kotlin / AGP / Gradle / JDK | À fixer ensemble, puis verrouiller dans les fichiers de build | Ne pas deviner |
| Canal privé de sécurité | À configurer | Bloquant avant annonce publique de procédure de signalement |

## Objectif du produit

Gestionnaire de contacts Android léger, agréable, rapide et personnalisable ; gestion locale des contacts, intégration avec les actions système, import/export vCard et synchronisation directe CardDAV multi-comptes à terme.

## Données et intégrations prévues

- Données personnelles : identité, numéros, adresses e-mail et postales, notes, photo et autres champs usuels si le stockage retenu les permet.
- Source de données Android : décision d'architecture à confirmer ; voir `ARCHITECTURE.md`.
- Synchronisation : CardDAV direct, plusieurs comptes, opt-in, serveur configuré par l'utilisateur.
- Import/export : fichiers vCard `.vcf`, sélectionnés via le Storage Access Framework ; pas de permission générale de stockage.
- Appels/SMS/e-mails : intents système, avec confirmation dans l'application système appropriée ; pas d'envoi automatique.
- Réseau : aucune activité réseau non explicitement liée à un compte de synchronisation configuré.

## Décisions bloquantes ou importantes

| Sujet | État | Proposition actuelle | Pourquoi cela compte |
|---|---|---|---|
| Modèle des contacts | À approuver | Intégrer le Contacts Provider Android et ses comptes plutôt qu'une base privée isolée | Détermine visibilité dans les autres applications, permissions, synchronisation et risques de modification |
| Synchronisation | À approuver | Compte Android + SyncAdapter CardDAV si le Contacts Provider est retenu | Intégration native et multi-comptes, mais plus complexe qu'une synchronisation interne |
| Édition de comptes tiers | À décider | Afficher la source du contact et n'éditer que les sources réellement modifiables | Éviter des modifications/suppressions involontaires dans des comptes externes |
| Import/export vCard | Prévu | V1 ou phase suivante selon la taille du premier incrément | Essentiel pour portabilité et sauvegarde |
| Groupes/étiquettes | Phase ultérieure probable | Définir correspondance avec les groupes Android et `CATEGORIES` vCard après choix du modèle | Les groupes sont parfois propres à un compte ou serveur |
| Authentification CardDAV | À rechercher | Commencer avec un mécanisme compatible avec les serveurs personnels ciblés ; ne jamais transmettre les identifiants sans HTTPS vérifié | Les serveurs ont des modes d'authentification variés |
| Conflits de synchronisation | À définir avant synchronisation bidirectionnelle | Détection avec ETag/état distant, aucune politique destructive implicite | Prévenir les pertes de contacts |
| Signature GitHub/F-Droid | À accepter explicitement | Clés et canaux distincts sauf solution valide et acceptée par F-Droid | Les signatures différentes empêchent normalement une mise à jour directe d'une installation par l'autre |

## Références à conserver

- Contacts Provider : https://developer.android.com/identity/providers/contacts-provider
- Architecture Android : https://developer.android.com/topic/architecture/recommendations
- Compose lists : https://developer.android.com/develop/ui/compose/lists
- F-Droid inclusion policy : https://f-droid.org/docs/Inclusion_Policy/
- CardDAV (RFC 6352) : https://www.rfc-editor.org/rfc/rfc6352
- CardDAV/CalDAV discovery (RFC 6764) : https://www.rfc-editor.org/rfc/rfc6764
- vCard 4.0 (RFC 6350) : https://www.rfc-editor.org/rfc/rfc6350
