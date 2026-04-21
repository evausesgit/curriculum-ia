# Les Skills Claude Code — Guide pédagogique pour équipes

> Ce guide explique comment créer, configurer et partager des Skills dans Claude Code — des instructions réutilisables que Claude applique automatiquement aux bonnes tâches, au bon moment.

**Cours original :** [Introduction to Agent Skills](https://anthropic.skilljar.com/introduction-to-agent-skills) — Anthropic Academy (gratuit)  
**Format :** 6 leçons · certificat

---

## Table des matières

1. [Pourquoi les Skills ?](#1-pourquoi-les-skills)
2. [Anatomie d'une Skill](#2-anatomie-dune-skill)
3. [Créer sa première Skill](#3-créer-sa-première-skill)
4. [Configuration avancée](#4-configuration-avancée)
5. [Skills vs. autres outils de personnalisation](#5-skills-vs-autres-outils-de-personnalisation)
6. [Partager les Skills avec son équipe](#6-partager-les-skills-avec-son-équipe)
7. [Diagnostiquer et résoudre les problèmes](#7-diagnostiquer-et-résoudre-les-problèmes)

---

## 1. Pourquoi les Skills ?

### Le problème qu'elles résolvent

Chaque fois que vous expliquez vos conventions de PR à Claude, vous vous répétez. Chaque revue de code, vous re-décrivez comment vous voulez le feedback structuré. Chaque message de commit, vous rappelez le format attendu.

Les Skills permettent d'enseigner ces choses à Claude **une seule fois**. Claude les applique ensuite automatiquement, sans que vous ayez à les réécrire.

### Ce qui distingue les Skills des autres options

Claude Code offre plusieurs façons de personnaliser son comportement. Voici comment les Skills se positionnent :

| Mécanisme | Chargement | Idéal pour |
|-----------|-----------|------------|
| `CLAUDE.md` | À chaque conversation | Standards permanents du projet |
| **Skills** | À la demande, selon le contexte | Expertise spécifique à une tâche |
| Slash commands | Invocation explicite | Actions ponctuelles déclenchées manuellement |
| Hooks | Événements (sauvegarde, commit...) | Automatisations déterministes |

L'avantage clé des Skills : elles ne consomment du contexte que quand elles sont pertinentes. Votre checklist de revue de PR n'a pas besoin d'être en mémoire quand vous déboguez.

---

## 2. Anatomie d'une Skill

### Structure d'un fichier SKILL.md

Une Skill est un dossier contenant un fichier `SKILL.md`. Ce fichier a deux parties séparées par du frontmatter YAML :

```
.claude/skills/nom-de-la-skill/
└── SKILL.md
```

```markdown
---
name: nom-de-la-skill
description: Ce que fait la skill et quand Claude doit l'utiliser.
---

Instructions détaillées pour Claude.
Tout ce qui vient après le frontmatter est le contenu de la skill.
```

### Les champs du frontmatter

| Champ | Requis | Contraintes | Rôle |
|-------|--------|-------------|------|
| `name` | ✅ | Minuscules, chiffres, tirets. Max 64 caractères. | Identifiant de la skill |
| `description` | ✅ | Max 1 024 caractères | Critère de déclenchement — c'est ce que Claude lit pour décider si la skill est pertinente |
| `allowed-tools` | Non | Liste d'outils | Restreint les outils disponibles quand la skill est active |
| `model` | Non | Identifiant de modèle | Spécifie le modèle Claude à utiliser |

### Comment Claude décide d'utiliser une skill

Au démarrage, Claude scanne les dossiers de skills mais ne charge que le `name` et la `description` — pas le contenu. Quand vous envoyez un message, il compare sémantiquement votre requête à l'ensemble des descriptions disponibles. Si une description correspond, Claude vous demande confirmation avant de charger le contenu complet.

> **Conséquence pratique :** une description bien rédigée = une skill qui se déclenche au bon moment. Une description vague = une skill invisible.

### Où vivent les Skills

| Type | Chemin | Portée |
|------|--------|--------|
| Personnelle | `~/.claude/skills/` (macOS/Linux) ou `C:\Users\<user>\.claude\skills\` (Windows) | Tous vos projets |
| Projet | `.claude/skills/` à la racine du repo | Tout le monde sur ce repo |

---

## 3. Créer sa première Skill

### Exemple : une Skill de description de PR

Nous allons créer une Skill personnelle qui standardise la rédaction des descriptions de PR. Personnelle = elle s'appliquera à tous vos projets.

**Étape 1 — Créer le dossier :**
```bash
mkdir -p ~/.claude/skills/pr-description
```

**Étape 2 — Créer le fichier SKILL.md :**
```markdown
---
name: pr-description
description: Rédige des descriptions de pull request. À utiliser quand on crée une PR,
  qu'on résume des changements, ou qu'on demande une description de pull request.
---

Pour rédiger une description de PR :

1. Exécuter `git diff main...HEAD` pour voir tous les changements de la branche
2. Rédiger la description au format suivant :

## Ce qui change
Une phrase expliquant ce que fait cette PR.

## Pourquoi
Contexte court sur la raison de ce changement.

## Détail des modifications
- Points listés des changements spécifiques
- Regrouper les changements liés
- Mentionner les fichiers supprimés ou renommés
```

**Étape 3 — Tester :**
Redémarrez votre session Claude Code (les Skills sont chargées au démarrage), puis dites : *"écris une description de PR pour mes changements"*. Claude confirmera qu'il utilise votre Skill et produira le même format à chaque fois.

### Priorité en cas de conflit de noms

Quand deux Skills portent le même nom, l'ordre de priorité est :

```
Enterprise > Personnel > Projet > Plugins
```

Si votre organisation a une Skill `code-review` en Enterprise, votre Skill personnelle `code-review` sera ignorée. Solution : utilisez des noms plus spécifiques (`frontend-code-review`, `api-code-review`).

### Mettre à jour ou supprimer une Skill

- **Mettre à jour** : modifier le fichier `SKILL.md`
- **Supprimer** : effacer le dossier entier
- **Important** : redémarrer Claude Code après toute modification

---

## 4. Configuration avancée

### Restreindre les outils avec `allowed-tools`

Par défaut, une Skill active n'impose aucune restriction sur les outils disponibles. Vous pouvez changer ça pour créer des Skills en lecture seule — utiles pour l'exploration de codebase, l'onboarding, ou les workflows sensibles :

```yaml
---
name: exploration-codebase
description: Aide à comprendre l'architecture et le fonctionnement du système.
  À utiliser pour l'onboarding ou les questions d'architecture.
allowed-tools: Read, Grep, Glob, Bash
model: sonnet
---
```

Quand cette Skill est active, Claude ne peut pas modifier de fichiers — même si vous le lui demandez explicitement.

### Progressive disclosure : structurer les grandes Skills

Le contenu d'une Skill est chargé entièrement en contexte quand elle est activée. Pour les Skills complexes, charger 2 000 lignes d'un coup serait inefficace.

La solution : garder l'essentiel dans `SKILL.md` et répartir les détails dans des fichiers annexes, chargés uniquement si nécessaire.

**Structure recommandée :**
```
.claude/skills/ma-skill/
├── SKILL.md              # Instructions essentielles (< 500 lignes)
├── references/           # Documentation détaillée
│   └── guide-archi.md
├── scripts/              # Scripts exécutables
│   └── validate.sh
└── assets/               # Templates, données
```

**Dans SKILL.md, indiquez explicitement quand charger les fichiers annexes :**
```markdown
Pour les questions d'architecture système, lire `references/guide-archi.md`.
Pour toute autre demande, utiliser les instructions ci-dessous uniquement.
```

Ainsi, Claude ne charge `guide-archi.md` que si la question le justifie.

> **Règle pratique :** si votre `SKILL.md` dépasse 500 lignes, c'est le signe qu'une partie du contenu devrait être dans un fichier de référence.

### Exécuter des scripts sans consommer de contexte

Les scripts référencés dans une Skill peuvent s'exécuter sans que leur contenu soit chargé en contexte — seul le résultat de l'exécution est visible par Claude. Dites à Claude d'**exécuter** le script, pas de le **lire**.

Particulièrement utile pour :
- La validation d'environnement
- Les transformations de données qui doivent être reproductibles
- Les opérations mieux codées que générées

---

## 5. Skills vs. autres outils de personnalisation

### Guide de décision rapide

```
La règle doit-elle s'appliquer à CHAQUE conversation ?
├── Oui → CLAUDE.md
└── Non → La règle s'applique à des tâches spécifiques ?
    ├── Oui → Skill
    └── La règle doit-elle s'appliquer à chaque fois qu'un événement se produit ?
        ├── Oui → Hook
        └── Avez-vous besoin d'un contexte d'exécution isolé ?
            ├── Oui → Sous-agent
            └── Avez-vous besoin d'un outil externe ?
                └── Oui → MCP
```

### Comparaison détaillée

**CLAUDE.md vs Skills**

Le `CLAUDE.md` est toujours en mémoire — chaque conversation commence avec ce contexte. Une Skill n'est chargée que quand elle est pertinente.

Exemple concret : *"utiliser TypeScript strict mode"* → `CLAUDE.md`. *"Checklist de revue de PR"* → Skill (inutile de la charger quand vous déboguez un problème de perf).

**Skills vs Sous-agents**

Une Skill ajoute des connaissances à votre conversation en cours. Un sous-agent crée un nouveau fil d'exécution isolé avec son propre contexte.

Utilisez un sous-agent quand vous voulez déléguer une tâche complète et n'en recevoir que le résultat. Utilisez une Skill quand vous voulez enrichir la façon dont Claude traite votre demande actuelle.

**Skills vs Hooks**

Les hooks se déclenchent sur des événements (une sauvegarde de fichier, un appel d'outil). Les Skills se déclenchent sur l'intention (ce que vous demandez).

Utilisez un hook pour ce qui doit toujours se produire automatiquement. Utilisez une Skill pour ce que Claude doit savoir faire quand on lui demande.

---

## 6. Partager les Skills avec son équipe

Trois niveaux de partage, selon la portée souhaitée.

### Niveau 1 : via le repo Git

La méthode la plus simple. Les Skills du projet sont dans `.claude/skills/`. En committant ce dossier, tout le monde qui clone le repo les obtient automatiquement — et les mises à jour se propagent via `git pull`.

**Idéal pour :**
- Les standards de code de l'équipe
- Les workflows spécifiques au projet
- Les Skills qui référencent la structure de votre codebase

### Niveau 2 : via des plugins

Les plugins permettent de distribuer des Skills au-delà d'un seul repo, via des marketplaces. Chaque utilisateur installe le plugin dans son Claude Code.

**Idéal pour :** des Skills utiles à la communauté, pas trop liées à un projet spécifique.

### Niveau 3 : via les Enterprise Managed Settings

Les administrateurs peuvent déployer des Skills à toute l'organisation. Ces Skills ont la **priorité absolue** — elles priment sur les Skills personnelles, projet et plugins de même nom.

Les managed settings permettent aussi de restreindre les sources d'installation de plugins :
```json
"strictKnownMarketplaces": [
  { "source": "github", "repo": "mon-org/plugins-approuvés" },
  { "source": "npm", "package": "@mon-org/compliance-plugins" }
]
```

**Idéal pour :** les standards obligatoires, les exigences de sécurité ou compliance.

### Skills et sous-agents : un point d'attention

Les sous-agents ne voient **pas** automatiquement les Skills disponibles — ils démarrent avec un contexte vide. De plus :

- Les agents intégrés (Explorer, Plan, Verify) **ne peuvent pas** accéder aux Skills
- Seuls les sous-agents **personnalisés** peuvent utiliser des Skills, et uniquement si vous les listez explicitement

Pour créer un sous-agent avec des Skills, utilisez `/agents` → "Créer un nouvel agent". Le frontmatter généré ressemble à :

```yaml
---
name: frontend-reviewer
description: "Utiliser pour la revue de code frontend, accessibilité et perf."
tools: Bash, Glob, Grep, Read, WebFetch, WebSearch
model: sonnet
color: blue
skills: accessibility-audit, performance-check
---
```

Ce pattern est particulièrement utile pour des sous-agents spécialisés : un reviewer frontend avec des Skills d'accessibilité, un reviewer backend avec des Skills de sécurité.

---

## 7. Diagnostiquer et résoudre les problèmes

### Outil de premier recours : le validator

Avant tout débogage manuel, lancez le validateur de Skills. Il détecte les problèmes structurels (frontmatter mal formé, chemin incorrect, etc.) avant que vous perdiez du temps à chercher ailleurs.

Installation recommandée via `uv`. Exécutez-le depuis votre dossier de Skills ou depuis n'importe où en passant le chemin en argument.

### Checklist de diagnostic rapide

| Symptôme | Cause probable | Solution |
|----------|---------------|----------|
| La Skill ne se déclenche pas | Description trop vague ou mal alignée avec vos requêtes | Ajouter des formulations que vous utilisez réellement |
| La Skill n'apparaît pas dans la liste | Mauvaise structure de fichier | Vérifier que `SKILL.md` est dans un sous-dossier nommé, pas à la racine |
| Mauvaise Skill utilisée | Descriptions trop similaires entre Skills | Rendre les descriptions plus distinctes et spécifiques |
| Skill personnelle ignorée | Conflit avec une Skill de priorité supérieure | Renommer votre Skill avec un nom plus spécifique |
| Skills de plugin absentes | Cache corrompu | Vider le cache, redémarrer Claude Code, réinstaller le plugin |
| Erreur à l'exécution | Dépendance manquante, permissions, chemin | Vérifier ci-dessous |

### La Skill ne se déclenche pas

Claude utilise la correspondance sémantique — votre requête doit avoir une intention qui recoupe la description. Si ça ne déclenche pas :

1. Comparez votre description avec comment vous formulez réellement vos demandes
2. Ajoutez des formulations alternatives : *"auditer les perfs"*, *"pourquoi c'est lent ?"*, *"optimiser ce code"*
3. Testez plusieurs variantes — si une ne déclenche pas, ajoutez ces mots dans la description

### La Skill n'est pas chargée

Exigences structurelles strictes :
- Le fichier doit s'appeler exactement `SKILL.md` — `SKILL` en majuscules, `.md` en minuscules
- Il doit être dans un sous-dossier nommé (ex: `~/.claude/skills/ma-skill/SKILL.md`), pas directement dans `~/.claude/skills/`

Lancez `claude --debug` pour voir les erreurs de chargement. Cherchez les messages mentionnant le nom de votre Skill.

### Erreurs à l'exécution

Trois causes fréquentes :

- **Dépendances manquantes** — si votre Skill utilise des packages externes, ils doivent être installés. Ajoutez l'info de dépendances dans votre description pour que Claude sache ce qui est nécessaire
- **Permissions de script** — les scripts référencés doivent être exécutables : `chmod +x mon-script.sh`
- **Séparateurs de chemin** — utilisez des slashes (`/`) partout, même sur Windows
