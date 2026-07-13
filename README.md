# Gîte Marie de la Loire — site vitrine

Site **statique une page**, responsive, sans build ni dépendances : pas de `npm install`, pas de
framework, pas de compilation. `index.html` contient le HTML, le CSS et le JS. Le contenu éditable
(galerie, équipements) vit dans des fichiers JSON, modifiables via une interface d'administration.

---

## Architecture en 30 secondes

```
index.html          tout le site : HTML + CSS + JS en un seul fichier
gallery.json        photos jour/nuit : ordre + titres        ← édité par le CMS
equipements.json    les 6 cartes « À votre disposition »     ← édité par le CMS
assets/             images, vidéo, favicon
admin/              interface de gestion (Sveltia CMS)
  index.html          charge le CMS depuis un CDN (version figée + empreinte SRI)
  config.yml          déclare les 2 modules éditables
robots.txt          écarte /admin/ des moteurs de recherche
```

Au chargement, `index.html` fait un `fetch()` de `gallery.json` et `equipements.json` et construit
la galerie et les cartes. Rien n'est codé en dur.

Le CMS écrit **directement dans le dépôt GitHub** : chaque enregistrement = un commit = un
redéploiement automatique. Il n'y a aucun serveur applicatif, aucune base de données.

---

## Environnement de développement

### Prérequis

| Outil | Pourquoi | Vérifier |
|---|---|---|
| **Python 3** | sert le site en local (aucun autre usage) | `python3 --version` |
| **Git** | cloner, versionner, publier | `git --version` |
| **Chrome, Edge ou Brave** | obligatoire pour le CMS en local | — |

> Firefox et Safari **ne conviennent pas** pour l'édition locale du contenu : ce mode s'appuie sur
> la *File System Access API*, qu'ils n'implémentent pas. Ils affichent le site sans problème,
> c'est seulement `/admin/` en mode local qui ne fonctionnera pas.

### Démarrer

```bash
git clone https://github.com/YoannCHVR/gite_marie_loire.git
cd gite_marie_loire
python3 -m http.server 8123 --bind 127.0.0.1
```

Le site tourne sur <http://localhost:8123>. Pour arrêter : `Ctrl+C`.

**Deux détails qui ne sont pas des détails :**

- **Ne double-cliquez pas sur `index.html`.** Depuis `file://`, les navigateurs interdisent la
  lecture des fichiers voisins : la galerie et les équipements resteraient vides. Il faut un
  serveur, même minimal.
- **Gardez `--bind 127.0.0.1`.** Sans lui, `http.server` écoute sur *toutes* les interfaces réseau :
  n'importe qui sur le même Wi-Fi peut alors parcourir l'intégralité de votre dossier de projet.
  Avec lui, seule votre machine peut s'y connecter.

Le port 8000 (défaut de Python) est souvent déjà pris ; 8123 évite la collision.

### Modifier le contenu en local, sans GitHub

Sveltia sait travailler directement sur les fichiers du disque, **sans jeton ni connexion**.

1. Serveur lancé, ouvrez <http://localhost:8123/admin/> dans Chrome, Edge ou Brave.
2. Cliquez sur **« Work with Local Repository »** et sélectionnez le dossier `gite_marie_loire`.
3. Éditez : les fichiers du disque changent réellement.
4. Vérifiez le rendu sur <http://localhost:8123>, puis `git commit` et `git push` pour publier.

Rien n'est mis en ligne tant que vous n'avez pas poussé vous-même.

### Modifier le code

Tout est dans `index.html`. Cherchez le mot **`PERSO`** pour les valeurs à renseigner :

1. **Liens Airbnb / Booking** — bloc `BOOKING_LINKS` : remplacer les `"#"` par les vraies URL.
2. **Google Business Profile** — variable `GOOGLE_BUSINESS`.
3. **Point GPS** — section Accès : remplacer `47.2275, 0.0560` et le `marker=` de la carte.
4. **Tutos vidéo des notices** — liens des boutons « Voir le tuto vidéo ».

Les **icônes** sont un sprite `<symbol>` en haut du `<body>`. Pour en ajouter une, créez le
`<symbol id="i-xxx">` puis ajoutez l'option correspondante dans `admin/config.yml`. Les deux
listes doivent rester synchronisées, sinon le CMS proposera une icône qui n'existe pas.

---

## Environnement de production

### Contrainte de départ

L'hébergement doit être **connecté au dépôt GitHub** : **GitHub Pages**, **Netlify** ou
**Cloudflare Pages**. Un hébergement FTP classique (OVH, Hostinger…) **ne convient pas** — le CMS a
besoin d'écrire dans le dépôt, et le site doit se reconstruire à chaque enregistrement.

### Mise en place

**1. Héberger le site.** Connectez le dépôt à l'hébergeur choisi. Il n'y a **pas de commande de
build** ni de dossier de sortie : le site est publié tel quel, depuis la racine.

Publiez **le dossier complet, `admin/` compris** — c'est l'interface de gestion, elle doit être
en ligne pour que la personne qui gère le contenu puisse s'en servir depuis n'importe où.

**2. Activer la connexion GitHub.** Sans cette étape, la seule façon de se connecter au CMS est de
coller un jeton personnel à la main — inutilisable pour une personne non technique.

- Déployez [sveltia-cms-auth](https://github.com/sveltia/sveltia-cms-auth) sur Cloudflare Workers
  (gratuit). Le dépôt détaille la marche à suivre ; il faut créer au passage une *OAuth App* GitHub.
- Reportez l'URL obtenue dans `admin/config.yml`, clé `base_url:` (la ligne existe déjà, en
  commentaire — il suffit de la décommenter).

