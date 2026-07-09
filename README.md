# Plugin agent du wiki YOD Ingénierie

Dépôt d'installation du plugin **`rapport-wiki-yod`** pour Claude Code : sa skill
`rapport-wiki` apprend à un agent à rédiger un rapport de découverte technique
conforme au schéma du wiki YOD Ingénierie (front-matter validé, structure de
chapitres qui alimente la table des matières, assets) et à le publier via l'API.

## Installation

```
/plugin marketplace add https://github.com/DeeRocking/wiki-yod-plugin
/plugin install rapport-wiki-yod@wiki-yod
```

Puis configurer les deux variables d'environnement que la skill utilise :

```bash
export WIKI_YOD_URL=https://…    # l'URL du wiki, fournie par votre admin
export WIKI_YOD_TOKEN=yod_…      # votre Personal Access Token
```

Le token s'obtient auprès d'un administrateur du wiki (page Admin → Inviter un membre).

## Maintenance

Ce dépôt est un **miroir en lecture seule** : son contenu est synchronisé
automatiquement par une GitHub Action depuis le dépôt de développement du wiki
à chaque évolution du plugin. Inutile d'y ouvrir des pull requests.
