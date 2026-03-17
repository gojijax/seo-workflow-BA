# Publish — Publication de contenu SEO sur WordPress

Publie du contenu SEO (title, meta description, contenu editorial) sur WordPress via l'API REST native. Gere la conversion Markdown → HTML, la sauvegarde du contenu existant, et la verification post-publication.

## Arguments

$ARGUMENTS = [client] [slug-page] [--dry-run] [--meta-only] [--content-only]
- Ex: `/publish mon-client maisons-traditionnelles`
- Ex: `/publish mon-client homepage --meta-only`
- `--dry-run` : simule la publication sans ecrire sur WordPress
- `--meta-only` : publie uniquement title + meta description
- `--content-only` : publie uniquement le contenu editorial
- Si le client n'est pas precise, demander quel client.

## Pre-requis

1. **Lis `clients/{client}/PROJECT-CONTEXT.md`** — config WP API (URL base, credentials, mapping pages/IDs)
2. **Verifie que le contenu redige existe** dans `clients/{client}/briefs/{slug}/contenu-redige.md`
3. **Verifie que l'API est accessible** : `GET /wp-json/wp/v2/pages/{id}`

## Config API WordPress

Le client doit avoir dans son PROJECT-CONTEXT.md :

```markdown
## WordPress API
- **Base URL** : https://www.mon-site.fr
- **Auth** : Application Password (user: admin, password: xxxx xxxx xxxx xxxx)
- **Plugin SEO** : yoast | seopress | rankmath
- **Pages** :

| Slug brief | WP ID | Slug WP | Type |
|------------|-------|---------|------|
| homepage | 2 | page-daccueil | page |
| page-service | 42 | nos-services | page |
```

**Authentification** : WordPress Application Passwords (natif depuis WP 5.6).
- Creer dans : WP Admin → Utilisateurs → Profil → Mots de passe d'application
- Utiliser en header : `Authorization: Basic base64(user:password)`

## Workflow complet — 7 etapes

### ETAPE 1 — Identification et lecture

1. Identifier le brief slug et le WP ID depuis le mapping dans PROJECT-CONTEXT.md
2. Lire le `contenu-redige.md` local dans `clients/{client}/briefs/{slug}/`
3. Si le fichier local n'existe pas, chercher sur Google Drive (dossier Drive du client)
4. Extraire du contenu redige :
   - **Title** retenu (section "Title" → choix retenu ou option recommandee)
   - **Meta description** (souvent "Non redigee volontairement" → ne PAS publier de meta desc)
   - **Contenu editorial** (section "Contenu" → tout le texte structure H2/H3/p/table/ul)
   - **Maillage interne** (section "Maillage interne" → tableau ancre/URL/raison)

### ETAPE 2 — Backup + audit maillage existant

**CETTE ETAPE EST OBLIGATOIRE. NE JAMAIS LA SAUTER.**

1. Lire la page actuelle via l'API REST :
   ```bash
   curl -s -H "Authorization: Basic {BASE64_CREDS}" \
     "https://www.mon-site.fr/wp-json/wp/v2/pages/{ID}?_fields=id,title,content,meta" | python3 -m json.tool
   ```
2. Sauvegarder le contenu existant dans `/tmp/backup-{ID}-content.txt`
3. **Auditer le maillage interne existant** :
   - Extraire TOUS les liens `<a href=...>` du champ `content.rendered` actuel
   - Lister les ancres et URLs
   - **REGLE CARDINALE : ne JAMAIS retirer un lien interne existant**
   - En rajouter : OUI. En retirer : JAMAIS.
4. Noter le type de page pour adapter la strategie de conversion HTML

### ETAPE 3 — Conversion Markdown → HTML

Le contenu redige est en Markdown. WordPress attend du HTML.

#### Regles de conversion

```markdown
### H2 : Mon titre                    → <h2>Mon titre</h2>
#### H3 : Mon sous-titre              → <h3>Mon sous-titre</h3>

Paragraphe de texte normal.           → <p>Paragraphe de texte normal.</p>

- Item 1                              → <ul><li>Item 1</li><li>Item 2</li></ul>
- Item 2

| Col1 | Col2 |                       → <table><thead><tr><th>Col1</th><th>Col2</th></tr></thead>
|------|------|                           <tbody><tr><td>Val1</td><td>Val2</td></tr></tbody></table>
| Val1 | Val2 |

[texte du lien](/url/)                → <a href="https://www.{domain}/url/">texte du lien</a>
**texte en gras**                     → <strong>texte en gras</strong>
*texte en italique*                   → <em>texte en italique</em>
```

#### Ce qu'il faut NE PAS inclure dans le HTML

- Le H1 (gere par WordPress, pas par le champ content)
- Le TL;DR (element de conception editoriale, pas du HTML brut)
- Les lignes `---` de separation Markdown
- Les marqueurs `[ANGLE UNIQUE]`
- Les instructions/consignes/notes du brief
- Les annotations `> Mot-cle : ...`

#### Ce qu'il FAUT inclure

- Tous les H2 et H3 du contenu (sans le prefixe "H2 :" ou "H3 :")
- Les paragraphes de texte
- Les tableaux (convertis en `<table>` HTML)
- Les listes a puces (`<ul><li>`)
- Les liens internes (ancre + URL complete avec domaine)
- Les encadres "Bon a savoir" (en `<p><strong>Bon a savoir</strong> — ...</p>`)
- La FAQ si presente (en H3 questions + paragraphe reponse)

### ETAPE 4 — Publication SEO meta (title + meta desc)

La methode depend du plugin SEO installe (configure dans PROJECT-CONTEXT.md) :

