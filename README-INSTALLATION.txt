BUSDirect — GitHub public + GitHub Pages
========================================

OBJECTIF
- Dépôt public GitHub pour le code et la configuration.
- GitHub Pages pour publier l'application, les modules et les paquets GTFS autorisés.
- GitHub Actions pour télécharger les GTFS déclarés, récupérer les bibliothèques épinglées et publier le site.
- IndexedDB sur l'appareil pour mettre en cache les données chargées.
- GTFS-RT direct depuis le navigateur quand le fournisseur autorise CORS; relais HTTPS facultatif sinon.
- Transitous reste la source de parcours de l'application actuelle pendant la migration.

INSTALLATION GITHUB (depuis iPhone ou ordinateur)
DÉPÔT CIBLE
- Propriétaire GitHub : exodusys
- Dépôt public : BusDirect
- URL du dépôt : https://github.com/exodusys/BusDirect
- URL GitHub Pages attendue : https://exodusys.github.io/BusDirect/

1. Créer le dépôt public https://github.com/exodusys/BusDirect (s’il n’existe pas déjà).
2. Envoyer tout le contenu de ce dossier à la racine du dépôt, branche "main".
3. Ouvrir Settings > Pages.
4. Dans Build and deployment > Source, choisir "GitHub Actions".
5. Ouvrir Actions et lancer "BusDirect — préparer et publier sur GitHub Pages" (workflow_dispatch), ou attendre le premier push.
6. L’adresse attendue est https://exodusys.github.io/BusDirect/. Elle ne sera active qu’après une première publication réussie.

IMPORTANT : les chemins de modules sont relatifs et le bootstrap est compatible avec une URL GitHub Pages de projet (avec /DEPOT/).
Aucun secret GitHub ni mot de passe n'est requis pour la publication Pages.

AJOUTER UNE SOURCE GTFS
Modifier busdirect/config/sources.json. Exemple de structure (valeurs fictives à remplacer, ne pas publier tel quel) :
{
  "schemaVersion": 1,
  "sources": [{
    "id": "ch-operateur-exemple",
    "name": "Nom officiel de l'opérateur",
    "country": "CH",
    "enabled": true,
    "static": {
      "downloadUrl": "https://URL-OFFICIELLE/gtfs.zip",
      "path": "../data/static/ch/operateur.zip",
      "output": "static/ch/operateur.zip"
    },
    "realtime": {
      "directUrl": {
        "tripUpdates": "https://URL-RT-TRIP-UPDATES",
        "vehiclePositions": "https://URL-RT-VEHICLES",
        "alerts": "https://URL-RT-ALERTS"
      }
    }
  }]
}
- downloadUrl est l'URL HTTPS officielle utilisée par GitHub Actions.
- path est le chemin public relatif au catalogue, pour le navigateur.
- output est le chemin de stockage dans busdirect/data.
- Ne déclarer une source qu'après vérification de la licence, des conditions de redistribution, de la stabilité de l'URL, du format et de la fréquence de mise à jour.
- Le catalogue est volontairement vide : aucun flux européen n'a été inventé ou activé.

LIMITES ET GARDE-FOUS
- GitHub Pages : site publié limité à 1 Go; bande passante indicative souple de 100 Go/mois; limites anti-abus possibles. Le script arrête la publication GTFS à 850 Mio pour conserver une marge. Les limites officielles peuvent évoluer.
- Le dépôt source doit rester léger : les ZIP GTFS téléchargés ne sont pas commités. Ils sont récupérés par Actions au moment du build et ajoutés à l'artefact Pages.
- Si le volume ou le trafic européen dépasse durablement les capacités Pages, garder Pages pour l'application, le catalogue et les index, puis placer les gros paquets sur un stockage/CDN adapté. Ce n'est pas un VPS de calcul.
- Une publication Pages rend tous les fichiers publiés publics. Ne jamais placer de clé API privée, mot de passe, données personnelles ou secret dans le dépôt ou dans l'artefact.
- Une URL GTFS-RT ne fonctionne directement dans le navigateur que si CORS et l'authentification le permettent. GitHub Pages ne contourne pas CORS et n'est pas un proxy.
- Les bibliothèques sont récupérées depuis des versions précises lors du build. Pour une chaîne de production stricte, vérifier et verrouiller également leurs empreintes cryptographiques.

ÉTAT RÉEL DE CETTE INTÉGRATION
- Le HTML BusDirect fourni est conservé; seul le chargement du pont GTFS est ajouté.
- Le pont expose window.BusDirectData et les modules chargent les flux à la demande.
- Le cache IndexedDB et les bases de lecture GTFS/GTFS-RT constituent le socle.
- Les appels Transitous de l'application restent inchangés et continuent de produire les itinéraires et les données live.
- Le routeur CSA fourni est un prototype qui attend des connexions pré-indexées. Il ne convertit pas encore directement tous les fichiers GTFS bruts en horaires prêts à router.
- Ce dépôt n'est donc PAS encore un routeur autonome couvrant toute l'Europe. Il faut encore construire l'indexation GTFS complète (calendriers, exceptions, fuseaux horaires, fréquences, transferts, parcours piétons, identifiants multi-sources), puis raccorder les résultats au modèle de parcours utilisé par les écrans BusDirect.
- Il faut également ajouter un catalogue de flux réellement vérifiés et tester les licences, CORS, quotas, stabilité et qualité de chaque fournisseur.

ARCHITECTURE CIBLE
1. Catalogue régional : sources, pays, opérateurs, versions, URLs et règles de rafraîchissement.
2. Pipeline Actions : téléchargements, validation ZIP/GTFS, contrôle d'intégrité, manifeste, indexation par région et publication.
3. Client : détection de zone, téléchargement des seuls paquets utiles, vérification de version, cache IndexedDB et purge LRU.
4. Routage local : horaires calculés pour le fuseau/date locale, marche et transferts, correspondances inter-réseaux, accessibilité et classement.
5. Temps réel : GTFS-RT trip updates, véhicules et alertes; rapprochement avec les IDs du flux statique correspondant; cache et limites par source.
6. Secours : Transitous tant que le routeur local ou une source RT n'est pas validée.

NE PAS CONFONDRE
- GitHub Pages = hébergement de fichiers statiques, pas base de données, pas API dynamique, pas serveur de calcul.
- GitHub Actions = tâches planifiées et pipeline de publication, pas polling temps réel permanent.
- Le temps réel doit être demandé au fournisseur ou à un relais autorisé avec cadence raisonnable et cache partagé.
