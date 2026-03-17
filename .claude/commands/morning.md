# Morning — rituel matin interactif

Demarre ta journee avec un etat des lieux complet et un plan d'action valide ensemble.

## Instructions

### 1. Chargement du contexte

Lis `MEMORY.md` (repertoire auto-memory) pour recuperer :
- Config client(s) actif(s)
- IDs Notion (Taches, Communications, Projets)
- Contacts et mappings
- Query Gmail

Si `MEMORY.md` est vide ou absent → propose `/setup-client`.

### 2. Collecte des donnees (en parallele)

Lance ces collectes simultanement :

#### a) Taches Notion
- Fetch la/les BDD Taches (IDs depuis MEMORY.md)
- Trie en 3 categories :
  - **En retard** : taches dont la date est depassee et statut ≠ termine
  - **Aujourd'hui** : taches prevues pour aujourd'hui
  - **Cette semaine** : taches des 7 prochains jours
- Pour chaque tache : titre, projet/client associe, date, statut

#### b) Gmail — messages en attente
- `gmail_search_messages` avec `is:unread` + la query projet depuis MEMORY.md
- `gmail_search_messages` avec `is:starred is:unread` (messages importants)
- Pour chaque email pertinent : expediteur, sujet, date, 1 ligne de resume

#### c) Communications Notion recentes
- Fetch les 10 dernieres entrees de la BDD Communications
- Identifie celles avec "Action requise" = oui et non traitees

#### d) Journal d'hier
- Lis `journal/JOURNAL.md` — les entrees d'hier
- Resume ce qui a ete fait

### 3. Presentation structuree

Affiche le resume dans cet ordre :

```markdown
## Bonjour ! Voici ton point du {date du jour}

### Taches en retard ({nb})
| Client/Projet | Tache | Retard |
|---------------|-------|--------|
| ... | ... | +X jours |

### Aujourd'hui ({nb} taches prevues)
| Client/Projet | Tache | Statut |
|---------------|-------|--------|
| ... | ... | ... |

### Emails en attente ({nb})
| De | Sujet | Depuis |
|----|-------|--------|
| ... | ... | X jours |

### Actions requises non traitees ({nb})
| Communication | De | Date | Resume |
|---------------|-----|------|--------|
| ... | ... | ... | ... |

### Hier en resume
- {resume des entrees journal d'hier}
```

### 4. Phase interactive — Questions

Pose ces questions UNE PAR UNE (attends la reponse avant la suivante) :

**Question 1 — Taches en retard** (si il y en a) :
> "Tu as {nb} taches en retard. Pour chacune, tu veux :
> - la faire aujourd'hui
> - la reporter (a quelle date ?)
> - la supprimer
> Dis-moi pour chaque."

**Question 2 — Priorite du jour** :
> "En dehors de ce qui est deja planifie, tu as quelque chose d'important a ajouter aujourd'hui ?"

**Question 3 — Emails** (si il y en a) :
> "Pour les emails en attente, tu veux que je prepare des reponses ou tu t'en occupes ?"

### 5. Mise a jour Notion

Apres validation des reponses :
- **Taches reportees** → mettre a jour la date dans Notion
- **Nouvelles taches** → creer dans la BDD Taches Notion
- **Taches supprimees** → archiver dans Notion
- Ne PAS modifier les taches que l'utilisateur n'a pas mentionnees

### 6. Plan du jour final

Affiche le plan consolide :

```markdown
## Plan du jour — {date}

### Priorites
1. {tache prioritaire}
2. {tache prioritaire}

### A faire aussi
- {tache secondaire}
- {tache secondaire}

### Emails a traiter
- {email 1}
- {email 2}
```

### 7. Log

Ajoute une entree au journal : "Morning brief : {nb} taches prevues, {nb} emails en attente, {nb} taches reportees."

Termine par : "C'est parti. Tu veux attaquer quoi en premier ?"
