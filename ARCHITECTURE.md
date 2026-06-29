# Architecture — Tampon Numérique

### Vue d'ensemble

L'application (~56 Ko de JSX + ~2,5 Mo de librairies/polices compressées) est découpée en fichiers statiques servis via GitHub Pages. Pas de bundler au runtime, pas de `node_modules`, pas de build step — tout est résolu à l'ouverture de la page dans le navigateur.

```
index.html                 — shell HTML (thumbnail de chargement + appel au loader)
css/
└── loader.css             — styles de l'écran de chargement
js/
└── loader.js              — loader : fetch des assets, décompression, injection du DOM
data/
├── manifest.json          — index des assets (UUID → {data base64, compressed, mime})
├── template.json          — HTML source + JSX de l'app (JSON-encoded string)
└── ext_resources.json     — mapping ressources externes (ex. worker PDF.js)
.github/
└── workflows/pages.yml    — déploiement automatique sur GitHub Pages à chaque push main
```

### Mécanisme de bundling

Au chargement, le loader (`js/loader.js`) :

1. Récupère en parallèle `data/manifest.json`, `data/template.json` et `data/ext_resources.json` via `fetch()`
2. Parse le manifest JSON (28 assets : librairies JS, fichiers woff2, worker PDF.js)
3. Décode chaque asset en base64 → `Uint8Array`
4. Décompresse via l'API native `DecompressionStream('gzip')` (pas de dépendance externe)
5. Crée un `blob:` URL par asset et les injecte dans le template HTML (remplacement des UUID par leurs URL)
6. Remplace le DOM par le template hydraté via `DOMParser` + `replaceWith`

Les polices Google Fonts (Cinzel, Stardos Stencil, Special Elite, Oswald, Ultra, Inter) sont embarquées en woff2 — leur CSS `@font-face` pointe vers des `blob:` URLs, donc elles s'affichent sans réseau.

### Stack

