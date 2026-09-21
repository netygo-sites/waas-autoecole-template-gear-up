# GearUp — port Astro du template Framer

Reproduction pixel-perfect du template Framer **GearUp (Driving School)** en Astro 7,
relevée sur la démo `happier-ferret-030578.framer.app` (styles calculés à 1440 / 1100 / 390 px).
Un exemplaire de chaque **type de page** est reproduit ; les autres fiches se déclinent à partir
des gabarits (voir « Pages »).

## Lancer

```bash
npm install
npm run dev
```

`npm run build` génère le site statique dans `dist/` (`.html` à plat, `trailingSlash: never`).

## Déploiement

Cloudflare Pages, projet `waas-autoecole-template-gear-up` relié à ce dépôt :
https://waas-autoecole-template-gear-up.pages.dev. Chaque push sur `main` met la production à jour,
toute autre branche obtient un aperçu (`<branche>.waas-autoecole-template-gear-up.pages.dev`).
Réglages : `npm run build`, sortie `dist`, variable `NODE_VERSION=22`.

- URL sans extension et redirections de `/page/` et `/page.html` vers `/page` : natives sur Pages,
  aucun fichier de configuration.
- `src/pages/404.astro` produit `dist/404.html`, que Pages sert avec le statut 404. Sans ce fichier,
  Pages prend le site pour une application monopage et répond l'accueil, en 200, à toute adresse inconnue.
- `functions/videos/[[path]].js` sert les vidéos par morceaux (réponses 206). Pages ignore l'en-tête
  `Range` sur ses fichiers statiques, et Safari (iPhone, iPad, Mac) ne lit pas une vidéo sans ces
  réponses. La fonction ne s'exécute que sur `/videos/*`, et pas en `npm run dev`.

## Points de rupture du template

| Palier | Largeur | Notes |
|---|---|---|
| Desktop | ≥ 1280 px | conteneur 1210 px (marges 20 px), sections 130 px |
| Tablette | 992 – 1279 px | sections 100 px |
| Mobile | ≤ 991 px | sections 80 px, menu burger |

Les presets typographiques ont leurs propres paliers (992–1279 / 768–991 / ≤ 767) — voir `src/styles/global.css`.

## Structure

| Dossier / fichier | Contenu |
|---|---|
| `src/styles/global.css` | tokens (Crème `#F8F7EC`, Encre `#1F1F1F`, Jaune `#FAB700`, Sable `#E6E5DC`/`#E7E5DC`, Graphite `#545454`, Gris `#AFAFAF`), presets `.t-h1`…`.t-small`, utilitaires (`.hpic`, `.roll`, `.hide-m`/`.show-m`, `.appear`) |
| `src/styles/fonts.css` | 119 `@font-face` Inter / Inter Display (fichiers dans `public/fonts/`) |
| `src/styles/icons.css` | les 26 icônes du template en masques SVG (`.ico.i-<nom>`) |
| `src/styles/richtext.css` | texte riche des fiches CMS |
| `src/layouts/Base.astro` | `<head>`, header, footer, apparition au défilement |
| `src/components/` | `Header` (menu « All Pages », burger), `Footer`, `Button` (libellé qui défile), `ArrowButton` (flèche qui glisse), `Pixels` (angle « pixels »), `IconRow`, `Mosaic`, `HeroCard`, `PageHero`, `CourseCard`, `InstructorCard`, `ServiceCard`, `Faq`, `CtaBand`, `PhotoStrip`, `StepByStep`, `TestiCarousel`, `TestiCards`, `ContactForm`, `Extras` |
| `src/components/home/` | sections de l'accueil (`Hero`, `Intro`, `Courses`, `Why`, `Trusted`, `Instructors`, `Steps`, `Testimonials`, `GalleryCta`) |
| `src/components/about/` | sections de la page About (`AboutHero`, `Numbers`, `Gain`, `FaqSection`) |
| `src/data/` | navigation et coordonnées (`site.ts`), `courses`, `services`, `instructors`, `faq`, `pricing`, contenus des fiches (`courseDetails`, `serviceDetails`, `instructorDetails`) |
| `public/images/` | images du template (noms Framer conservés) · `public/videos/` les deux vidéos |
| `functions/videos/` | fonction Cloudflare Pages qui sert les vidéos par morceaux (voir « Déploiement ») |

