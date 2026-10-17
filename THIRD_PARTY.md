# Bibliothèques et ressources tierces

Cet inventaire doit être tenu à jour avec les dépendances réellement compilées et les ressources distribuées. Ne pas marquer un composant « approuvé » tant que sa version, son origine, sa licence, sa maintenance et ses dépendances transitives n'ont pas été vérifiées.

## Règles

- Priorité aux composants libres, maintenus, documentés et compatibles avec une distribution source via F-Droid.
- Pas de SDK propriétaire, publicité, analytics, télémétrie, dépendance obligatoire à Google Play Services ou service fermé pour les fonctions essentielles.
- Utiliser des versions stables et figées. Éviter les versions dynamiques, snapshots et builds non reproductibles.
- Examiner les dépendances transitives et le dépôt qui fournit chaque artefact.
- Considérer la vérification Gradle des dépendances (empreintes, éventuellement signatures) après un bootstrap contrôlé ; ne pas approuver automatiquement des artefacts inconnus.
- Pour chaque police, icône, illustration, image, son ou autre ressource, consigner l'origine et les droits de redistribution.
- Réexaminer les licences et la politique F-Droid avant les versions publiées.

## Dépendances de l'application

| Composant | Version réellement résolue | Usage | Licence vérifiée | Origine | Transitives examinées | Statut / note |
|---|---|---|---|---|---|---|
| [À renseigner après choix et inspection de Gradle] | — | — | — | — | — | Non approuvé |

## Bibliothèques candidates étudiées, non approuvées

Ces pistes doivent être réévaluées au moment du choix : leur présence ici ne signifie pas qu'elles sont incluses dans le projet.

| Candidat | Usage possible | Licence déclarée / point de contrôle | Référence |
|---|---|---|---|
| `dav4jvm` | WebDAV/CardDAV en Kotlin/JVM | MPL-2.0 selon le dépôt amont ; vérifier version publiée, dépendances Ktor, disponibilité de l'artefact et compatibilité F-Droid avant adoption | https://github.com/bitfireAT/dav4jvm |
| `ez-vcard` | Lecture/écriture vCard 2.1, 3.0 et 4.0 ; import/export | FreeBSD/BSD selon les métadonnées du projet ; vérifier la licence exacte distribuée et les dépendances transitives de la version retenue | https://github.com/mangstadt/ez-vcard |

Une bibliothèque de protocole peut être réutilisée indépendamment de l'application qui l'a créée ; son utilisation ne doit pas rendre l'installation de DAVx⁵ obligatoire. Les obligations de licence et la chaîne de compilation restent à vérifier.

## Polices, icônes et ressources

| Ressource | Version/origine | Licence | Attribution requise | Fichier de licence conservé | Statut |
|---|---|---|---|---|---|
| [À compléter seulement si une ressource est retenue] | — | — | — | — | Non approuvé |

## Génération de l'inventaire

Après création des fichiers Gradle, relever les versions réellement résolues et inspecter les artefacts transitifs, puis comparer avec les déclarations amont et leurs licences. Les rapports automatiques facilitent l'inventaire mais ne remplacent pas la vérification des licences, des ressources ou des conditions de redistribution.
