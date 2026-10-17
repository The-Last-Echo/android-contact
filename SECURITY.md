# Sécurité

## Engagement

La sécurité des données de contact et des identifiants de synchronisation est une exigence de premier ordre. Le projet vise une approche de privilège minimal, le traitement local par défaut et aucune collecte cachée.

## Signaler une vulnérabilité

**Canal privé de signalement à configurer et tester avant la première release stable.** Ne publiez pas les détails exploitables d'une vulnérabilité non corrigée dans une issue publique. Ne pas inventer d'adresse de contact ou promettre un délai de réponse avant que le canal et la capacité de suivi soient établis.

## Données de contact et synchronisation

- Ne jamais journaliser les noms, adresses, numéros de téléphone, e-mails, notes privées, photos, jetons ni mots de passe.
- Aucune donnée de contact ne doit quitter l'appareil, sauf transfert explicitement demandé ou synchronisation vers un compte serveur configuré par l'utilisateur.
- Toute synchronisation réseau doit utiliser TLS avec validation normale des certificats. Aucun contournement TLS ne doit exister dans une build de production.
- Ne pas conserver de mot de passe en clair dans DataStore, SharedPreferences, Room, les exports ou les logs. Choisir et documenter un mécanisme de stockage des secrets adapté aux API Android retenues avant l'implémentation.
- Valider les fichiers d'importation vCard, les URI, les réponses WebDAV, les champs serveur et les tailles d'images ; borner les ressources consommées par les entrées malformées.
- Une réponse réseau incomplète ou une erreur temporaire ne doit jamais être interprétée comme la suppression de tous les contacts distants.
- Définir explicitement les règles de conflit avant d'autoriser la synchronisation bidirectionnelle.
- Ne pas ajouter de publicité, d'analyse, de télémétrie ou de rapport d'erreurs distant sans décision produit explicite et consentement éclairé. Par défaut, aucune télémétrie.
- Les composants Android ne doivent pas être exportés sauf besoin précis et justifié ; protéger les fournisseurs, services, récepteurs et points d'entrée concernés.

## Dépendances et chaîne d'approvisionnement

- Vérifier la provenance, la licence exacte, la maintenance, les versions transitives et les besoins réels de chaque dépendance directe ou indirecte.
- Ne pas considérer une bibliothèque comme approuvée sur la seule base de son nom, d'un badge de licence ou de ses métadonnées. Conserver les preuves et décisions dans `THIRD_PARTY.md`.
- Utiliser des versions figées et mettre en place la vérification Gradle des dépendances après un bootstrap contrôlé. Réexaminer les mises à jour et leurs changements de licence.
- Ne pas ajouter de SDK fermé, de dépendance obligatoire à Google Play Services ni de service de suivi.
- Les rapports de sécurité automatisés ne remplacent pas l'examen des licences, du code, des ressources et des flux réseau.

## Sécurité du dépôt et de la CI

- Ne jamais committer une clé de signature, un mot de passe, un jeton, un secret CardDAV, une clé privée ou un fichier de configuration sensible.
- Ne jamais afficher les secrets dans les logs ou les artefacts CI. Les secrets de release ne doivent être disponibles que dans le job de publication autorisé, jamais dans le contexte d'une pull request non fiable.
- Donner au `GITHUB_TOKEN` uniquement les permissions nécessaires. Séparer les tests de pull request du job qui peut créer des releases.
- Épingler les actions tierces GitHub Actions à un SHA de commit complet vérifié lorsque c'est possible ; ne pas copier un SHA sans vérifier qu'il appartient bien au dépôt officiel de l'action.
- La release doit être construite à partir du commit exact désigné par le tag, après les contrôles requis. Ne pas téléverser un APK construit depuis un état de travail différent.
- Protéger la branche `main`, passer par des pull requests et empêcher les pushes forcés. Les protections de branche et les contrôles obligatoires doivent être configurés dans les paramètres GitHub ; leur simple mention dans un document ne les active pas.
- La clé de publication Android doit disposer d'une sauvegarde hors ligne protégée et d'une procédure de récupération documentée. La perte de la clé peut empêcher les mises à jour en place des APK GitHub.
- Ne jamais remplacer ni déplacer un tag de release publié. Une correction après publication utilise un nouveau numéro de version.

## Vérifications avant release

- [ ] Toutes les permissions du manifeste et du manifeste fusionné sont justifiées.
- [ ] Les composants exportés, intents, fournisseurs, services et récepteurs sont revus.
- [ ] Les données personnelles sont absentes des logs, rapports publiés et artefacts temporaires.
- [ ] Les dépendances transitives, licences et ressources sont recensées dans `THIRD_PARTY.md`.
- [ ] Les refus/révocations de permissions et les pannes réseau sont testés.
- [ ] Les imports vCard malformés et les échecs partiels de synchronisation sont testés avant d'activer ces fonctions.
- [ ] Aucun secret n'est présent dans le diff, l'historique pertinent ou les artefacts.
- [ ] Les workflows CI et release ont réussi depuis le commit tagué.
- [ ] Les APK et leurs signatures/versionnements sont vérifiés ; les SHA-256 sont générés après la finalisation des fichiers.
- [ ] Le canal privé de signalement est configuré et testé avant la première version stable.
- [ ] La politique F-Droid actuelle et la construction depuis les sources sont réévaluées avant soumission.

## Limites

Un document de politique ou un scan automatisé ne prouve pas que l'application est sécurisée. Toute déclaration de sécurité doit préciser les vérifications réellement exécutées et les limites connues.

## Références

- GitHub Actions — usage sécurisé : https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions
- Gradle — vérification des dépendances : https://docs.gradle.org/current/userguide/dependency_verification.html
- F-Droid — politique d'inclusion : https://f-droid.org/docs/Inclusion_Policy/
- F-Droid — builds reproductibles : https://f-droid.org/docs/Reproducible_Builds/
- Android — signature des applications : https://developer.android.com/studio/publish/app-signing
