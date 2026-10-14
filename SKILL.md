---
name: qishim-session-manager
description: |
  Gestionnaire de session QISHIM-CORTEX. Scanne les projets, charge le contexte,
  genere un knowledge graph des changements, et sauvegarde les checkpoints.
  Active quand: debut de session, "checkpoint", "session", "resume", "status projets",
  "ou en est-on", "reprendre", "bilan", "save", fin de session, ou automatiquement.
  TOUJOURS actif en arriere-plan pour detecter le besoin de checkpoint.
user-invocable: true
---

# QISHIM Session Manager

Tu es le gestionnaire de session du projet QISHIM-CORTEX. Tu assures la continuite
entre les sessions Claude Max (tokens limites) en chargeant le contexte au debut
et en sauvegardant l'etat a la fin.

## Commandes

```
/session                # Rapport complet: etat de tous les projets
/session resume         # Charger le contexte de la derniere session
/session checkpoint     # Sauvegarder l'etat actuel dans les journaux
/session status         # Dashboard rapide de tous les projets
/session graph          # Generer un knowledge graph des fichiers modifies
/session graph <path>   # Knowledge graph d'un projet specifique
```

## Projets suivis

| Projet | Dossier | Journal |
|--------|---------|---------|
| TabascoCity | `sites/tabascocity/` | `cortex/project-manager/journal/tabascocity/` |
| autres sites | `sites/autres-sites/` | `cortex/project-manager/journal/autres-sites/` |
| RSP Game | `games/Rival-Screen-Punks/` | `cortex/project-manager/journal/rsp/` |
| Sibilization | `games/veracita/` | `cortex/project-manager/journal/sibilization/` |
| Trading | `trading/` | (dans DASHBOARD.md) |
| Prospection | `sales/` | (dans DASHBOARD.md) |

## Deadlines dures

- **TabascoCity v1**: www.tabasco.city → **7 mai 2026**
- **Autres sites v1**: autres sites → **12 mai 2026**
- **Sibilization hex designs**: ASAP
- **RSP 47 fighters**: continu (~5/semaine)

---

## WORKFLOW 1: Resume de session (debut)

Quand invoque avec `/session` ou `/session resume`, ou en debut de conversation:

### Etape 1 — Charger les derniers journaux

```bash
# Trouver le journal le plus recent pour chaque projet
for project in tabascocity autres-sites rsp sibilization; do
  LATEST=$(ls -t cortex/project-manager/journal/$project/*.md 2>/dev/null | head -1)
  if [ -n "$LATEST" ]; then
    echo "=== $project: $LATEST ==="
    cat "$LATEST"
    echo ""
  else
    echo "=== $project: AUCUN JOURNAL ==="
  fi
done
```

### Etape 2 — Charger le dashboard et l'agenda

Lire:
- `cortex/project-manager/DASHBOARD.md`
- `cortex/project-manager/AGENDA_SEMAINE.md`

### Etape 3 — Scanner les changements recents

```bash
# Fichiers modifies dans les dernieres 48h pour chaque projet
echo "=== Changements recents (48h) ==="
for dir in sites/tabascocity sites/autres-sites games/Rival-Screen-Punks games/veracita trading sales; do
  COUNT=$(find "$dir" -type f -newer $(date -d '2 days ago' +%Y%m%d 2>/dev/null || echo "2026-04-11") -not -path '*/node_modules/*' -not -path '*/.next/*' -not -path '*/.venv/*' -not -path '*/__pycache__/*' 2>/dev/null | wc -l)
  if [ "$COUNT" -gt 0 ]; then
    echo "$dir: $COUNT fichiers modifies"
    find "$dir" -type f -newer $(date -d '2 days ago' +%Y%m%d 2>/dev/null || echo "2026-04-11") -not -path '*/node_modules/*' -not -path '*/.next/*' -not -path '*/.venv/*' -not -path '*/__pycache__/*' 2>/dev/null | head -10
  fi
done
```

### Etape 4 — Git status rapide

```bash
git status --short | head -30
git log --oneline -5 2>/dev/null
```

### Etape 5 — Presenter le briefing

Format du briefing a presenter a naim:

```
BRIEFING SESSION — [date]
Derniere session: [date du dernier journal]

DEADLINES:
- TabascoCity: J-XX (7 mai)
- Autres sites: J-XX (12 mai)

ETAT RAPIDE:
- [projet]: [1 ligne d'etat depuis le journal]
- [projet]: [1 ligne d'etat depuis le journal]

PROCHAINES ETAPES (depuis le dernier journal):
1. [etape]
2. [etape]
3. [etape]

CHANGEMENTS DEPUIS DERNIERE SESSION:
- [X fichiers modifies dans Y]

On attaque quoi?
```

---

## WORKFLOW 2: Checkpoint (fin de session)

Quand invoque avec `/session checkpoint`, ou quand:
- Naim dit "checkpoint", "save", "sauvegarde", "a demain", "bonne nuit"
- La conversation est tres longue (80+ echanges)
- Le systeme compresse des messages anterieurs
- Beaucoup de travail a ete fait sans sauvegarde

### Etape 1 — Identifier les projets touches

Analyser la conversation courante. Pour chaque projet touche pendant la session:

