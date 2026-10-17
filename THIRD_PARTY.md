# Bibliothèques et ressources tierces

Cet inventaire doit rester synchronisé avec les dépendances réellement résolues et les ressources effectivement distribuées. Ne pas marquer un composant « approuvé » tant que sa version exacte, son origine, sa licence, sa maintenance, ses transitives et son usage n'ont pas été vérifiés.

## Règles obligatoires

- Priorité aux composants libres, maintenus, documentés et compatibles avec une distribution à partir des sources via F-Droid.
- Pas de SDK propriétaire, de publicité, d'analytics, de télémétrie, de dépendance obligatoire à Google Play Services ou de service fermé pour les fonctions essentielles.
- Utiliser des versions stables et figées ; éviter les versions dynamiques, snapshots et builds non reproductibles.
- Examiner les dépôts de publication et la provenance de chaque artefact, y compris les plugins Gradle et les outils de build.
- Après un bootstrap contrôlé, configurer la vérification Gradle des dépendances (sommes de contrôle et, lorsqu'elles sont disponibles et vérifiables, signatures). Ne pas approuver automatiquement un premier téléchargement non vérifié.
- Pour chaque police, icône, illustration, image, son ou autre ressource, consigner l'origine, la licence, l'attribution et les droits de redistribution.
- Réexaminer les licences, versions et règles F-Droid avant chaque release.
- Ne pas exclure une dépendance transitive simplement pour réduire le poids ou la surface d'attaque sans vérifier que les fonctions nécessaires et les tests restent valides.

## Dépendances de l'application

| Composant | Version réellement résolue | Usage | Licence source vérifiée | Origine vérifiée | Transitives examinées | Décision/statut |
|---|---|---|---|---|---|---|
| À renseigner après le choix et l'inspection du build Gradle | — | — | — | — | — | Aucune dépendance approuvée à ce stade |

## Bibliothèques candidates étudiées, non approuvées

La présence d'un candidat dans ce tableau ne signifie pas qu'il est intégré ou approuvé.

| Candidat | Usage possible | Point de contrôle | Référence |
|---|---|---|---|
| `dav4jvm` | WebDAV/CardDAV en Kotlin/JVM | Vérifier la version exacte, la provenance de l'artefact, la licence source, Ktor, les transitives et la compatibilité du build source F-Droid. | https://github.com/bitfireAT/dav4jvm |
| `ez-vcard` | Lecture/écriture vCard 2.1, 3.0 et 4.0 | Vérifier le fichier de licence de la version distribuée, le POM, les transitives et les fonctions effectivement utilisées avant toute exclusion. | https://github.com/mangstadt/ez-vcard |

Une bibliothèque de protocole peut être réutilisée sans rendre l'application d'origine obligatoire. Les obligations de licence et la chaîne de compilation doivent rester documentées.

## Polices, icônes et ressources

| Ressource | Version/origine | Licence | Attribution requise | Fichier de licence conservé | Statut |
|---|---|---|---|---|---|
| À compléter seulement si une ressource est retenue | — | — | — | — | Non approuvé |

## Contrôles par release

Avant chaque publication, comparer cet inventaire aux dépendances réellement résolues dans le build tagué. Tout nouvel artefact doit avoir une provenance et une licence vérifiées. Conserver les rapports de dépendances et de licences comme preuve CI ; ne publier aucun secret ni aucun fichier local sensible avec ces rapports.

Les rapports automatiques aident à trouver les composants, mais ne remplacent pas la vérification des licences des sources, des fichiers réellement distribués, des exceptions et des conditions de redistribution.