Le secret de l'application OAuth reste dans le Worker, jamais dans le site.

**3. Donner l'accès à la personne qui gère le contenu.**

- Elle crée un compte GitHub et y active la **double authentification**.
- Vous l'invitez comme collaboratrice du dépôt : *Settings → Collaborators → Add people*.
- Elle ouvre `https://VOTRE-SITE/admin/` et clique sur **« Se connecter avec GitHub »**.

**4. Régénérer le QR code** (`assets/img/qr-site.png`) vers l'adresse définitive — l'actuel pointe
vers une adresse provisoire.

### Après le déploiement, vérifier

- `https://VOTRE-SITE/` — la galerie affiche 19 photos, la section « À votre disposition » affiche
  6 cartes. Si elles sont vides, c'est que les JSON ne sont pas servis : ouvrez la console.
- `https://VOTRE-SITE/admin/` — l'écran de connexion Sveltia s'affiche.
- Un enregistrement de test dans le CMS crée bien un commit, et le site se reconstruit (~1 min).

### Le point qui coince : la vidéo

`assets/video/drone.mp4` pèse **95 Mo**, sur 219 Mo de dépôt. C'est sous la limite GitHub
(100 Mo/fichier), mais tout juste, et ça pénalise à la fois le clonage et le chargement du site.

**Recommandation : héberger la vidéo sur YouTube ou Vimeo et l'intégrer**, puis la retirer du
dépôt. Tant qu'elle est là, chaque visiteur qui lance la vidéo télécharge 95 Mo.

---

## Sécurité

**Le CMS est chargé en version figée, avec une empreinte d'intégrité.** `admin/index.html` pointe
sur `@sveltia/cms@0.170.8` avec un attribut `integrity="sha384-…"`. C'est délibéré : l'URL sans
version redirige vers la dernière publiée, ce qui reviendrait à mettre à jour tout seul, sans
relecture, un script qui détient les **droits d'écriture sur le dépôt**. L'empreinte fait échouer
le chargement si un seul octet diffère.

**Ce qui protège réellement `/admin/`, c'est GitHub.** La page ne contient aucun mot de passe ni
jeton : c'est du HTML qui charge un script. Sans compte autorisé sur le dépôt, un visiteur ne voit
qu'un écran de connexion inutile. Les vraies serrures :

- **Double authentification** sur les comptes GitHub ayant accès au dépôt.
- N'inviter comme collaborateur que les personnes qui doivent éditer le site.
- Si un jeton personnel est créé, le restreindre à ce seul dépôt, permission
  « Contents : read/write », rien d'autre.

**`robots.txt` n'est pas une protection.** Il demande poliment aux moteurs de ne pas indexer
`/admin/` ; il n'en interdit l'accès à personne.

**Verrou supplémentaire (facultatif).** Sur Cloudflare Pages, **Cloudflare Access** (Zero Trust,
gratuit jusqu'à 50 utilisateurs) peut exiger un code par e-mail avant même d'afficher `/admin/`.
Une ceinture en plus de la bretelle GitHub.

### Mettre à jour le CMS

Les mises à jour sont **manuelles**, conséquence assumée du verrouillage ci-dessus. Changez le
numéro de version dans `admin/index.html`, puis recalculez l'empreinte :

```bash
curl -s https://unpkg.com/@sveltia/cms@LA-VERSION/dist/sveltia-cms.js \
  | openssl dgst -sha384 -binary | openssl base64 -A
```

Collez le résultat dans `integrity="sha384-…"`. Si l'empreinte ne correspond pas, le CMS ne se
charge pas du tout — c'est voulu, et ça se voit immédiatement.

---

## Gérer le contenu au quotidien

| Module | Ce qu'il édite | Fichier |
|---|---|---|
| **Galerie photos** | photos jour / nuit, leur ordre, leurs titres | `gallery.json` |
| **Équipements** | les 6 cartes « À votre disposition » | `equipements.json` |

Dans le module Équipements, l'**icône** se choisit dans une liste déroulante. Ce ne sont pas des
fichiers à téléverser, mais les dessins déjà intégrés au site : le CMS n'enregistre qu'un
identifiant (`i-pot`, `i-spa`…). L'emoji devant chaque libellé sert seulement à s'y retrouver dans
l'admin, il ne part jamais sur le site. Dans le champ **Texte**, **chaque ligne devient une puce**.

> **Redimensionnez les photos avant de les envoyer** (1600 px de large suffisent). Le CMS ne les
> compresse pas : une photo brute de smartphone alourdit la page pour tous les visiteurs.

---

## Contenu du site

Présentation de la maison et des équipements · galerie jour (19) et nuit (19) avec visionneuse
plein écran · vidéo drone · plan d'intérieur (maison + préau) · notices (spa, chauffage/clim,
sèche-serviettes, lave-vaisselle, micro-ondes, aire de jeux) · aux alentours (châteaux, parcs,
caves, marchés, restaurants) · accès (adresse, GPS, carte, itinéraire) · liens Airbnb, Booking,
Festivini, Office de tourisme de Saumur, Google Business · QR code · mentions FLB IMMO (SIRET).

---

*Réalisé à partir du « Descriptif du besoin » et des documents fournis (livret d'accueil, photos,
vidéo, notices, plan).*
