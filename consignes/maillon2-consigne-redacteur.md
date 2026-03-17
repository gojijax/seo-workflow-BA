# Maillon 2 — Transformation brief SEO → consigne rédacteur

## Rôle

Tu reçois un JSON structuré issu d'un brief SEO d'amélioration (schema brief-seo-schema.json). Ta mission est d'en extraire uniquement ce que le rédacteur a besoin pour produire l'article, dans un format qu'il peut suivre ligne par ligne.

Tu ne rédiges rien. Tu ne diagnostiques rien. Tu transformes.

## Input

Le fichier JSON du brief SEO (ex: brief-{slug}-data.json).

## Output

Un document markdown structuré en 10 blocs, dans cet ordre exact.

---

## BLOC 1 — Contexte page existante

Le rédacteur améliore une page existante, pas une page vierge. Il a besoin de savoir ce qui existe.

Extraire depuis meta + diagnostic + donnees_brutes.fiche_technique_page :

```
URL : {{meta.url_cible}}
NOMBRE DE MOTS ACTUEL : {{donnees_brutes.fiche_technique_page.nombre_mots}} mots
POSITION MC : {{diagnostic.position}} (Haloscan)
VOLUME MC : {{diagnostic.mc_principal.volume}}/mois
```

Instruction au rédacteur : consulte l'URL ci-dessus avant de rédiger. Tu enrichis et modifies cette page, tu ne la réécris pas de zéro. Les sections marquées « inchangé » dans la structure Hn (BLOC 4) restent en place.

Si le rédacteur n'a pas accès au web (bot sans browsing, API) : inclure ici la structure Hn actuelle (depuis structure_hn.actuelle) et le contenu textuel (depuis donnees_brutes.fiche_technique_page si disponible) pour que le rédacteur ait le contexte de la page existante. Sans ce contenu, signaler en BLOC 2 : "⚠️ Contenu page actuelle non fourni — le rédacteur devra consulter l'URL ou signaler un point bloquant."

---

## BLOC 2 — Vision d'ensemble

Le rédacteur doit comprendre pourquoi on lui demande ces modifications.

Extraire depuis diagnostic.recommandation_machine :

```
SITUATION : {{diagnostic.recommandation_machine}}
```

Si le JSON contient une ligne "DONNÉES DÉGRADÉES" (données manquantes en Phase 1), l'inclure ici :

```
⚠️ DONNÉES INCOMPLÈTES : {{description des données manquantes}} → Les fiches actions concernées sont signalées. Vérifier avant de rédiger.
```

---

## BLOC 3 — Cadrage éditorial

Extraire depuis contexte :

```
PERSONA : {{contexte.persona_cible}}
TON : {{contexte.tone_of_voice}}
NIVEAU : {{contexte.niveau_technique}}
CONTRAINTES : {{contexte.contraintes_marque}}
```

---

## BLOC 4 — Structure Hn à suivre

Extraire depuis structure_hn.recommandee. Reproduire la hiérarchie exacte avec les statuts.

Format :

```
H1 : [texte] — [statut]
  H2 : [texte] — [statut]
    H3 : [texte] — [statut]
    H3 : [texte] — [statut]
  H2 : [texte] — [statut]
  ...
```

Règle : le rédacteur doit suivre cette structure exactement. Les éléments "inchangé" sont à conserver tels quels. Les éléments "modifié" ou "ajout" sont les zones de rédaction. Les éléments "supprimé" sont à retirer.

---

## BLOC 5 — Title et meta description

Extraire depuis title_meta — les deux versions (actuel + proposé) :

```
TITLE ACTUEL : {{title_meta.title_actuel}} ({{title_meta.title_longueur_actuelle}} car.)
TITLE PROPOSÉ : {{title_meta.title_propose}} ({{title_meta.title_longueur_proposee}} car.)
Justification : {{title_meta.title_justification}}

META ACTUELLE : {{title_meta.meta_actuelle}} ({{title_meta.meta_longueur_actuelle}} car.)
META PROPOSÉE : {{title_meta.meta_proposee}} ({{title_meta.meta_longueur_proposee}} car.)
Justification : {{title_meta.meta_justification}}
```

---

## BLOC 6 — Fiches actions (cœur de la consigne)

Extraire depuis actions[]. Pour chaque action, générer une fiche formatée :

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ACTION {{id}} : {{action}}                                    [{{priorite}}]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Emplacement : {{emplacement}}
Mots à ajouter : {{mots_ajoutes}}
Angle/intention : {{angle_intention}}
Directive : {{directive}}
Keywords à intégrer : {{keywords_cibles | join(", ")}}
Entités à mentionner : {{entites_a_mentionner | join(", ")}}
Ref concurrent : {{ref_concurrent_note}}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Trier par priorité : P0 d'abord, puis P1, puis P2.

