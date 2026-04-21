# Claude Code 101 — Guide pédagogique pour équipes

> Ce guide enseigne à une équipe de développement comment utiliser Claude Code efficacement au quotidien. Il couvre les concepts fondamentaux, les workflows pratiques et les outils de personnalisation.

**Cours original :** [Claude Code 101](https://anthropic.skilljar.com/claude-code-101) — Anthropic Academy (gratuit)  
**Format :** 13 leçons · 1h30 de vidéo · quiz · certificat

---

## Table des matières

1. [Ce qu'est vraiment Claude Code](#1-ce-quest-vraiment-claude-code)
2. [Comment ça marche sous le capot](#2-comment-ça-marche-sous-le-capot)
3. [Les slash commands](#3-les-slash-commands)
4. [Installation selon votre environnement](#4-installation-selon-votre-environnement)
5. [Écrire son premier prompt](#5-écrire-son-premier-prompt)
6. [Le workflow Explore → Plan → Code → Commit](#6-le-workflow-explore--plan--code--commit)
7. [Gérer le contexte sur une longue session](#7-gérer-le-contexte-sur-une-longue-session)
8. [Revue de code avec Claude](#8-revue-de-code-avec-claude)
9. [Personnaliser Claude Code pour votre équipe](#9-personnaliser-claude-code-pour-votre-équipe)

---

## 1. Ce qu'est vraiment Claude Code

### La différence avec un assistant chat

La plupart des développeurs ont déjà utilisé Claude ou ChatGPT en copiant-collant du code. Claude Code fonctionne différemment : il a un **accès direct** à votre filesystem, votre terminal et votre dépôt. Il n'a pas besoin qu'on lui montre le code — il va le lire lui-même, l'éditer, et exécuter des commandes.

C'est ce qui en fait un **agent**, pas un simple assistant.

### Qu'est-ce qu'un agent ?

Un agent IA est un programme qui perçoit son environnement et prend des actions pour atteindre un objectif. Concrètement, Claude Code peut :

- **Lire et comprendre votre codebase** — explorer les fichiers, tracer un bug, comprendre une architecture
- **Modifier des fichiers** — refactoriser une fonction et mettre à jour tous les fichiers qui l'utilisent
- **Exécuter des commandes** — lancer les tests, installer des dépendances, lire les logs, et s'en servir pour décider quoi faire ensuite
- **Chercher sur le web** — consulter de la documentation ou des références d'API à la volée

### Ce qu'il faut garder en tête

Trois points importants pour bien travailler avec Claude Code :

**La fenêtre de contexte, c'est sa mémoire de travail.** Il peut tenir beaucoup d'informations en tête, mais pas tout votre projet à la fois. Il explore donc stratégiquement votre codebase pour trouver ce dont il a besoin.

**Il demande votre accord avant d'agir.** Par défaut, Claude Code attend votre validation avant de modifier un fichier ou d'exécuter une commande. Vous gardez le contrôle.

**Il peut se tromper.** Comme tout outil, il peut mal interpréter une demande ou introduire un bug. Rester dans la boucle permet d'attraper ces erreurs tôt.

---

## 2. Comment ça marche sous le capot

Comprendre le fonctionnement interne de Claude Code permet de l'utiliser bien plus efficacement.

### La boucle agentique

À chaque fois que vous envoyez un message, Claude Code suit cette boucle :

```
Votre prompt
    ↓
Collecte du contexte (lecture de fichiers, recherche web...)
    ↓
Action (modification de fichier, exécution de commande...)
    ↓
Vérification (les résultats répondent-ils à la demande ?)
    ↓
Si oui → attend votre prochain message
Si non → recommence la boucle
```

Vous pouvez interrompre ou orienter Claude à n'importe quelle étape de cette boucle.

### La fenêtre de contexte

Tout ce que Claude "voit" lors d'une session — vos messages, les fichiers lus, les résultats de commandes — occupe de la place dans sa fenêtre de contexte. Quand elle se remplit, Claude compacte automatiquement les échanges passés pour libérer de l'espace. Il peut perdre certains détails dans l'opération, d'où l'importance de bien gérer le contexte (voir section 6).

### Les outils (tools)

Les outils sont ce qui transforme Claude d'un générateur de texte en agent capable d'agir. Chaque capacité — lire un fichier, exécuter une commande, faire une recherche web — est un outil que Claude choisit d'utiliser selon la situation.

### Les modes de permission

Claude Code propose plusieurs niveaux d'autonomie :

| Mode | Comportement |
|------|-------------|
| **Approbation (défaut)** | Demande confirmation avant chaque modification de fichier ou commande |
| **Auto-accept** | Modifie les fichiers sans demander, mais demande toujours pour les commandes |
| **Plan Mode** | N'utilise que des outils en lecture seule pour élaborer un plan avant toute action |

> **Conseil équipe :** commencer en mode Approbation le temps de se familiariser avec ce que Claude fait, puis passer en Auto-accept pour les tâches de routine.

---

## 3. Les slash commands

### Ce que c'est

Les slash commands sont des instructions intégrées que vous tapez directement dans le prompt de Claude Code. Elles ne sont pas envoyées au modèle comme un message de conversation — elles déclenchent des comportements spécifiques dans l'application Claude Code elle-même.

Pensez-y comme au panneau de contrôle de votre session : elles permettent de gérer le contexte, inspecter l'état, configurer des outils et lancer des workflows — sans quitter le terminal.

### Comment les invoquer

Tapez `/` suivi du nom de la commande et appuyez sur Entrée. La plupart des commandes ne nécessitent pas d'arguments.

```
/compact
```

Certaines commandes ouvrent un menu interactif (comme `/agents` ou `/hooks`). D'autres s'exécutent immédiatement (comme `/clear`).

Vous pouvez aussi taper `/` seul et appuyer sur Tab pour voir les commandes disponibles.

### Référence des commandes

#### Session & contexte

| Commande | Ce qu'elle fait | Quand l'utiliser |
|----------|----------------|-----------------|
| `/context` | Affiche l'utilisation de la fenêtre de contexte : taille totale, répartition par catégorie, graphique visuel | Avant une longue tâche, ou quand les réponses semblent moins précises |
| `/compact` | Résume et compresse le contexte actuel pour libérer de la place | En cours de feature, quand on approche de la limite |
| `/clear` | Efface tout le contexte — nouvelle session vierge | Pour démarrer une nouvelle feature sans lien avec la précédente |
| `/cost` | Affiche le coût en tokens de la session en cours | Pour suivre l'utilisation ou déboguer un contexte anormalement grand |

#### Initialisation du projet

| Commande | Ce qu'elle fait | Quand l'utiliser |
|----------|----------------|-----------------|
| `/init` | Analyse votre projet et génère un fichier `CLAUDE.md` | Première mise en place de Claude Code sur un projet |

#### Outils & intégrations

| Commande | Ce qu'elle fait | Quand l'utiliser |
|----------|----------------|-----------------|
| `/mcp` | Liste les serveurs MCP connectés, leur statut, et permet de les activer/désactiver | Gérer les intégrations externes pendant une session |
| `/agents` | Ouvre le gestionnaire de sous-agents : lister, créer, modifier | Configurer ou revoir les agents de votre projet |
| `/hooks` | Ouvre le configurateur de hooks | Ajouter ou revoir des guardrails déterministes |

#### Git & code

| Commande | Ce qu'elle fait | Quand l'utiliser |
|----------|----------------|-----------------|
| `/commit-push-pr` | Commit, push et création de PR — en une seule étape | En fin de feature, quand vous êtes prêt à shipper |

#### Aide & diagnostic

| Commande | Ce qu'elle fait | Quand l'utiliser |
|----------|----------------|-----------------|
| `/help` | Liste toutes les commandes disponibles | Quand vous avez oublié une commande |
| `/doctor` | Diagnostique votre installation Claude Code (config, auth, connectivité) | Quelque chose ne fonctionne pas et vous ne savez pas pourquoi |

### Flags CLI (en dehors de la session)

Certains comportements se contrôlent via des flags au lancement de Claude Code, pas depuis l'intérieur d'une session :

| Flag | Ce qu'il fait |
|------|--------------|
| `claude --from-pr <numéro>` | Reprend la session liée à une PR (pour répondre à des commentaires de review, corriger un build cassé...) |
| `claude --debug` | Lance une session avec un output de debug verbeux — utile pour diagnostiquer les problèmes de chargement de Skills ou MCP |
| `claude mcp add <nom>` | Ajoute un nouveau serveur MCP à votre configuration |

---

## 4. Installation selon votre environnement

Claude Code s'installe dans plusieurs environnements. Choisissez celui qui correspond à votre workflow.

### Terminal (macOS / Linux / WSL)

```bash
# Installation via curl (supporte les mises à jour auto)
curl -fsSL https://claude.ai/install.sh | sh

# Lancer Claude Code dans votre projet
cd mon-projet
claude
```

Au premier lancement : choix du thème, connexion avec votre compte Claude (Pro, Max, ou Enterprise) ou une clé API.

> Claude Code a accès au répertoire depuis lequel vous le lancez et tous ses sous-dossiers.

### Windows

Plusieurs options disponibles via PowerShell, CMD, ou `winget`. Note : `winget` et Homebrew ne supportent pas les mises à jour automatiques — privilégiez `curl` si possible.

### VS Code

1. Panneau Extensions → chercher **"Claude Code"** (extension Anthropic, badge de vérification bleu)
2. Installer → redémarrer VS Code si nécessaire
3. `Ctrl/Cmd + Shift + P` → "Claude Code Open in New Tab"

L'expérience est quasi-identique au terminal. Un paramètre permet de désactiver l'UI et d'utiliser directement le terminal intégré.

### JetBrains

Installer le plugin **Claude Code** depuis le JetBrains Marketplace. Après redémarrage, le logo Claude apparaît dans la barre latérale et ouvre un panneau terminal intégré à l'éditeur.

### Claude Desktop

Ouvrir Claude Desktop → activer le toggle **"Code"** en haut. Permet de travailler dans un dossier spécifique avec des permissions configurables, et de laisser Claude tourner en arrière-plan pendant que vous faites autre chose.

### Web (claude.ai/code)

Accès via `claude.ai/code` ou le menu latéral de Claude.ai. Même expérience que Desktop, mais limité aux dépôts GitHub.

### Quel environnement choisir ?

| Besoin | Environnement recommandé |
|--------|--------------------------|
| Avoir les dernières fonctionnalités | Terminal |
| Rester dans son éditeur | VS Code / JetBrains |
| Laisser Claude tourner en fond | Desktop |
| Travailler sur un repo GitHub à distance | Web |

---

## 5. Écrire son premier prompt

### Choisir son mode d'interaction

`Shift + Tab` fait défiler les modes disponibles :

- **Approbation** → Claude demande avant chaque action
- **Auto-accept** → les modifications de fichiers sont automatiques
- **Plan Mode** → Claude analyse et planifie avant de toucher quoi que ce soit

### Plan Mode : indispensable pour les tâches complexes

Le Plan Mode est particulièrement adapté aux implémentations multi-étapes. En mode Plan, Claude utilise uniquement des outils de lecture pour explorer votre code et vous poser des questions de clarification. Il soumet ensuite un plan détaillé avant de commencer — et c'est votre validation qui déclenche l'exécution.

**Exemple de prompt en Plan Mode :**
```
Mon app a besoin d'un système de thème sombre.
Crée un toggle dans le header qui bascule entre light et dark mode
pour l'ensemble de l'application, en cohérence avec ma palette existante.
```

Claude va explorer vos fichiers CSS/Tailwind, vous poser des questions si nécessaire, puis vous soumettre un plan step-by-step avant d'écrire la moindre ligne.

### Bonnes pratiques de prompt

- **Soyez précis** — un prompt vague force Claude à explorer davantage ce qui consomme plus de contexte
- **Définissez un critère de succès** — "les tests doivent passer" ou "le rendu doit correspondre au design"
- **Mentionnez les contraintes** — framework, version, conventions du projet

---

## 6. Le workflow Explore → Plan → Code → Commit

C'est le workflow central à adopter en équipe. Il évite le principal piège : demander du code directement sans avoir établi de contexte, ce qui entraîne des corrections coûteuses en cours de route.

### Explore

Avant de toucher du code, donnez à Claude le contexte dont il a besoin. En Plan Mode, Claude explore votre codebase en lecture seule. Vous pouvez aussi déclencher l'exploration explicitement pour obtenir un résumé d'architecture sans intention immédiate de modifier quoi que ce soit.

**Exemple :**
```
Je dois ajouter la conversion WebP dans notre pipeline d'upload d'images.
Dis-moi où dans le pipeline ça devrait s'insérer, quelles dépendances
seraient nécessaires, et comment tu envisages l'implémentation.
```

### Plan

Claude analyse les fichiers pertinents, fait des recherches si besoin, et vous soumet un plan d'action. C'est le **meilleur moment pour corriger le tir** — avant que du code soit écrit. Posez des questions, demandez des révisions sur certains points. Un plan bien validé mène à une implémentation propre.

### Code

Une fois le plan validé, Claude travaille la liste d'étapes. Quelques conseils pour cette phase :

- **Critère de succès explicite** — indiquez à Claude ce que "correct" signifie (tests verts, comportement attendu...)
- **Suite de tests** — donnez-la à Claude comme source de vérité continue. Il peut aussi en écrire une pour vous
- **Outils adaptés** — pour une UI web, l'extension Claude in Chrome permet à Claude de contrôler un onglet et tester visuellement
- **Mémorisez les corrections** — si Claude répète les mêmes erreurs, demandez-lui de sauvegarder la solution dans `CLAUDE.md`

### Commit

Avant de pousser votre code :

1. **Lancez une revue par sous-agent** (voir section 7) — regard neuf, sans le biais de la session de codage
2. **Demandez à Claude de générer le message de commit** dans le style de votre équipe

---

## 7. Gérer le contexte sur une longue session

La fenêtre de contexte est une ressource limitée. La gérer activement maintient la qualité des réponses tout au long d'une session.

### Les trois commandes essentielles

| Commande | Effet | Quand l'utiliser |
|----------|-------|-----------------|
| `/compact` | Compacte l'historique en gardant un résumé | En cours de feature, quand vous approchez de la limite |
| `/clear` | Repart de zéro, contexte vide | Entre deux features distinctes |
| `/context` | Vue d'ensemble : taille, catégories, répartition | Pour diagnostiquer ce qui consomme du contexte |

> **Règle pratique :** `/compact` en cours de feature, `/clear` entre features. Ce que vous voulez que Claude retienne d'une session à l'autre → dans votre `CLAUDE.md`.

### Stratégies pour économiser du contexte

**Soyez précis dans vos prompts.** Un prompt vague oblige Claude à explorer plus largement pour deviner votre intention — ce qui consomme bien plus de contexte qu'un prompt détaillé.

**Désactivez les serveurs MCP inutilisés.** Chaque serveur MCP connecté charge ses définitions d'outils dans le contexte, même si vous ne l'utilisez pas. Vérifiez avec `/mcp` et désactivez ce qui n'est pas pertinent pour la tâche en cours.

**Utilisez des sous-agents pour les recherches.** Quand vous avez besoin d'explorer votre codebase pour une question précise, un sous-agent fait le travail dans son propre contexte et retourne uniquement la réponse — sans polluer votre contexte principal.

---

## 8. Revue de code avec Claude

### Revue par sous-agent : regard neuf garanti

Avant de pousser une PR, déléguez la revue à un sous-agent. L'avantage clé : le sous-agent démarre avec un contexte vierge, sans le biais accumulé pendant la session de codage.

**Configuration recommandée pour un sous-agent de revue :**
- Outils en lecture seule uniquement — un reviewer signale des problèmes, il ne corrige pas
- Versionner la configuration dans le repo pour que toute l'équipe utilise le même reviewer

### Commit, push et PR en une commande

La skill `/commit-push-pr` enchaîne commit, push et création de PR en une seule étape. Si vous avez un serveur MCP Slack configuré avec vos canaux dans `CLAUDE.md`, elle peut aussi poster automatiquement le lien de la PR.

### Reprendre le travail sur une PR existante

Quand Claude crée une PR via `gh pr create`, la session est liée à cette PR. Pour reprendre le travail plus tard :

```bash
claude --from-pr <NUMÉRO_DE_PR>
```

Utile pour répondre à des commentaires de review ou corriger un build cassé.

---

## 9. Personnaliser Claude Code pour votre équipe

### 8.1 Le fichier CLAUDE.md

Le `CLAUDE.md` est un fichier Markdown à la racine de votre projet. Claude le lit automatiquement à chaque démarrage de session. Pensez-y comme à un **document d'onboarding pour Claude** — il évite de redécouvrir les mêmes choses à chaque session.

#### Structure minimale recommandée

```markdown
# Projet

Application Next.js 15 avec App Router, Tailwind CSS et Drizzle ORM.

# Commandes
- Dev : `pnpm dev`
- Tests : `pnpm test`
- Lint : `pnpm lint`

# Conventions
- Indentation : 2 espaces
- Exports nommés plutôt que défaut
- Routes API dans `app/api/`
- Préférer les Server Actions aux routes API quand possible
```

#### Hiérarchie des fichiers mémoire

| Fichier | Emplacement | Portée |
|---------|-------------|--------|
| `CLAUDE.md` projet | Racine du repo | Toute l'équipe (à versionner dans git) |
| `CLAUDE.md` personnel | Dossier config utilisateur | Vous uniquement, tous vos projets |

#### Conseils pratiques

**Laissez émerger le contenu.** Commencez sans `CLAUDE.md` et observez où vous corrigez souvent Claude. Ces corrections méritent d'être mémorisées. Utilisez `/init` pour que Claude génère une première version.

**Référencez votre documentation existante :**
```markdown
## Architecture
Pour plus de contexte : @docs/architecture.md
```

**Mémorisez les corrections récurrentes.** Si vous vous surprenez à redire la même chose, demandez à Claude : *"Sauvegarde cette règle dans le CLAUDE.md du projet."*

---

### 8.2 Les sous-agents

Un sous-agent est une instance de Claude qui tourne en parallèle avec son propre contexte isolé. Il reçoit une tâche, la traite, et retourne uniquement son résultat — sans polluer votre contexte principal.

#### Pourquoi c'est utile

Quand vous demandez à Claude d'explorer votre codebase pour localiser quelque chose, Claude lit des dizaines de fichiers — tout ça consomme du contexte. Si vous n'avez besoin que de la réponse finale, tout ce travail exploratoire est du bruit. Un sous-agent fait exactement la même exploration dans sa propre bulle, et vous retourne juste la conclusion.

#### Créer un sous-agent

```
/agents → "Créer un nouvel agent"
```

Vous définissez : le périmètre (personnel ou projet), l'objectif, les outils accessibles, et quand Claude doit l'appeler automatiquement.

Les sous-agents sont des fichiers Markdown avec frontmatter YAML — versionnables dans votre repo.

---

### 8.3 MCP (Model Context Protocol)

MCP est un standard ouvert qui connecte Claude Code à des outils et sources de données externes — Linear, Slack, GitHub, documentation de dépendances, bases de données...

#### Ajouter un serveur MCP

```bash
claude mcp add <nom-du-serveur>
```

Deux types : **HTTP** (services distants) et **Stdio** (processus locaux).

#### Portée des serveurs

| Portée | Fichier de config | Partage |
|--------|------------------|---------|
| Local | Settings personnel | Vous uniquement |
| Utilisateur | Settings global | Tous vos projets |
| Projet | `.mcp.json` dans le repo | Toute l'équipe automatiquement |

> Pour partager les MCP avec l'équipe : commitez `.mcp.json` à la racine. Tout le monde obtient les mêmes serveurs au prochain `git pull`.

#### Attention au contexte

Chaque serveur MCP connecté charge ses définitions d'outils dans le contexte, même quand vous ne l'utilisez pas. Si un outil a un équivalent CLI (`gh`, `aws`...), le CLI est plus économe en contexte.

---

### 8.4 Hooks

Les hooks sont des commandes shell qui s'exécutent automatiquement à des moments précis du cycle de vie de Claude Code. Leur force : ils sont **déterministes**. Mettre une règle dans `CLAUDE.md` → Claude la respecte *la plupart du temps*. Un hook → la règle s'applique **à chaque fois, sans exception**.

#### Événements disponibles

| Événement | Déclenchement |
|-----------|--------------|
| `PreToolUse` | Avant l'exécution d'un outil |
| `PostToolUse` | Après l'exécution d'un outil |
| `UserPromptSubmit` | Quand vous soumettez un prompt, avant traitement |
| `Stop` | Quand Claude termine sa réponse |
| `Notification` | Quand Claude envoie une notification |

#### Cas d'usage typiques pour une équipe

- **Auto-formatage** : `PostToolUse` sur `Edit|MultiEdit|Write` → Prettier, `gofmt`, etc.
- **Audit** : logger toutes les commandes exécutées par Claude
- **Guardrails** : bloquer les modifications de config prod, les `rm -rf`, les commits directs sur `main`
- **Notifications** : alerte Slack ou desktop quand une longue tâche se termine

#### Bloquer une action avec PreToolUse

Un hook `PreToolUse` peut bloquer une action avant qu'elle s'exécute. Il reçoit le nom de l'outil et ses paramètres en JSON sur `stdin` :

| Code de sortie | Comportement |
|---------------|-------------|
| `0` | L'action continue normalement |
| `2` | Action bloquée — le message `stderr` est transmis à Claude pour qu'il comprenne pourquoi |
| Autre | Erreur non-bloquante, visible par vous uniquement |

#### Partager les hooks avec l'équipe

Commitez `.claude/settings.json` dans votre repo. Toute l'équipe bénéficie automatiquement des mêmes guardrails. Utilisez `CLAUDE_PROJECT_DIR` pour référencer des scripts du projet dans vos commandes de hook.

> **Principe :** ce qui doit arriver à chaque fois sans exception → un hook. Ce qui est une préférence → le `CLAUDE.md`.

---

## Récapitulatif

| Concept | À retenir |
|---------|-----------|
| **Agent** | Claude Code agit directement sur vos fichiers et commandes — pas de copier-coller |
| **Boucle agentique** | Collecte → Action → Vérification → recommence si nécessaire |
| **Fenêtre de contexte** | Mémoire de travail limitée — gérez-la avec `/compact`, `/clear`, `/context` |
| **Plan Mode** | `Shift+Tab` — explore et planifie avant de coder. Idéal pour les tâches complexes |
| **Workflow principal** | Explore → Plan → Code → Commit |
| **CLAUDE.md** | Mémoire persistante du projet — onboarding script pour Claude |
| **Sous-agents** | Contexte isolé pour déléguer des tâches sans polluer le contexte principal |
| **MCP** | Connexion à des outils externes (Linear, Slack, GitHub...) |
| **Hooks** | Guardrails déterministes — s'exécutent toujours, sans exception |
