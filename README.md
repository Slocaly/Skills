# Skills

Mes skills d'agent (Claude Code, Cursor, Codex…) : ma façon de coder, packagée et installable via [skills.sh](https://skills.sh).

## Installation

```bash
# Lister les skills disponibles
npx skills add <github-user>/<repo> --list

# Installer toutes les skills
npx skills add <github-user>/<repo>

# Installer une skill précise
npx skills add <github-user>/<repo> --skill <nom-de-la-skill>
```

> Remplace `<github-user>/<repo>` par l'emplacement GitHub de ce repo une fois poussé.

## Ajouter une skill

1. Créer `skills/<catégorie>/<nom-de-la-skill>/SKILL.md` : le `name` du frontmatter doit correspondre au nom du dossier (kebab-case), et la `description` doit expliquer *quand* la skill doit se déclencher.
2. Ajouter si besoin des fichiers annexes (`references/`, `scripts/`) à côté du `SKILL.md`.
3. Commit + push : la skill est installable immédiatement.

## Structure

```
skills/
  <catégorie>/              # librairies/, testing/, ...
    <nom-de-la-skill>/
      SKILL.md        # obligatoire (frontmatter name + description)
      references/     # optionnel
      scripts/        # optionnel
```
