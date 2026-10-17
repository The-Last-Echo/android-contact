# Sécurité

## Engagement

La sécurité des données de contact et des identifiants de synchronisation est une exigence de premier ordre. Le projet vise une approche de privilège minimal, le traitement local par défaut et aucune collecte cachée.

## Signaler une vulnérabilité

**Canal privé de signalement à configurer avant la première publication publique.** Ne publiez pas de détails exploitables d'une vulnérabilité non corrigée dans une issue publique.

Avant la première version stable, le responsable du projet doit activer et tester un moyen privé de recevoir les signalements (par exemple, la fonctionnalité de signalement privé de vulnérabilité de GitHub lorsqu'elle est disponible pour le dépôt). Ne pas inventer d'adresse e-mail ou promettre un délai de réponse avant que le canal et la capacité de suivi soient établis.

## Exigences de sécurité

- Ne jamais journaliser les noms, adresses, numéros de téléphone, e-mails, notes privées, photos, jetons ni mots de passe.
- Aucune donnée de contact ne doit quitter l'appareil, sauf transfert explicitement demandé par l'utilisateur ou synchronisation vers un compte serveur qu'il a configuré.
- Toute synchronisation réseau doit utiliser TLS avec validation normale des certificats. Ne pas proposer de contournement de validation TLS en production.
- Ne pas conserver de mot de passe en clair dans DataStore, SharedPreferences, Room, les fichiers de sauvegarde ou les journaux. Choisir et documenter un mécanisme de stockage des secrets adapté aux API Android actuelles avant l'implémentation.
- Ne jamais intégrer de secret, clé de signature, mot de passe de compte ou jeton dans le dépôt, les ressources, les tests publics ou l'APK.
- Demander uniquement les permissions nécessaires et expliquer leur utilité au moment opportun. N'utiliser ni `READ_CALL_LOG`, ni `WRITE_CALL_LOG`, ni `CALL_PHONE` si les besoins sont satisfaits avec les intents système.
- Valider les fichiers d'importation vCard, les URI, les réponses WebDAV, les champs fournis par le serveur et les tailles de photo. Limiter les ressources consommées par les entrées malformées.
- Les suppressions et les synchronisations doivent être résistantes aux erreurs réseau : une réponse incomplète ou une erreur serveur ne doit jamais être interprétée comme la suppression de tous les contacts distants.
- Définir explicitement les règles de conflit avant d'autoriser une synchronisation bidirectionnelle.
- Ne pas ajouter de SDK de publicité, d'analyse, de télémétrie ou de rapport d'erreurs réseau sans consentement éclairé et décision de produit. Par défaut, aucune télémétrie.
- Les composants Android n'ont pas à être exportés sauf besoin précis et justifié. Protéger les fournisseurs, services, récepteurs et points d'entrée concernés.

## Vérifications avant publication

- [ ] Vérifier toutes les permissions du manifeste et leur usage effectif.
- [ ] Inspecter le manifeste fusionné des variantes de publication.
- [ ] Examiner les dépendances directes et transitives, leurs licences et leur provenance.
- [ ] Vérifier les données écrites dans les journaux, rapports de test et fichiers exportés.
- [ ] Tester les permissions refusées ou révoquées et les pannes réseau.
- [ ] Tester les importations vCard malformées et les échecs partiels de synchronisation.
- [ ] Vérifier l'absence de secrets dans l'historique Git et les artefacts.
- [ ] Examiner la politique F-Droid en vigueur et vérifier le paquet réel.
- [ ] Configurer et tester le canal privé de signalement.

## Limites

Un document de politique ne prouve pas que l'application est sécurisée. Toute déclaration de sécurité doit préciser les vérifications réellement exécutées et les limites connues.