| Librairie | Version | Rôle |
|-----------|---------|------|
| [React](https://react.dev) | 18 | UI réactive |
| [Babel Standalone](https://babeljs.io/docs/babel-standalone) | latest | Transpilation JSX in-browser |
| [pdf-lib](https://pdf-lib.js.org) | 1.17.1 | Écriture PDF, rotation, embed image |
| [PDF.js](https://mozilla.github.io/pdf.js/) | 3.11 | Rendu PDF dans `<canvas>` |

### Composants React

```
App                        — composant racine, tout l'état
├── StampSVG               — rendu SVG pur du tampon (stateless)
├── SectionLabel           — en-tête de section avec slot droit optionnel
├── FieldGroup             — label + hint + children + slider optionnel
├── SizeSlider             — curseur A——A avec accentColor
├── TextInput / TextArea   — inputs stylés avec focus ring
├── Chip                   — bouton toggle (actif/inactif)
├── PrimaryButton          — CTA avec press animation
└── SecondaryButton        — action secondaire avec hover
```

### État de l'application

**Tampon**

| State | Type | Rôle |
|-------|------|------|
| `shape` | string | Forme (`circle` \| `rectangle` \| `hexagon` \| `diamond`) |
| `ci` | number | Index couleur (0–5) |
| `fi` | number | Index police (0–5) |
| `main` | string | Texte principal (séparateur `\n`) |
| `sub` | string | Adresse (séparateur `\n`) |
| `date` | string | Date affichée |
| `tva` | string | N° TVA |
| `mainFs` `subFs` `dateFs` `tvaFs` | number | Tailles de police (fraction de `size`) |
| `opa` | number | Opacité (0–1) |
| `rot` | number | Rotation en degrés (−30 à +30) |
| `autoDate` | bool | Mise à jour automatique de la date |
| `freeMode` | bool | Mode texte libre (vs. structuré) |
| `freeText` | string | Texte libre (séparateur `\n`, gras inline `**…**`) |
| `freeFs` | number | Taille de police du texte libre |
| `freeAlign` | string | Alignement (`left` \| `center` \| `right` \| `justify`) |

**PDF**

| State | Type | Rôle |
|-------|------|------|
| `stampPt` | number | Taille du tampon en points PDF |
| `applyMode` | string | `current` \| `all` \| `range` |
| `rangeFrom` `rangeTo` | number | Plage de pages |
| `pos` | `{x, y}` | Position en pixels canvas (origine haut-gauche) |
| `cvSz` | `{w, h}` | Dimensions du canvas de prévisualisation |
| `pdfDims` | `{w, h}` | Dimensions de la page PDF en points |
| `drag` `resizing` | bool | Mode interaction en cours |

### Algorithme de layout SVG

Le tampon centre **l'ensemble du contenu** (date + lignes décoratives + texte principal + adresse + TVA) dans la forme, quelle que soit la combinaison de zones actives et de tailles de police.

La constante `CAP = 0.72` représente le rapport hauteur des capitales / corps de la police (cap-height ratio), utilisé pour convertir une taille de police en hauteur visuelle réelle.

```
totalH  =  mainBlockH
        +  (hasDate ? dFs + gDateLine + gLineMain : 0)
        +  (hasSub  ? gMainSub + subBlockH : 0)
        +  (hasTva  ? gap + vFs × CAP : 0)

curY = cy − totalH / 2    ← point de départ : tout est centré sur cy
```

Chaque zone est ensuite placée séquentiellement en descendant `curY`, les baselines calculées à partir du haut des capitales (`curY + fs × CAP`).

**Mode texte libre.** La fonction pure `freeGeom(cfg, size)` estime la largeur du bloc (ligne la plus longue × avance moyenne par glyphe, le gras comptant un peu plus large) et sa hauteur, puis en déduit les demi-extents `rx`/`ry` de la forme et un `viewBox` ajusté au contenu — la forme **épouse le texte** au lieu d'un carré fixe. `stampAspect(cfg)` renvoie le ratio largeur/hauteur correspondant (1 en mode structuré), utilisé pour ne pas déformer le tampon à l'écran et sur le PDF. Le gras inline est parsé par `parseBold()` en segments `<tspan>` ; l'alignement `justify` étire chaque ligne (sauf la dernière) via `textLength`/`lengthAdjust`.

### Pipeline SVG → PDF

```
StampSVG (React)
  └─ sérialisé en string XML
       └─ injecté dans <style> avec les @font-face CSS embarqués   ← getEmbeddedFontCSS()
            └─ rendu dans <canvas> via HTMLImageElement (blob URL)
                 └─ exporté en PNG (canvas.toBlob)
                      └─ embarqué dans le PDF via pdf-lib (embedPng)
                           └─ drawImage() avec rotation pivot-centré
```

L'empreinte du tampon vaut `stampW = stampPt` en largeur et `stampH = stampPt / stampAspect(cfg)` en hauteur (égales en mode structuré). La rotation est appliquée autour du **centre** du tampon. pdf-lib pivote nativement autour du coin bas-gauche de l'image, donc la position est recalculée :

```js
const cx = pdfX + stampW/2,  cy = pdfY + stampH/2;
const xNew = cx − (stampW/2)×cos + (stampH/2)×sin;
const yNew = cy − (stampW/2)×sin − (stampH/2)×cos;
```

### Conversion de coordonnées

Le canvas de prévisualisation (PDF.js) utilise l'origine **haut-gauche**, pdf-lib utilise l'origine **bas-gauche** (convention PDF). La conversion :

```js
pdfX =  (pos.x / cvSz.w) × pdfPageWidth
pdfY =  pdfPageHeight − ((pos.y / cvSz.h) × pdfPageHeight) − stampPt
```

### Design system

L'interface utilise des variables CSS déclarées sur `:root` :

```
--bg           fond général
--surface      fond des cartes
--surface-2    fond des inputs
--ink          texte principal
--ink-soft     texte secondaire
--ink-faint    texte tertiaire / placeholders
--line         séparateurs légers
--line-strong  bordures des inputs
--radius       border-radius des cartes (14px)
--radius-sm    border-radius des éléments (10px)
--shadow       ombre des cartes
--shadow-lift  ombre au survol
```

### Export SVG avec polices embarquées

`getEmbeddedFontCSS()` lit les règles `@font-face` déjà chargées dans les stylesheets du document (dont les `url()` pointent vers des `blob:` URLs après hydratation) et les injecte dans le `<svg>` exporté via un élément `<style>`. Le fichier SVG est ainsi **auto-suffisant** — les polices s'affichent correctement même sans l'application.

### Lien partageable

`collectCfg()` sérialise la configuration du tampon, `encodeCfg()` l'encode en **base64url** (`btoa(unescape(encodeURIComponent(json)))`, unicode-safe) et la place dans le hash (`#cfg=…`). Au chargement, un `useEffect` lit le hash, appelle `applyCfg()` pour restaurer l'état, puis nettoie l'URL via `history.replaceState`. Tout reste côté navigateur — aucune donnée n'est transmise.