---
name: rapport-wiki
description: Rédige, formalise et publie un rapport de découverte technique pour le wiki YOD Ingénierie — front-matter YAML conforme au schéma (titre, auteur, theme, synopsis), structure en chapitres (##) et sections (###) qui alimente la table des matières du wiki, images dans assets/, publication via POST /api/articles avec Bearer token. Use when the user asks to rédiger un rapport wiki, formaliser une découverte pour le wiki, préparer ou publier un article sur le wiki YOD, ou transformer une exploration technique (session de code, POC, veille) en rapport partageable.
allowed-tools: Read Write Bash(curl *)
---

# Rédiger un rapport pour le wiki YOD Ingénierie

Le wiki n'accepte que du **markdown UTF-8 (≤ 1 Mo)** dont le front-matter respecte un schéma figé. Ce front-matter hydrate automatiquement le catalogue, le rendu et les statistiques ; la structure de titres hydrate les panneaux de table des matières. Un rapport mal structuré s'affiche sans table des matières — la structuration n'est pas optionnelle.

## 1. Collecter la matière

Rassembler ce que la personne a exploré : la conversation en cours, le code produit, les notes, les erreurs rencontrées et les conclusions. Un rapport raconte **une seule découverte** (techno, framework, architecture, concept) ; si la session en contient plusieurs, proposer plusieurs rapports.

## 2. Front-matter — schéma exact

Le fichier DOIT commencer par ce bloc (validé côté serveur, rejet sinon) :

```yaml
---
titre: <3 à 200 caractères>
auteur: <2 à 100 caractères — prénom et nom de l'auteur humain>
theme: <2 à 60 caractères — thème court et réutilisable, ex. « Frontend », « Bases de données », « Outillage »>
synopsis: <10 à 500 caractères — 1 à 3 phrases lues dans le catalogue>
date: 2026-07-08        # optionnelle, format ISO ; défaut = date d'upload
co_auteurs:             # optionnelle, liste YAML, 10 entrées max
  - pseudo-du-membre    # pseudo (recommandé) ou nom exact d'un membre du wiki
---
```

Règles :
- Les quatre premiers champs sont **obligatoires**. Aucun autre champ n'est accepté utilement.
- **Piège YAML fréquent** : toute valeur contenant « : » doit être entourée de guillemets (`synopsis: "Redis : retour d'expérience"`), sinon le serveur rejette le fichier avec « mapping values are not allowed here ».
- `theme` : réutiliser un thème existant du wiki quand c'est pertinent (les thèmes alimentent les filtres et les stats) plutôt que d'en inventer une variante.
- `synopsis` : c'est la vitrine de l'article dans le catalogue — le soigner.
- `co_auteurs` : si l'exploration a été menée à plusieurs, les lister ici (l'auteur
  principal n'y figure pas). Chaque entrée doit désigner un **membre du wiki**, par son
  **pseudo** (recommandé — chaque membre voit le sien sur sa page Mon espace) ou son nom
  exact ; une entrée inconnue est **refusée à l'upload (422)** — demander son pseudo au
  co-auteur plutôt que de deviner l'orthographe du nom. Chaque co-auteur est crédité
  dans les statistiques du wiki et affiché sur l'article ; les doublons sont ignorés.

## 3. Structure du corps — elle alimente la table des matières

Le wiki affiche un panneau « Chapitres » (les `##`) et un panneau « Dans ce chapitre » (les `###` du chapitre en cours de lecture). Structurer ainsi :

- **Pas de `#` (h1)** dans le corps : le titre vient du front-matter.
- Ouvrir par un court paragraphe d'introduction **sans titre** (le contexte, le problème de départ).
- Découper en **chapitres `##`** : par exemple « Le problème », « La découverte », « Mise en pratique », « Limites et pièges », « Verdict ».
- Sous-découper les chapitres denses en **sections `###`**. Une section = une idée.
- Blocs de code avec la langue annotée (```python, ```bash…) : le wiki colore la syntaxe.

Un modèle complet est fourni : [modele_rapport.md](modele_rapport.md).

## 4. Images et assets

- Les images vont dans un dossier `assets/` à côté du `.md`, et se référencent **en relatif** : `![légende](assets/mon-schema.png)`.
- Formats acceptés : png, jpg, jpeg, gif, svg, webp — **≤ 5 Mo chacune**.
- Ne jamais référencer d'image par URL externe ni chemin absolu.

## 5. Checklist avant publication

Vérifier chaque point, corriger avant d'aller plus loin :

1. Le fichier commence bien par `---` et le front-matter contient titre, auteur, theme, synopsis dans les bornes de taille.
2. Aucun `#` de niveau 1 dans le corps ; au moins deux chapitres `##`.
3. Toutes les images référencées existent dans `assets/` et sont en chemin relatif.
4. Le rapport se suffit à lui-même : un collègue qui n'a pas vécu l'exploration comprend le contexte, la découverte et le verdict.
5. Encodage UTF-8, taille < 1 Mo.

## 6. Publication (uniquement sur confirmation explicite)

Ne JAMAIS publier sans que l'utilisateur ait validé le rapport final. La publication utilise deux variables d'environnement à demander si absentes :

- `WIKI_YOD_URL` : URL de base du wiki (ex. `https://wiki.exemple.dev`)
- `WIKI_YOD_TOKEN` : Personal Access Token du membre (`yod_…`)

```bash
curl -sS -X POST "$WIKI_YOD_URL/api/articles" \
  -H "Authorization: Bearer $WIKI_YOD_TOKEN" \
  -F "fichier=@rapport.md;type=text/markdown" \
  -F "assets=@assets/schema.png;type=image/png"
```

Répéter `-F "assets=@…"` pour chaque image. Réponses :
- **201** : `{"slug": …, "url": …}` — donner l'URL complète à l'utilisateur.
- **422** : la réponse liste les erreurs de schéma champ par champ — corriger le front-matter et réessayer.
- **401** : token invalide ou révoqué — demander un token valide, ne pas insister.

## 7. Mettre à jour un article existant

Pour une nouvelle version d'un rapport déjà publié (même découverte, contenu enrichi ou
corrigé), ne PAS republier en POST — cela créerait un doublon. Utiliser `PUT` avec le
slug de l'article :

```bash
curl -sS -X PUT "$WIKI_YOD_URL/api/articles/<slug>" \
  -H "Authorization: Bearer $WIKI_YOD_TOKEN" \
  -F "fichier=@rapport.md;type=text/markdown" \
  -F "assets=@assets/nouveau-schema.png;type=image/png"
```

- L'URL de l'article et ses étoiles sont **conservées**, même si le titre change.
- Le nouveau fichier remplace l'ancien ; les assets fournis s'ajoutent ou écrasent
  ceux de même nom. Mêmes règles de validation qu'en publication.
- **403** : l'utilisateur n'est pas l'auteur de cet article — seul le membre qui l'a
  publié peut le mettre à jour.
- La mise à jour exige la même confirmation explicite que la publication.
