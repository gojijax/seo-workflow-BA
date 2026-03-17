# Setup Client — Wizard d'initialisation

Configure un client dans le monorepo. Ce wizard cree/met a jour le fichier `clients/{client}/PROJECT-CONTEXT.md`.

## Arguments

$ARGUMENTS = slug du client (ex: `mon-client`)

## Instructions

Si le dossier `clients/{slug}/` n'existe pas, le creer avec `briefs/` et `journal/`.

### Etape 1 — Informations de base

Pose ces questions (une par une) :
1. **Nom du projet** (defaut = slug)
2. **Domaine** (sans https://)
3. **Pays / Langue** (defaut: France / fr)
4. **Persona cible**
5. **Ton editorial** (defaut: informatif, accessible, vouvoiement)
6. **Contraintes redactionnelles**

### Etape 2 — Propriete GSC

- Lancer `list_sites` pour afficher les proprietes disponibles
- Demander a l'utilisateur de choisir

### Etape 3 — Google Drive

- Lancer `list-folders` pour afficher les dossiers disponibles
- Ou demander l'ID du dossier parent

### Etape 4 — Contacts (optionnel)

- Redacteur/redactrice : nom + email
- Autres contacts : nom + email + role
- Pour chaque contact, attribuer un label "De"

### Etape 5 — Notion (optionnel)

- BDD Communications : ID du data source (ou proposer de la creer)
- BDD Taches : ID du data source
- Page Projet : URL de la page Notion du projet client

### Etape 6 — Gmail (optionnel)

Si Gmail MCP configure, construire la query de scan automatiquement depuis le domaine + contacts.

### Etape 7 — Ecriture PROJECT-CONTEXT.md

Ecrire `clients/{slug}/PROJECT-CONTEXT.md` avec toutes les infos collectees.

Format :

```markdown
# Contexte projet — {nom_projet}

> Source de verite du client. Ce fichier est dans git.

## Config client
- **Projet** : {nom}
- **Domaine** : {domaine}
- **Pays / Langue** : {pays} / {langue}
- **Persona** : {persona}
- **Ton** : {ton}
- **Contraintes** : {contraintes}

## Propriete GSC
- **ID** : {gsc_property}

## Google Drive
- **Dossier parent** : {drive_folder_id}

## Contacts
{contacts formates}
- **Query Gmail** : {query}

## Notion
{config notion}

## Regles apprises
(vide au depart — se remplit au fil des sessions)

## Etat projet
(vide au depart — se remplit au fil des sessions)
```

### Etape 8 — Verification

Afficher un resume et demander confirmation.
Si confirme : "Setup termine. Lance `/brief {slug} <url>` pour demarrer."
