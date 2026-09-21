# Confidentialité et conservation — V0.8

ChatGarde n'exploite **aucun backend de navigation, aucun proxy propriétaire, aucune télémétrie et aucun historique distant**. Les décisions de sécurité et les signaux utilisés par le moteur restent sur l'appareil.

## Ce qui reste local

| Donnée / signal | Traitement | Persisté par ChatGarde | Envoyé à ChatGarde / éditeur |
|---|---|---:|---:|
| Texte analysé par le moteur | mémoire uniquement | non | non |
| Catégorie / score de risque | mémoire uniquement | non | non |
| Instantané visuel analysé dans le navigateur protégé | mémoire uniquement pendant l’inférence | non | non |
| Domaine DNS évalué | mémoire uniquement | non | non |
| Sélection Family Controls Apple | stockage local des jetons opaques nécessaires à la politique | oui, localement | non |
| Code parent | Keychain iOS / Keystore Android | oui, localement | non |
| Règles embarquées | ressource signée de l'application | oui | non |
| Historique de navigation de ChatGarde | aucun journal d'activité | non | non |

## Navigation Web

La présence d'un navigateur protégé change une précision importante : **charger une page Web nécessite naturellement de contacter le site demandé**. Une recherche lancée dans le navigateur protégé contacte également le moteur de recherche configuré. Ce trafic est le trafic Web normal demandé par l'utilisateur ; ChatGarde n'en envoie pas une copie à un serveur de l'éditeur.

- **iPhone / iPad** : le navigateur protégé utilise un stockage WebKit non persistant. Le texte, les métadonnées média et, lorsque la politique Apple l’autorise, un instantané visuel sont analysés localement avant affichage. Sensitive Content Analysis ne doit produire aucune télémétrie applicative sur ses résultats.
- **Android** : le navigateur protégé analyse localement le texte, les métadonnées média et un instantané visuel de son propre WebView avec un modèle LiteRT embarqué. L’instantané et le score ne sont pas persistés. Cookies, cache, WebStorage, historique interne et données de formulaire sont nettoyés lorsque ce navigateur est détruit.
- **DNS Android** : une requête autorisée est transmise au résolveur réseau actuellement utilisé par Android pour pouvoir résoudre le domaine. Elle n'est pas dupliquée vers un service ChatGarde.
- Les sites visités et les fournisseurs réseau/recherche peuvent naturellement recevoir les données nécessaires au fonctionnement normal du Web selon leurs propres politiques.

Le produit n'utilise **ni interception TLS, ni certificat racine, ni MITM, ni Accessibility détourné** pour lire le contenu d'autres applications.

Le modèle visuel Android est récupéré uniquement au moment du build depuis une révision upstream épinglée puis incorporé à l’APK. L’application installée n’a pas besoin de télécharger ce modèle pour effectuer ses inférences.

## Absence de surveillance

Il n'existe pas de tableau de bord affichant l'activité de l'enfant, d'historique des sites ou recherches consultables par le parent, de compte enfant ChatGarde, de SDK publicitaire/analytique, de crash reporter tiers contenant l'activité ni d'endpoint propriétaire recevant URL, texte, recherche ou classification.

Le parent ou l'administrateur contrôle la **politique de protection**, pas un journal de surveillance.

## Règles de développement

1. Ne jamais journaliser domaines, URL, recherches, texte analysé, jetons Family Controls ou classifications.
2. Ne jamais sauvegarder les entrées du moteur dans les préférences, fichiers, crash reports ou analytics.
3. Toute nouvelle dépendance réseau doit être justifiée et auditée.
4. Les règles embarquées sont mises à jour avec une version signée de l'application ; aucun téléchargement silencieux de profil utilisateur n'est requis par le moteur actuel.
5. Les tests automatisés utilisent des signaux synthétiques et des domaines réservés.
6. Le trafic Web normal du navigateur protégé doit rester séparé de toute télémétrie applicative.

## Vérification avant publication

Avant une release publique : exécuter les audits statiques du dépôt ; tester en mode avion les décisions purement locales ; capturer le trafic de l'application en laboratoire et vérifier l'absence d'endpoint propriétaire/télémétrie ; inspecter le conteneur après des décisions ; vérifier les Privacy Manifests et déclarations App Store / Google Play avec les binaires finaux ; vérifier séparément les comportements de confidentialité de WebKit/WebView et du moteur de recherche configuré.

La promesse produit doit donc être formulée ainsi : **« l'analyse et la décision de protection sont locales ; ChatGarde ne transmet ni ne conserve votre historique de navigation »**, et non « aucune donnée réseau ne quitte jamais l'appareil », ce qui serait incompatible avec l'accès normal au Web.
