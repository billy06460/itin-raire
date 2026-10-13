# Itinéraire & Péages

Calcul d'itinéraire avec détection des gares de péage sur le trajet (jeu de
données officiel embarqué), packagée en app Android via
[Capacitor](https://capacitorjs.com/) et construite automatiquement par
GitHub Actions.

## Mettre le dépôt sur GitHub

```bash
cd capacitor-app
git init
git add .
git commit -m "Init app Itinéraire & Péages"
git branch -M main
git remote add origin https://github.com/<ton-compte>/<nom-du-repo>.git
git push -u origin main
```

## Récupérer l'APK

Onglet **Actions** du dépôt : le workflow se déclenche à chaque push sur
`main` (ou via "Run workflow" pour le relancer manuellement). L'APK est
ensuite disponible dans la section **Artifacts** du run, sous le nom
`itineraire-peages-debug-apk`.

## À savoir pour le moment

La signature persistante a été retirée pour le moment (sur demande) : chaque
build est signé avec une clé de debug générée à la volée par le runner
GitHub. Résultat, Android peut refuser de réinstaller une nouvelle version
par-dessus l'ancienne ("conflit avec un package existant") — dans ce cas,
désinstalle l'app avant de réinstaller le nouvel APK.

Icône : par défaut, celle fournie par Capacitor, sauf si tu as ajouté
`resources/icon.png` (voir plus haut).


## Convoi : suivi à distance

Dans l'app, le bouton **👥 Convoi** permet de créer un convoi (un code unique)
ou d'en rejoindre un. Pendant un trajet, chaque véhicule du convoi partage sa
position ; elle s'affiche en direct sur la page `www/suivi.html` (une carte
par convoi : le lien contient le code, `suivi.html?c=CODE`).

Pour héberger la page de suivi gratuitement avec GitHub Pages :

1. Dépôt GitHub → **Settings → Pages → Source : GitHub Actions**.
2. Pousse le projet (le workflow `pages.yml` publie `suivi.html`).
3. L'adresse est `https://<ton-compte>.github.io/<nom-du-repo>/suivi.html`.
   Colle-la dans l'app : **👥 Convoi → Adresse de la page de suivi**.

Les positions transitent par des relais MQTT publics gratuits (aucun compte à
créer). Le code du convoi fait office de mot de passe : ne le donne qu'aux
participants. Pour utiliser ton propre relais, définis `window.CONVOY_BROKERS`
(liste d'adresses `wss://…`) avant le chargement de la page.


## GPS en arrière-plan

Le suivi GPS utilise le plugin `@capacitor-community/background-geolocation`
(service Android au premier plan, notification « Suivi GPS du trajet en
cours »). Le trajet et le partage de convoi continuent écran éteint ou app
réduite.

Au premier trajet, Android demande la localisation : choisis **Toujours
autoriser** (et accepte les notifications). Pour éviter que le système coupe
l'app, règle aussi **Batterie → Sans restriction** (raccourci : Réglages de
l'app → « Ouvrir les réglages de l'app »).

Dans un navigateur (sans l'APK), l'app retombe automatiquement sur le GPS
standard, qui s'arrête en arrière-plan.


## Structure du code

Tout est en JavaScript ; les deux `.html` ne sont que des coquilles de quelques lignes.

- `www/index.html` : l'application complète en UN SEUL fichier HTML (styles, interface et code intégrés). `www/mappy-sdk/` : SDK Mappy (à garder à côté).
- `www/suivi.html` : page de suivi (un seul fichier).

## Icône

Les images sont dans `resources/` (`icon-only.png`, `icon-foreground.png`, `icon-background.png`) ; le workflow génère toutes les tailles Android avec `capacitor-assets`. Pour changer l'icône, remplace ces fichiers (1024×1024).

## Suivi écran éteint

- L'app utilise un service de localisation au premier plan (notification « Suivi GPS du trajet en cours ») et `android.useLegacyBridge` (dans `capacitor.config.json`) pour que les positions continuent d'arriver à l'app écran éteint.
- Sur le téléphone : Réglages > Applications > Itinéraire & Péages > Batterie > « Sans restriction », et localisation « Toujours autoriser ».


## Mises à jour sans désinstaller

- `signing/itineraire.keystore` est la clé de signature FIXE de l'app (alias `itineraire`, mot de passe `itineraire2026`). **Ne la supprime jamais et garde-la dans le dépôt** : tant que c'est la même clé, Android installe la nouvelle APK par-dessus l'ancienne.
- Le numéro de version monte tout seul à chaque build (numéro d'exécution GitHub + 1000), Android accepte donc toujours la mise à jour.
- **Une seule fois** : si une ancienne APK (signée avec l'ancienne clé de debug) est installée, désinstalle-la avant d'installer celle-ci. Ensuite, plus jamais.
- La clé est dans le dépôt : garde le dépôt **privé** si tu veux qu'elle reste confidentielle (sinon n'importe qui pourrait signer une fausse mise à jour de ton app).
- Le workflow affiche l'empreinte SHA-256 de signature : elle doit être identique à chaque build (`EF:71:C3:D4:…`).