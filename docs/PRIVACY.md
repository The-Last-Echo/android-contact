# Confidentialité et flux des données

## Engagements attendus

- Pas de publicité, profilage, analytique, télémétrie ou rapport automatique d'erreurs par défaut.
- Pas de serveur central du projet et aucun compte propriétaire central.
- Les contacts restent sur l'appareil, dans la source de stockage choisie, jusqu'à une exportation déclenchée par l'utilisateur ou la configuration et l'activation explicite d'un compte de synchronisation.
- Chaque compte CardDAV ne reçoit que les contacts qui lui sont associés selon l'architecture approuvée. Les destinations et le sens de synchronisation doivent être visibles.
- Une fonctionnalité réseau ne doit pas être ajoutée simplement parce qu'une bibliothèque en apporte une.

## Flux à documenter avant publication

| Donnée | Stockage/source | Destination éventuelle | Déclencheur | Statut |
|---|---|---|---|---|
| Noms, téléphones, e-mails, adresse, notes | Contacts Provider Android ou base privée selon la décision | Compte CardDAV configuré uniquement | Synchronisation explicitement activée | À confirmer par l'architecture et le code |
| Photo de contact | Source locale ou contact Android | Seulement si incluse dans une vCard que l'utilisateur synchronise | Action de création/modification et sync | À tester selon le support serveur |
| Identifiants de serveur | Stockage protégé à choisir | Serveur configuré lors des requêtes d'authentification | Synchronisation | À concevoir et vérifier |
| Fichier `.vcf` | URI choisie par le système de fichiers | Emplacement choisi par l'utilisateur pour l'export | Action d'export explicite | À implémenter |
| Journaux de diagnostic | Journaux locaux, réduits | Aucun envoi automatique | Seulement lorsque nécessaire | À définir ; aucune donnée personnelle |

## Permissions prévues, sous réserve de l'architecture

- `READ_CONTACTS` et `WRITE_CONTACTS` si le Contacts Provider Android est choisi. Ce sont des permissions sensibles ; demander au moment d'une action pertinente, gérer le refus et offrir une explication claire.
- `INTERNET` uniquement si la synchronisation CardDAV directe est dans l'application. Cette permission n'est pas équivalente au consentement de synchronisation : aucun compte ne doit être appelé avant sa configuration explicite.
- Les actions via les applications de téléphone, SMS et e-mail doivent utiliser des intents système lorsque cela répond au besoin, pour éviter des permissions téléphoniques inutiles.
- Le sélecteur de documents et le sélecteur de photos doivent être privilégiés aux permissions d'accès large au stockage ou aux médias lorsqu'ils conviennent.

Le manifeste final et le manifeste fusionné de publication doivent être contrôlés. Les permissions réelles peuvent changer si l'architecture est modifiée.

## Limites et transparence

Le fait qu'une app ne comporte pas de SDK de suivi ne suffit pas à prouver l'absence de transmissions. Avant les publications, examiner les appels réseau, le manifeste, les transitives et les journaux. Mettre à jour cette page dès qu'un flux de données change.