**Yoast SEO** :
```bash
curl -s -X POST \
  -H "Authorization: Basic {BASE64_CREDS}" \
  -H "Content-Type: application/json" \
  --data-binary @- \
  "https://www.mon-site.fr/wp-json/wp/v2/pages/{ID}" <<'ENDJSON'
{
  "meta": {
    "_yoast_wpseo_title": "Le title retenu",
    "_yoast_wpseo_metadesc": "La meta description"
  }
}
ENDJSON
```

**SEOPress** :
```bash
curl -s -X POST \
  -H "Authorization: Basic {BASE64_CREDS}" \
  -H "Content-Type: application/json" \
  --data-binary @- \
  "https://www.mon-site.fr/wp-json/wp/v2/pages/{ID}" <<'ENDJSON'
{
  "meta": {
    "_seopress_titles_title": "Le title retenu",
    "_seopress_titles_desc": "La meta description"
  }
}
ENDJSON
```

**Rank Math** :
```bash
curl -s -X POST \
  -H "Authorization: Basic {BASE64_CREDS}" \
  -H "Content-Type: application/json" \
  --data-binary @- \
  "https://www.mon-site.fr/wp-json/wp/v2/pages/{ID}" <<'ENDJSON'
{
  "meta": {
    "rank_math_title": "Le title retenu",
    "rank_math_description": "La meta description"
  }
}
ENDJSON
```

**IMPORTANT — Syntaxe shell pour UTF-8 :**
- TOUJOURS utiliser `--data-binary @-` avec un heredoc `<<'ENDJSON'` (quotes autour de ENDJSON)
- JAMAIS utiliser `-d '...'` inline — les accents et caracteres speciaux cassent l'echappement shell

**Notes :**
- Si le contenu-redige.md dit "Non redigee volontairement" pour la meta desc → ne PAS envoyer de meta description
- Verifier quel plugin SEO est installe avant de publier

### ETAPE 5 — Publication contenu editorial

```bash
curl -s -X POST \
  -H "Authorization: Basic {BASE64_CREDS}" \
  -H "Content-Type: application/json" \
  --data-binary @- \
  "https://www.mon-site.fr/wp-json/wp/v2/pages/{ID}" <<'ENDJSON'
{
  "content": "<h2>Mon H2</h2>\n<p>Mon paragraphe...</p>\n..."
}
ENDJSON
```

**ATTENTION — Echappement JSON :**
- Le HTML doit etre une seule chaine JSON valide
- Les guillemets dans le HTML doivent etre echappes : `\"`
- Les retours a la ligne : `\n`
- Tester la validite JSON avant d'envoyer : `echo '{...}' | python3 -m json.tool`

### ETAPE 6 — Verification post-publication

1. **Relire la page via API** :
   ```bash
   curl -s -H "Authorization: Basic {BASE64_CREDS}" \
     "https://www.mon-site.fr/wp-json/wp/v2/pages/{ID}?_fields=content,meta" | python3 -m json.tool
   ```

2. **Verifier** :
   - Le title SEO correspond au title publie
   - La meta description correspond (ou est vide si non publiee)
   - Le contenu contient le nouveau HTML
   - Les liens internes sont presents dans le HTML (`<a href=...>`)
   - Le nombre de mots est coherent avec le contenu-redige.md

3. **Compter les mots et liens** :
   ```bash
   curl -s -H "Authorization: Basic {BASE64_CREDS}" \
     "https://www.mon-site.fr/wp-json/wp/v2/pages/{ID}" | \
     python3 -c "
   import json,sys,re
   d=json.load(sys.stdin)
   c=d.get('content',{}).get('rendered','')
   text=re.sub(r'<[^>]+>','',c)
   words=len(text.split())
   links=re.findall(r'<a [^>]*href=[\"\\']([^\"\\']+)[\"\\']',c)
   print(f'Mots: {words}')
   print(f'Liens: {len(links)}')
   for l in links: print(f'  -> {l}')
   "
   ```

### ETAPE 7 — Journalisation

Ajouter une entree dans `clients/{client}/journal/JOURNAL.md` :

```
| {YYYY-MM-DD HH:MM} | Contenu editorial {slug} publie sur WordPress (~{N} mots, {liens} liens internes) |
```

Mettre a jour `clients/{client}/PROJECT-CONTEXT.md` :
- Marquer la page comme "publiee" dans la section Etat projet

## Garde-fous

### Maillage interne — REGLE ABSOLUE
- **JAMAIS retirer un lien interne existant du contenu**
- Avant chaque publication, extraire les liens `<a>` du contenu actuel
- Si l'ancien contenu avait des liens HTML, les retrouver dans le nouveau contenu ou les ajouter

### Donnees inventees — REGLE ABSOLUE
- Si le contenu-redige.md contient des `[PLACEHOLDER]`, ne PAS les publier tels quels
- Soit remplacer par des vraies donnees, soit supprimer la phrase contenant le placeholder
- Lister les placeholders restants et demander a l'utilisateur

### Backup — REGLE ABSOLUE
- TOUJOURS sauvegarder le contenu existant AVANT d'ecrire
- Fichier backup : `/tmp/backup-{ID}-content.txt`
- Si la publication echoue, pouvoir restaurer

## Regles cardinales (rappel)

1. **Maillage interne** — NE JAMAIS retirer de liens, uniquement en ajouter
2. **Backup** — TOUJOURS sauvegarder avant d'ecrire
3. **Heredoc** — TOUJOURS utiliser `--data-binary @- <<'ENDJSON'` pour les requetes curl avec accents
4. **Placeholders** — NE PAS publier de `[PLACEHOLDER]` sur le site
5. **Verification** — TOUJOURS relire via API apres publication
6. **Journal** — TOUJOURS logger dans JOURNAL.md + mettre a jour PROJECT-CONTEXT.md
