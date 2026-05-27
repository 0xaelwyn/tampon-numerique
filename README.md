# Tampon Numérique

Créez et apposez des tampons d'entreprise sur vos PDF — directement dans le navigateur, sans installation, sans serveur, sans internet.

![offline](https://img.shields.io/badge/offline-100%25-brightgreen?style=flat-square) ![fichier unique](https://img.shields.io/badge/distribution-fichier%20unique-orange?style=flat-square) ![licence](https://img.shields.io/badge/licence-MIT-blue?style=flat-square)

---

## Aperçu

| Personnalisation | Apposition PDF |
|-----------------|----------------|
| Texte, forme, couleur, police, rotation | Glisser-déposer, redimensionnement, nudge clavier |
| Taille de police par zone | Page courante, toutes les pages, plage |
| Export SVG & PNG | Téléchargement immédiat, aucun upload |

---

## Utilisation

Double-cliquez sur **`TamponNumerique.html`** — c'est tout.

Le fichier (~2,6 Mo) embarque l'intégralité des librairies et des polices compressées. Aucune connexion internet n'est nécessaire.

### Workflow

**1 — Personnaliser**
Configurez le texte principal (multiligne), l'adresse, le numéro de TVA et la date dans le panneau gauche. Choisissez la forme, la couleur, la police et l'angle de rotation. L'aperçu se met à jour en temps réel.

**2 — Exporter**
- **SVG** — vectoriel, polices embarquées, prêt pour Word ou Illustrator
- **PNG** — 1 200 × 1 200 px, prêt pour email ou présentation

**3 — Apposer sur un PDF**
Importez votre PDF, glissez le tampon à l'endroit voulu, affinez la position au clavier (←↑→↓, +⇧ pour un pas de 10 px), choisissez les pages à tamponner, téléchargez.

---

## Fonctionnalités

### Tampon

- Texte principal **multiligne** avec statuts prédéfinis (APPROUVÉ, VALIDÉ, CONFIDENTIEL, URGENT…)
- Adresse sur deux lignes, N° TVA intracommunautaire, date
- **Date automatique** — toggle pour afficher la date du jour, mise à jour à minuit
- Taille de police réglable **par zone** en temps réel
- Centrage vertical automatique quel que soit le contenu
- **4 formes** — cercle, rectangle, hexagone, losange
- **6 coloris** — rouge, bleu, vert, violet, noir, or
- **6 polices** (toutes offline)

  | Police | Caractère |
  |--------|-----------|
  | Inter | Sans-serif neutre |
  | Cinzel | Sceau légal |
  | Stardos Stencil | Pochoir / douane |
  | Special Elite | Machine à écrire |
  | Oswald | Condensé moderne |
  | Ultra | Slab expressif |

- **Rotation** −30° à +30°
- **Opacité** réglable

### PDF

- Positionnement libre par glisser-déposer
- Redimensionnement par la poignée coin bas-droit (synchronisé avec le curseur)
- Nudge au clavier ←↑→↓ — +⇧ pour un pas de 10 px
- Rotation appliquée avec pivot centré sur le tampon
- Modes d'application : **page courante**, **toutes les pages**, **plage** (ex. 3–7)
- Navigation multi-pages

### Templates

Sauvegardez une configuration complète (textes, polices, couleur, forme, rotation, opacité) et rechargez-la en un clic. Les templates persistent entre les sessions via `localStorage`.

---

## Mentions légales (France)

Articles R123-237 à R123-238 du Code de commerce — mentions recommandées sur un tampon d'entreprise :

| Zone | Contenu |
|------|---------|
| Texte principal | Dénomination sociale ou statut |
| Adresse ligne 1 | Adresse du siège social |
| Adresse ligne 2 | Code postal et ville |
| TVA | N° de TVA intracommunautaire |

---

## Stack

| | |
|-|-|
| [React 18](https://react.dev) | UI |
| [pdf-lib 1.17](https://pdf-lib.js.org) | Écriture PDF, rotation |
| [PDF.js 3.11](https://mozilla.github.io/pdf.js/) | Rendu PDF |
| [Babel Standalone](https://babeljs.io/docs/babel-standalone) | JSX in-browser |
| Google Fonts (Cinzel, Oswald, Ultra…) | Polices embarquées |

Tout est compressé (gzip) et encodé en base64 dans le fichier HTML. Décompression à la volée via l'API native `DecompressionStream` du navigateur.

---

## Confidentialité

Aucune donnée ne quitte votre machine. Les PDF, les textes et les templates restent sur votre ordinateur — les templates dans le `localStorage` de votre navigateur.

---

## Licence

[MIT](LICENSE)
