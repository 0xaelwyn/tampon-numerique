# Tampon Numérique

Créez et apposez des tampons d'entreprise sur vos PDF — directement dans le navigateur, sans installation.

> **Vos PDF ne quittent jamais votre machine.** Tout le traitement (lecture, tamponnage, export) s'effectue localement dans votre navigateur. Aucun fichier n'est envoyé à un serveur applicatif — le site est statique et ne sert que les assets de l'interface.

![github pages](https://img.shields.io/badge/hosted-GitHub%20Pages-blue?style=flat-square) ![licence](https://img.shields.io/badge/licence-MIT-blue?style=flat-square)

---

## Utilisation

**En ligne** — ouvrez la page GitHub Pages du projet dans votre navigateur. Aucune installation requise.

**En local** — clonez le dépôt et démarrez un serveur HTTP dans le dossier du projet :

```bash
git clone https://github.com/<votre-compte>/<votre-repo>.git
cd <votre-repo>
python3 -m http.server 8080
```

Puis ouvrez **http://localhost:8080** dans votre navigateur.

> L'ouverture directe depuis le système de fichiers (`file://`) ne fonctionne pas car le chargeur utilise `fetch()` pour récupérer les assets.

L'application (~2,6 Mo d'assets) embarque l'intégralité des librairies et des polices compressées. Une fois la page chargée, aucun appel réseau n'est effectué vers un CDN ou un service tiers — le traitement reste 100 % local.

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
- **6 coloris** — rouge, bleu, vert, violet, noir, or + **couleur personnalisée** via sélecteur natif
- **6 polices**

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

## Ce que cet outil fait — et ne fait pas

**Ce qu'il fait**
Il appose un tampon **visuel** sur un PDF. Le document reste numérique, non dégradé, et peut être envoyé directement — sans impression, sans tampon physique, sans scan.

C'est l'équivalent numérique du geste : imprimer → tamponner → scanner. En mieux, parce que le PDF ne perd pas en qualité.

**Ce qu'il ne fait pas**
Il n'applique aucune signature cryptographique. Le fichier produit **ne garantit pas l'intégrité du document** (une modification ultérieure ne serait pas détectable) et **n'a pas de valeur légale au sens du règlement eIDAS** (cachet électronique qualifié).

**Pour quel usage**
Documents internes, bons de livraison, validation de devis, courrier administratif courant — partout où le tampon physique suffisait jusqu'ici. Pour des documents engageant juridiquement la personne morale (facturation électronique B2B, marchés publics, actes réglementaires), un prestataire de services de confiance qualifié (PSCo agréé ANSSI) est nécessaire.

---



Articles R123-237 à R123-238 du Code de commerce — mentions recommandées sur un tampon d'entreprise :

| Zone | Contenu |
|------|---------|
| Texte principal | Dénomination sociale ou statut |
| Adresse ligne 1 | Adresse du siège social |
| Adresse ligne 2 | Code postal et ville |
| TVA | N° de TVA intracommunautaire |

---

## Pour aller plus loin

Les détails d'implémentation (bundler, composants, algorithme de layout, pipeline SVG → PDF, coordonnées) sont documentés dans [ARCHITECTURE.md](ARCHITECTURE.md).

---

## Licence

[MIT](LICENSE)
