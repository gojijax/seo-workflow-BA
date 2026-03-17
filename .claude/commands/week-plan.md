# Week Plan — planification hebdomadaire (lundi)

Lance le lundi matin. Passe en revue chaque client, fixe les objectifs de la semaine.

## Instructions

### 1. Chargement du contexte

Lis `MEMORY.md` (repertoire auto-memory) pour recuperer :
- Liste des clients actifs
- IDs Notion (Taches, Communications, Projets)
- Contacts

### 2. Collecte des donnees

#### a) Snapshot de la semaine precedente (si existe)
- Lis `snapshots/week-{YYYY-Wnn-1}.md` (semaine precedente)
- Si le fichier n'existe pas : noter "Premiere semaine trackee, pas de comparatif"

#### b) Taches en cours par client
- Fetch toutes les BDD Taches Notion
- Groupe par client/projet
- Pour chaque : titre, statut, date prevue, retard eventuel

#### c) Communications non traitees
- Fetch BDD Communications : actions requises = oui, non traitees

#### d) Bilan GSC rapide par client (si configure)
- Pour chaque client : `get_performance_overview` sur les 7 derniers jours
- Comparer avec les 7 jours precedents (clicks, impressions, CTR, position moyenne)

### 3. Presentation par bloc

```markdown
## Plan de la semaine — Semaine {numero} ({date lundi} → {date vendredi})

---

### Client : {nom_client_1}

**Semaine derniere** (si snapshot dispo) :
- Prevu : {nb} taches → Fait : {nb} ({%})
- GSC : {clicks} clicks ({+/-X%} vs S-1)

**Taches en cours / en retard** :
| Tache | Statut | Date | Retard |
|-------|--------|------|--------|
| ... | ... | ... | ... |

**Communications en attente** : {nb}

---

### Client : {nom_client_2}
{meme structure}

---

### Admin / Divers
- Taches non rattachees a un client
```

### 4. Phase interactive

**Question 1 — Par client** :
> "Pour {client}, voila ce qui est prevu. Tu veux ajouter, retirer ou reprioriser quelque chose ?"

Repeter pour chaque client.

**Question 2 — Objectif cle** :
> "Un objectif cle pour toi cette semaine ? (le truc qui fait que vendredi tu es satisfait)"

### 5. Consolidation du plan

Apres validation, affiche le plan consolide :

```markdown
## Objectifs semaine {numero}

### Objectif cle : {objectif}

### Par client
- **{client 1}** : {2-3 objectifs cles}
- **{client 2}** : {2-3 objectifs cles}

### Taches critiques (non reportables)
1. {tache}
2. {tache}
```

### 6. Sauvegarde du plan

- Cree le fichier `snapshots/week-{YYYY-Wnn}-plan.md` avec le plan valide
- Si le dossier `snapshots/` n'existe pas, le creer
- Log dans le journal : "Week plan S{nn} : {nb} objectifs, {nb} taches planifiees"

Termine par : "Bonne semaine. Lance `/morning` chaque matin pour le suivi quotidien."
