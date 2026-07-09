# Plugin « Rapport Wiki YOD »

Skill pour agents de codage (Claude Code) qui rédige, formalise et publie des rapports de découverte technique conformes au schéma du wiki YOD Ingénierie.

## Installation

Depuis le dépôt du wiki (qui sert de marketplace) :

```
/plugin marketplace add https://github.com/DeeRocking/wiki-yod-ingenieurie
/plugin install rapport-wiki-yod@wiki-yod
```

Pour tester en local pendant le développement :

```bash
claude --plugin-dir ./plugin/rapport-wiki-yod
```

## Usage

Dans une session de l'agent, après une exploration technique :

> « Formalise cette découverte en rapport pour le wiki »

ou explicitement : `/rapport-wiki-yod:rapport-wiki`. La skill produit un `rapport.md` (+ `assets/`) conforme, puis propose la publication.

## Publication

La publication passe par l'API du wiki et demande deux variables d'environnement :

```bash
export WIKI_YOD_URL="https://<domaine-du-wiki>"
export WIKI_YOD_TOKEN="yod_…"   # votre Personal Access Token
```

La skill ne publie jamais sans confirmation explicite.