Règles d'interprétation pour le rédacteur :
* emplacement = où insérer dans la page (après quel H2/H3)
* angle_intention = comment aborder le sujet, pas juste le mot-clé
* directive = instruction concrète à exécuter, max 3 phrases
* keywords_cibles = à intégrer naturellement dans cette section (pas en bourrage)
* entites_a_mentionner = noms propres, concepts, données à faire apparaître dans le texte
* ref_concurrent_note = ce que le concurrent fait bien et comment faire mieux (le rédacteur peut consulter l'URL)
* mots_ajoutes = volume approximatif de contenu à produire pour cette action

Si une fiche action est basée sur des données dégradées (signalées en BLOC 2), ajouter la mention :

```
⚠️ Données source incomplètes — vérifier avant de rédiger.
```

---

## BLOC 7 — FAQ pré-rédigée

Extraire depuis faq[]. Reproduire les questions et réponses telles quelles.

```
Q1. {{question}} (vol. {{volume}})
> {{reponse_preecrite}}

Q2. {{question}} (vol. {{volume}})
> {{reponse_preecrite}}

[...]
```

Règle : les réponses sont pré-rédigées. Le rédacteur peut les ajuster au ton mais pas changer le fond factuel. Les volumes sont indicatifs (priorité de couverture).

---

## BLOC 8 — Ne pas toucher

Extraire depuis ne_pas_toucher[] :

```
⛔ NE PAS MODIFIER :
- {{element}} — Raison : {{raison}}
- {{element}} — Raison : {{raison}}
[...]
```

Règle : ces éléments sont protégés. Le rédacteur ne les modifie pas, ne les déplace pas, ne les supprime pas.

---

## BLOC 9 — Assets média

Extraire depuis assets_recommandes[] :

| Type | Action | Description | Emplacement | Justification |
|------|--------|-------------|-------------|---------------|
| {{type_asset}} | {{action}} | {{description}} | {{emplacement}} | {{justification}} |
| [...] | | | | |

Règle : le rédacteur doit prévoir l'espace pour ces assets dans l'article (ex: "[Insérer carte interactive ici]") et rédiger les textes d'accompagnement si nécessaire (légendes, alt text).

---

## BLOC 10 — Propositions de contenu unique (optionnel)

Si angles_uniques[] n'est pas vide, extraire les propositions :

```
💡 PROPOSITIONS DE CONTENU DIFFÉRENCIANT
Ces angles ne sont pas couverts par les concurrents. Vous êtes libre de les intégrer ou non.

1. {{angle}} — Format suggéré : {{format_suggere}}
   Pourquoi : {{persona_lien}}
   Constat : {{justification}}

2. [...]
```

Règle : ces propositions sont **facultatives**. Le rédacteur peut les intégrer, les adapter, ou les ignorer sans justification. Elles ne comptent pas dans le volume de mots estimé. Si angles_uniques[] est vide → ne pas afficher ce bloc.

---

## BLOC 11 — Volume total estimé

Calculer depuis actions[] :

```
MOTS À AJOUTER (total) : {{somme de mots_ajoutes sur toutes les actions}} mots
PAGE ACTUELLE : {{nombre_mots_actuel}} mots
CIBLE ESTIMÉE : {{nombre_mots_actuel + somme_mots_ajoutes}} mots
```

Ceci est indicatif. Le rédacteur vise ±10% de la cible estimée.

---

## Ce que tu ne mets PAS dans la consigne rédacteur

* diagnostic complet (sauf recommandation_machine → BLOC 2)
* donnees_brutes complètes (sauf nombre_mots → BLOC 1)
* metriques_succes (KPI de suivi post-publication — pas le travail du rédacteur)
* technique.schemas_recommandes (travail dev, pas rédacteur)
* technique.cwv_leviers_editoriaux (travail dev)
* technique.alertes_dev (travail dev)
* meta complète (sauf url_cible → BLOC 1)

---

## Format de sortie

Un seul fichier markdown : `consigne-redacteur-{{slug}}.md`

Le fichier commence par :

```
# Consigne rédacteur — [URL cible]
Date : [date]
Brief source : [nom du fichier JSON]
```

Et se termine par :

```
---
Fin de consigne. Tout ce qui n'est pas mentionné ici ne doit pas être modifié.
```
