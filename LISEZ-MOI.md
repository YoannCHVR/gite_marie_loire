# Site vitrine — Gîte Marie de la Loire

Site **1 page** clé en main, responsive (mobile/tablette/ordinateur), conforme au descriptif du besoin.

## Ouvrir le site en local
La galerie photo est chargée depuis `gallery.json` : un double-clic sur `index.html` ne suffit plus
(les navigateurs interdisent la lecture de fichiers voisins depuis `file://`).
Ouvrez un terminal **dans ce dossier** (`Site Marie de la Loire`) et lancez :

```
python3 -m http.server 8123
```

puis rendez-vous sur <http://localhost:8123>. (Le port 8000, choisi par défaut, est souvent
déjà pris — par Docker notamment. N'importe quel port libre convient.)

Pour arrêter le serveur : `Ctrl+C`.

## Contenu intégré
- Présentation de la maison + tous les équipements (livret d'accueil)
- Galerie photos **de jour** (19) et **de nuit** (19), avec visionneuse plein écran
- **Vidéo drone** de la propriété
- **Plan d'intérieur** (maison + préau) + composition
- **Notices** des équipements : spa, chauffage/clim, sèche-serviettes, lave-vaisselle, micro-ondes, aire de jeux
- **Aux alentours** : châteaux, parcs, nature, caves, marchés, restaurants
- **Accès** : adresse, point GPS, carte, itinéraire Google Maps, distances
- Liens **Airbnb**, **Booking**, **Festivini**, **Office de tourisme de Saumur**, **Google Business**
- **QR code** renvoyant vers le site
- Mentions propriétaire FLB IMMO (SIRET)

## À personnaliser (5 minutes)
Ouvrez `index.html` avec un éditeur de texte et cherchez le mot **PERSO** :

1. **Liens Airbnb / Booking** — bloc `BOOKING_LINKS` : remplacez les `"#"` par vos URL d'annonces.
2. **Google Business Profile** — variable `GOOGLE_BUSINESS` : collez le lien de votre fiche.
3. **Point GPS exact** — section Accès : remplacez `47.2275, 0.0560` par vos coordonnées précises (relevées sur Google Maps) ; ajustez aussi le `marker=` de la carte.
4. **Tutos vidéo des notices** — remplacez les liens des boutons « Voir le tuto vidéo ».
5. **Dépôt GitHub** — fichier `admin/config.yml`, clé `repo` : mettez `propriétaire/dépôt`.

Les photos de la galerie ne se modifient plus à la main : voir la section suivante.

## Gérer les photos de la galerie (interface `/admin`)

La galerie « Jour & nuit » est pilotée par **Sveltia CMS**, un CMS qui écrit directement dans le dépôt
GitHub. Aucun serveur à installer, aucun mot de passe stocké dans le site.

- Les photos vivent dans `assets/img/gallery/`.
- Leur ordre et leurs titres vivent dans `gallery.json`.
- L'interface se trouve à l'adresse `https://VOTRE-SITE/admin/`.

### Mise en service (une seule fois)

1. Le site doit être hébergé depuis le dépôt GitHub (GitHub Pages, Netlify ou Cloudflare Pages).
   Un hébergement FTP classique ne convient pas : le CMS a besoin d'écrire dans le dépôt.
2. Renseignez `repo:` dans `admin/config.yml`.
3. Choisissez une méthode de connexion :
   - **Jeton d'accès** — le plus simple. Sur l'écran de connexion, « Sign In Using Access Token » :
     GitHub ouvre une page où le jeton est pré-configuré, vous le collez, c'est fini.
     Convient si vous seul gérez le site.
   - **Bouton « Se connecter avec GitHub »** — plus confortable pour une personne non technique.
     Il faut déployer https://github.com/sveltia/sveltia-cms-auth sur Cloudflare Workers (gratuit),
     puis renseigner `base_url:` dans `admin/config.yml`.

### Tester l'interface en local, sans rien publier

Sveltia sait travailler directement sur le dépôt de votre disque, sans passer par GitHub.
C'est le meilleur moyen d'essayer l'interface avant de la mettre en ligne.

1. Lancez le serveur depuis le dossier `Site Marie de la Loire` :
   `python3 -m http.server 8123`
2. Ouvrez <http://localhost:8123/admin/> dans **Chrome, Edge ou Brave**.
   Ce mode s'appuie sur la *File System Access API*, absente de Firefox et de Safari.
3. Cliquez sur **« Work with Local Repository »**, puis sélectionnez le dossier
   **`gite_marie_loire`** — la racine du dépôt, pas le dossier du site.
   Les chemins du `config.yml` sont écrits par rapport à cette racine.
4. Modifiez la galerie : les fichiers de votre disque changent réellement.
   Rien n'est publié tant que vous ne faites pas `git commit` puis `git push` vous-même.

Rechargez <http://localhost:8123> pour voir le résultat sur le site.

### Au quotidien

Ouvrez `/admin/`, connectez-vous, cliquez sur **Galerie photos**. Vous pouvez ajouter une photo,
changer son titre, la déplacer dans la liste ou la supprimer, pour le jour comme pour la nuit.
Chaque enregistrement crée un commit ; le site se reconstruit tout seul en une minute environ.

> **Redimensionnez vos photos avant de les envoyer** (1600 px de large suffisent largement).
> Le CMS ne les compresse pas : une photo brute de smartphone alourdit la page pour tous les visiteurs.

## Mettre le site en ligne
1. Choisir un nom de domaine (ex. *gitemariedelaloire.fr*) + un hébergement (OVH, Hostinger, Netlify…).
2. Envoyer le dossier complet (`index.html` + `assets/`).
3. Générer le QR code définitif pointant vers l'adresse finale (le QR actuel pointe vers une adresse provisoire).

> La vidéo (`assets/video/drone.mp4`, ~99 Mo) est lourde : pour un site en ligne, il est recommandé de l'héberger sur YouTube/Vimeo et de l'intégrer, pour accélérer le chargement.

---
*Réalisé à partir du « Descriptif du besoin » et des documents fournis (livret d'accueil, photos, vidéo, notices, plan).*