## Pages

| Route | Type | Gabarit |
|---|---|---|
| `/` | Accueil | `pages/index.astro` |
| `/about-us` | À propos | `pages/about-us.astro` |
| `/courses` · `/services` · `/instructors` | Listes | pages dédiées (sections propres à chacune) |
| `/pricing` · `/faq` · `/contact-us` | Pages simples | pages dédiées |
| `/courses/[slug]` | Fiche cours | exemplaire complet : `defensive-driving` |
| `/services/[slug]` | Fiche service | exemplaire complet : `pick-up-drop-off` |
| `/instructors/[slug]` | Fiche instructeur | exemplaire complet : `jason-miller` |
| toute adresse inconnue | Page 404 | `pages/404.astro` |

Les autres fiches (5 cours, 5 services, 5 instructeurs) sont générées avec le même gabarit :
titre, image et accroche viennent des listes de `src/data/`, le corps de texte reprend celui de
l'exemplaire tant que `courseDetails` / `serviceDetails` / `instructorDetails` n'ont pas leur entrée.
La partie **Actus** (liste, article, catégorie) n'est volontairement pas reproduite.

## À brancher

- **Formulaires** (`ContactForm.astro`) : renseigner `action` (Web3Forms, Formspree…).
- Le texte de la démo est conservé tel quel (y compris l'e-mail volontairement erroné `info@examle.com` de la page Contact et la faute « your are » de la page 404).

## Performances

- **Images** : les originaux Framer (jusqu'à 7008 × 4672 px) sont plafonnés à 2000 px par
  `node scripts/optimize-images.mjs`. Toute nouvelle image ajoutée dans `public/images/` doit
  repasser par ce script : il est idempotent et sauvegarde les originaux dans `.image-originaux/`
  (hors git, hors build). Chargement différé et décodage asynchrone partout, sauf les images du
  premier écran (`loading="eager"`, `fetchpriority="high"` pour l'image principale).
- **Vidéos** : pas de lecture automatique. Une vidéo marquée `data-inview` se charge et se lit à
  l'approche de l'écran, et se met en pause en dehors (script dans `Base.astro`).
  Les MP4 du template (4K, 50 i/s, 25 Mbit/s, avec une piste audio inutile) sont réencodés en
  **1080p**, même cadence et même nombre d'images, sans audio. Originaux dans `.video-originaux/`
  (hors git). Similarité mesurée (SSIM) à la résolution d'affichage Retina : 0,98 et 0,99.
  Recette pour toute nouvelle vidéo de fond (ffmpeg installé via `winget install Gyan.FFmpeg`) :

  ```bash
  ffmpeg -i source.mp4 -map 0:v:0 -an -map_metadata -1 \
    -vf "scale=1920:1080:flags=lanczos,format=yuv420p" \
    -c:v libx264 -preset slow -crf 23 -profile:v high -level:v 4.2 -g 100 \
    -color_primaries bt709 -color_trc bt709 -colorspace bt709 -movflags +faststart sortie.mp4
  ```

  Le niveau 4.2 garantit le décodage matériel sur les téléphones, et `faststart` permet de
  démarrer la lecture avant la fin du téléchargement.
- **Apparitions au défilement** : le masquage initial est posé dans le `<head>` (classe `js-appear`),
  l'animation est une animation CSS (et non une transition, qui écraserait les survols des boutons).
  Le sélecteur des éléments animés existe à deux endroits, `global.css` et `Base.astro`, qui doivent
  rester alignés. Tout est désactivé si le visiteur demande moins d'animations.
