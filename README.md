# Étiquettes prix + code-barres — Narbonne Accessoires

Génération et impression d'étiquettes prix avec code-barres, à partir de la base tarif du réseau.
Application autonome (une seule page), fonctionne hors ligne, aucune installation en magasin.

## Utilisation

1. Ouvrir l'URL GitHub Pages du dépôt.
2. Chercher une référence (par code article **ou** par désignation), ou scanner directement
   avec la douchette dans le champ de recherche.
3. Ajuster le nombre d'exemplaires par référence.
4. Choisir le format (broche / tablette / grande) puis **Imprimer**.

Imprimer à **100 %**, sans « ajuster à la page ». Les étiquettes sont dimensionnées en millimètres
réels : broche 23 × 41 mm, tablette 62 × 40 mm, grande 94 × 62 mm. Vérifier à la règle après le
premier tirage — si les cotes ne tombent pas juste, c'est le pilote d'impression qui met à l'échelle.

Sur le format broche, le code-barres est imprimé **à la verticale** : 23 mm de large ne suffisent
pas à loger un symbole lisible à l'horizontale. Le scan fonctionne dans les deux orientations.

## Code-barres

| Cas | Symbologie | Contenu encodé |
|---|---|---|
| Référence avec EAN connu | EAN-13 | le code EAN du fabricant |
| Référence sans EAN | Code 39 | la référence article interne |

Le mode « Toujours Code 39 » force un format unique sur toutes les étiquettes.

### Clé de contrôle Code 39

La clé modulo 43 est **désactivée par défaut**. Elle est facultative dans la norme Code 39, et
une douchette qui n'est pas configurée pour la vérifier la renvoie comme un caractère de données :
`703425` devient `703425L`, `MARIAIR13` devient `MARIAIR137`, et SAGE répond
« Article-Vente inexistant ». Ne cocher l'option que si les douchettes sont explicitement
paramétrées pour vérifier **et retirer** la clé.

⚠️ Le Code 39 n'encode pas un EAN : le scan renvoie la référence interne. Vérifier que le logiciel
de caisse accepte la recherche article par code interne avant de généraliser.

## Données

- **Tarif** : figé dans `index.html` (base du 27/08/2026, 18 654 références).
  À chaque nouveau tarif, régénérer la constante `BASE` et bumper `CACHE` dans `sw.js`.
- **Base EAN** : chargée en direct depuis
  `raw.githubusercontent.com/pierrenarb/inventaire-na/main/ean_base.json` à chaque ouverture.
  Les associations EAN saisies sur le terrain via l'appli inventaire remontent donc
  automatiquement. Une copie embarquée (2 834 correspondances) sert de secours hors ligne.

Si le dépôt `inventaire-na` change d'URL ou passe en privé, adapter l'URL dans la fonction
`refreshEan()` d'`index.html` — sinon l'appli bascule silencieusement sur la base embarquée.

## Fichiers

| Fichier | Rôle |
|---|---|
| `index.html` | Application complète : base tarif, encodeurs EAN-13 / Code 39, interface |
| `manifest.json` | Manifeste PWA (installable sur tablette) |
| `sw.js` | Service worker — cache applicatif. Ne met jamais la base EAN en cache |
| `icon-*.png` | Icônes |
| `.nojekyll` | Désactive Jekyll sur GitHub Pages |

## Déploiement

```bash
git init && git add . && git commit -m "Étiquettes prix avec code-barres"
git branch -M main
git remote add origin https://github.com/<compte>/etiquettes.git
git push -u origin main
```

Puis **Settings → Pages → Source: Deploy from a branch → main / (root)**.

Après chaque mise à jour, bumper `CACHE` dans `sw.js` (`etiquettes-na-v2`, `v3`…), sinon les postes
qui ont déjà ouvert l'appli continueront de servir l'ancienne version depuis leur cache.

## Encodeurs

Les motifs EAN-13 et Code 39 sont générés en JavaScript et ont été vérifiés bit à bit contre la
bibliothèque de référence `python-barcode`. Le Code 39 inclut la clé de contrôle modulo 43,
l'EAN-13 la clé de contrôle standard. Les zones de silence (10 modules) sont intégrées au SVG.