### Etape 2 — Ecrire/mettre a jour les journaux

Pour chaque projet touche, ecrire dans:
`cortex/project-manager/journal/{projet}/YYYY-MM-DD.md`

Format OBLIGATOIRE:

```markdown
# [Projet] — Journal du YYYY-MM-DD

## Ce qui a ete fait aujourd'hui
- (liste des actions concretes realisees)

## Etat actuel
- (ou en est le projet MAINTENANT)

## Bugs et problemes rencontres
- (problemes ouverts, erreurs, blocages)

## Decisions prises
- (choix faits pendant la session, avec contexte)

## Prochaines etapes immediates
- (ce qu'il faut faire DES LA PROCHAINE SESSION)

## Code/fichiers modifies
- (liste des fichiers touches avec une ligne de contexte)

## Notes pour la prochaine conversation
- (tout ce que le prochain Claude doit savoir)
```

### Etape 3 — Mettre a jour le dashboard

Lire puis mettre a jour `cortex/project-manager/DASHBOARD.md`:
- Changer les metriques qui ont evolue
- Mettre a jour l'etat de chaque projet touche
- Recalculer les jours restants avant les deadlines

### Etape 4 — Mettre a jour l'agenda

Lire puis mettre a jour `cortex/project-manager/AGENDA_SEMAINE.md`:
- Cocher les taches completees
- Ajouter les nouvelles taches decouvertes
- Ajuster si le planning a derive

### Etape 5 — Mettre a jour les memoires si necessaire

Si des informations structurelles ont change (nouvelle stack, nouveau pattern,
nouvelle decision importante), mettre a jour les memoires dans:
`~/.claude/projects/C--Users-naimd-Documents-QISHIM-CORTEX/memory/`

### Etape 6 — Confirmation

Afficher un resume:
```
CHECKPOINT SAUVEGARDE — [date]

Journaux mis a jour:
- tabascocity/YYYY-MM-DD.md ✓
- autres-sites/YYYY-MM-DD.md ✓

Dashboard: mis a jour ✓
Agenda: mis a jour ✓
Memoires: [aucune modif / X memoires modifiees]

La prochaine session pourra reprendre depuis ces journaux.
```

---

## WORKFLOW 3: Knowledge Graph (graphify)

Quand invoque avec `/session graph` ou `/session graph <path>`:

### Prerequis

```bash
# Verifier que graphify est installe
python3 -c "import graphify" 2>/dev/null || pip install graphifyy -q 2>/dev/null
```

Si graphify n'est pas installable (pas de Python ou pip), utiliser le mode degrade:
scanner manuellement les fichiers et generer un rapport texte.

### Mode complet (graphify disponible)

```bash
mkdir -p graphify-out
python3 -c "
import json
from graphify.detect import detect
from pathlib import Path
result = detect(Path('TARGET_PATH'))
print(json.dumps(result))
" > graphify-out/.graphify_detect.json
```

Puis suivre le pipeline graphify standard:
1. Detect → 2. Extract (AST + semantic) → 3. Build graph → 4. Cluster → 5. Report

Generer le rapport dans `graphify-out/GRAPH_REPORT.md` et presenter un resume.

### Mode degrade (pas de graphify)

Scanner manuellement:

```bash
# Compter les fichiers par type pour chaque projet
for dir in sites/tabascocity sites/autres-sites games/Rival-Screen-Punks games/veracita; do
  echo "=== $dir ==="
  find "$dir" -type f -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.py" -o -name "*.md" \
    | grep -v node_modules | grep -v .next | grep -v __pycache__ \
    | awk -F. '{print $NF}' | sort | uniq -c | sort -rn
done
```

Puis generer un rapport structure:
- Nombre de fichiers par projet et par type
- Fichiers les plus recemment modifies
- Dependances entre projets (shared/prisma, imports croises)
- Points d'attention (fichiers tres gros, configs manquantes)

---

## WORKFLOW 4: Status rapide

Quand invoque avec `/session status`:

Generer un tableau compact:

```
QISHIM-CORTEX — Status [date]

| Projet | Sante | Deadline | J-XX | Dernier changement |
|--------|-------|----------|------|--------------------|
| TabascoCity | 6/10 | 7 mai | J-25 | [date] |
| autres sites | 7/10 | 12 mai | J-30 | [date] |
| RSP | 5/10 | continu | - | [date] |
| Sibilization | 6/10 | ASAP | - | [date] |
| Trading | 1/10 | S1 | - | [date] |
```

---

## Detection automatique de besoin de checkpoint

SANS ETRE INVOQUE EXPLICITEMENT, ce skill doit detecter quand un checkpoint est necessaire.

Signaux a surveiller:
1. La conversation depasse 60-80 echanges substantiels
2. Plusieurs fichiers ont ete modifies dans differents projets
3. Des decisions importantes ont ete prises
4. Le contexte est dense et risque d'etre perdu

Quand detecte, PROPOSER proactivement:
"On a fait pas mal de travail. Je propose un checkpoint pour sauvegarder le contexte. /session checkpoint ?"

**REGLE D'OR: Sauvegarder > Finir une tache.**
Si le contexte est presque plein, PRIORISER le checkpoint sur tout le reste.
