# Claude Code — Parcours d'apprentissage complet

> Ce guide agrège l'ensemble de la formation Claude Code en un parcours pédagogique progressif : des fondations jusqu'aux techniques avancées d'automatisation.

**Sources :** [Claude Code 101](https://anthropic.skilljar.com/claude-code-101) · [Introduction to Agent Skills](https://anthropic.skilljar.com/introduction-to-agent-skills) · [Best practices](https://code.claude.com/docs/en/best-practices) — Anthropic Academy & Documentation officielle

---

## Table des matières

**Partie 1 — Comprendre Claude Code**
1. [Ce qu'est vraiment Claude Code](#1-ce-quest-vraiment-claude-code)
2. [La contrainte centrale : la fenêtre de contexte](#2-la-contrainte-centrale--la-fenêtre-de-contexte)

**Partie 2 — Premiers pas**
3. [Installation selon votre environnement](#3-installation-selon-votre-environnement)
4. [Les slash commands](#4-les-slash-commands)
5. [Écrire son premier prompt](#5-écrire-son-premier-prompt)

**Partie 3 — Workflow fondamental**
6. [Le workflow Explore → Plan → Code → Commit](#6-le-workflow-explore--plan--code--commit)
7. [Écrire de bons prompts](#7-écrire-de-bons-prompts)
8. [Gérer le contexte activement](#8-gérer-le-contexte-activement)

**Partie 4 — Vérification et qualité**
9. [Donner à Claude un moyen de vérifier son travail](#9-donner-à-claude-un-moyen-de-vérifier-son-travail)
10. [Revue de code avec Claude](#10-revue-de-code-avec-claude)

**Partie 5 — Personnaliser Claude Code**
11. [CLAUDE.md : la mémoire persistante](#11-claudemd--la-mémoire-persistante)
12. [Les modes de permission](#12-les-modes-de-permission)
13. [Les hooks](#13-les-hooks)
14. [Les sous-agents](#14-les-sous-agents)
15. [MCP — Model Context Protocol](#15-mcp--model-context-protocol)
16. [Les Skills](#16-les-skills)

**Partie 6 — Maîtrise avancée**
17. [Laisser Claude vous interviewer](#17-laisser-claude-vous-interviewer)
18. [Gérer sa session comme un pro](#18-gérer-sa-session-comme-un-pro)
19. [Automatiser et passer à l'échelle](#19-automatiser-et-passer-à-léchelle)
20. [Les patterns d'échec courants](#20-les-patterns-déchec-courants)

[Récapitulatif global](#récapitulatif-global)

---

## Partie 1 — Comprendre Claude Code

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

### Trois choses à garder en tête

**La fenêtre de contexte, c'est sa mémoire de travail.** Il peut tenir beaucoup d'informations en tête, mais pas tout votre projet à la fois. Il explore donc stratégiquement votre codebase pour trouver ce dont il a besoin.

**Il demande votre accord avant d'agir.** Par défaut, Claude Code attend votre validation avant de modifier un fichier ou d'exécuter une commande. Vous gardez le contrôle.

**Il peut se tromper.** Comme tout outil, il peut mal interpréter une demande ou introduire un bug. Rester dans la boucle permet d'attraper ces erreurs tôt.

---

## 2. La contrainte centrale : la fenêtre de contexte

Presque toutes les bonnes pratiques de ce parcours découlent d'une seule contrainte : **la fenêtre de contexte de Claude se remplit vite, et les performances se dégradent à mesure qu'elle se remplit.**

Tout ce que Claude "voit" pendant une session — vos messages, les fichiers lus, les sorties de commandes — occupe de l'espace dans sa fenêtre de contexte. Une seule session de débogage peut consommer des dizaines de milliers de tokens. Quand la fenêtre est pleine, Claude peut oublier des instructions précédentes ou faire davantage d'erreurs.

**La fenêtre de contexte est la ressource la plus importante à gérer.**

Retenez cette idée : chaque technique que vous apprendrez dans ce guide — les prompts précis, les sous-agents, les skills à la demande, les commandes `/compact` et `/clear` — vise, directement ou indirectement, à préserver cet espace pour ce qui compte vraiment.

---

## Partie 2 — Premiers pas

## 3. Installation selon votre environnement

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

Ouvrir Claude Desktop → activer le toggle **"Code"** en haut. Permet de travailler dans un dossier spécifique avec des permissions configurables, et de laisser Claude tourner en arrière-plan.

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

## 4. Les slash commands

Les slash commands sont des instructions intégrées que vous tapez directement dans le prompt de Claude Code. Elles ne sont pas envoyées au modèle comme un message — elles déclenchent des comportements spécifiques dans l'application elle-même.

Pensez-y comme au panneau de contrôle de votre session : gestion du contexte, inspection de l'état, configuration des outils, lancement de workflows — sans quitter le terminal.

Tapez `/` puis Tab pour voir toutes les commandes disponibles.

### Session & contexte

| Commande | Ce qu'elle fait | Quand l'utiliser |
|----------|----------------|-----------------|
| `/context` | Affiche l'utilisation de la fenêtre de contexte : taille, répartition, graphique | Avant une longue tâche, ou quand les réponses semblent moins précises |
| `/compact` | Résume et compresse le contexte actuel pour libérer de la place | En cours de feature, quand vous approchez de la limite |
| `/clear` | Efface tout le contexte — nouvelle session vierge | Pour démarrer une nouvelle tâche sans lien avec la précédente |
| `/cost` | Affiche le coût en tokens de la session | Pour suivre l'utilisation ou déboguer un contexte anormalement grand |
| `/rewind` | Ouvre le menu de rembobinage pour restaurer un état précédent | Après une mauvaise direction ou pour revenir à un checkpoint |
| `/btw` | Pose une question rapide sans l'ajouter au contexte | Vérifier un détail sans gonfler la session |
| `/rename` | Renomme la session en cours | Pour retrouver facilement une session plus tard |

### Initialisation du projet

| Commande | Ce qu'elle fait | Quand l'utiliser |
|----------|----------------|-----------------|
| `/init` | Analyse votre projet et génère un fichier `CLAUDE.md` | Première mise en place de Claude Code sur un projet |

### Outils & intégrations

| Commande | Ce qu'elle fait | Quand l'utiliser |
|----------|----------------|-----------------|
| `/mcp` | Liste les serveurs MCP connectés, leur statut, active/désactive | Gérer les intégrations externes |
| `/agents` | Gestionnaire de sous-agents : lister, créer, modifier | Configurer les agents du projet |
| `/hooks` | Configurateur de hooks | Ajouter ou revoir des guardrails |
| `/permissions` | Gère les listes blanches de commandes autorisées | Réduire les prompts de permission répétitifs |

### Git & code

| Commande | Ce qu'elle fait | Quand l'utiliser |
|----------|----------------|-----------------|
| `/commit-push-pr` | Commit, push et création de PR en une seule étape | En fin de feature |

### Aide & diagnostic

| Commande | Ce qu'elle fait | Quand l'utiliser |
|----------|----------------|-----------------|
| `/help` | Liste toutes les commandes disponibles | Quand vous avez oublié une commande |
| `/doctor` | Diagnostique votre installation (config, auth, connectivité) | Quelque chose ne fonctionne pas |

### Flags CLI (au lancement)

| Flag | Ce qu'il fait |
|------|--------------|
| `claude --continue` | Reprend la session la plus récente |
| `claude --resume` | Choisit dans une liste de sessions sauvegardées |
| `claude --from-pr <numéro>` | Reprend la session liée à une PR |
| `claude -p "prompt"` | Lance Claude en mode non-interactif (CI, scripts) |
| `claude --debug` | Lance une session avec output de debug verbeux |
| `claude mcp add <nom>` | Ajoute un nouveau serveur MCP |

---

## 5. Écrire son premier prompt

### Choisir son mode d'interaction

`Shift + Tab` fait défiler les modes disponibles :

| Mode | Comportement |
|------|-------------|
| **Approbation (défaut)** | Claude demande confirmation avant chaque modification de fichier ou commande |
| **Auto-accept** | Les modifications de fichiers sont automatiques ; demande toujours pour les commandes |
| **Plan Mode** | N'utilise que des outils de lecture pour explorer et planifier avant toute action |

> **Conseil équipe :** commencez en mode Approbation le temps de vous familiariser avec ce que Claude fait, puis passez en Auto-accept pour les tâches de routine.

### Plan Mode : indispensable pour les tâches complexes

Le Plan Mode est particulièrement adapté aux implémentations multi-étapes. Claude explore votre code en lecture seule, vous pose des questions de clarification, puis soumet un plan détaillé — votre validation déclenche l'exécution.

**Exemple de prompt en Plan Mode :**
```
Mon app a besoin d'un système de thème sombre.
Crée un toggle dans le header qui bascule entre light et dark mode
pour l'ensemble de l'application, en cohérence avec ma palette existante.
```

Claude va explorer vos fichiers CSS/Tailwind, vous poser des questions si nécessaire, puis vous soumettre un plan step-by-step avant d'écrire la moindre ligne.

---

## Partie 3 — Workflow fondamental

## 6. Le workflow Explore → Plan → Code → Commit

C'est le workflow central à adopter en équipe. Il évite le principal piège : demander du code directement sans établir de contexte, ce qui entraîne des corrections coûteuses en cours de route.

### Explore

Avant de toucher du code, donnez à Claude le contexte dont il a besoin. En Plan Mode, Claude explore votre codebase en lecture seule. Vous pouvez aussi déclencher l'exploration explicitement pour obtenir un résumé d'architecture.

```
Je dois ajouter la conversion WebP dans notre pipeline d'upload d'images.
Dis-moi où dans le pipeline ça devrait s'insérer, quelles dépendances
seraient nécessaires, et comment tu envisages l'implémentation.
```

### Plan

Claude analyse les fichiers pertinents, fait des recherches si besoin, et vous soumet un plan d'action. C'est le **meilleur moment pour corriger le tir** — avant que du code soit écrit. Posez des questions, demandez des révisions. Un plan bien validé mène à une implémentation propre.

### Code

Une fois le plan validé, Claude travaille la liste d'étapes. Conseils :

- **Critère de succès explicite** — indiquez ce que "correct" signifie (tests verts, comportement attendu)
- **Suite de tests** — donnez-la à Claude comme source de vérité continue ; il peut aussi en écrire une
- **Outils adaptés** — pour une UI web, l'extension Claude in Chrome permet à Claude de contrôler un onglet et tester visuellement
- **Mémorisez les corrections** — si Claude répète les mêmes erreurs, demandez-lui de sauvegarder la règle dans `CLAUDE.md`

### Commit

Avant de pousser votre code :

1. **Lancez une revue par sous-agent** — regard neuf, sans le biais de la session de codage (voir section 10)
2. **Demandez à Claude de générer le message de commit** dans le style de votre équipe

---

## 7. Écrire de bons prompts

### Patterns de prompting

Le niveau de précision de votre prompt détermine directement la qualité du résultat et la quantité de contexte consommée.

| Stratégie | Prompt faible | Prompt fort |
|-----------|---------------|-------------|
| **Cadrer la tâche** | *"ajoute des tests pour foo.py"* | *"écris un test pour foo.py couvrant le cas où l'utilisateur est déconnecté. évite les mocks."* |
| **Pointer vers les sources** | *"pourquoi ExecutionFactory a une API bizarre ?"* | *"regarde l'historique git de ExecutionFactory et résume comment son API en est arrivée là"* |
| **Référencer les patterns** | *"ajoute un widget calendrier"* | *"regarde HotDogWidget.php pour comprendre notre pattern de widget, puis implémente un widget calendrier qui permet de sélectionner un mois et paginer par année. pas de nouvelles bibliothèques."* |
| **Décrire le symptôme** | *"corrige le bug de login"* | *"les utilisateurs rapportent que le login échoue après expiration de session. vérifie le flux auth dans src/auth/, notamment le token refresh. écris un test qui reproduit le bug, puis corrige-le"* |
| **Critère de vérification** | *"implémente validateEmail"* | *"écris validateEmail. cas de test : user@example.com → true, invalid → false, user@.com → false. lance les tests après."* |

### Fournir du contexte riche

- **`@filename`** — référencer un fichier directement ; Claude le lit avant de répondre
- **Coller des images** — copier-coller ou glisser des captures d'écran, maquettes, diagrammes
- **Donner des URLs** — coller des liens de documentation ; utiliser `/permissions` pour mettre en liste blanche les domaines fréquents
- **Piper des données** — `cat error.log | claude` envoie le contenu d'un fichier directement
- **Laisser Claude chercher** — lui dire d'utiliser Bash, des outils MCP, ou de lire des fichiers pour récupérer ce dont il a besoin

### Utiliser les outils CLI

Claude est très efficace avec les outils CLI comme `gh` (GitHub), `aws`, `gcloud`, `sentry-cli`. Ces outils sont plus économes en contexte que leurs équivalents MCP et Claude les connaît déjà.

```
Use gh to list open PRs assigned to me, then summarize each one.
```

Pour les outils que Claude ne connaît pas : `Use 'foo-cli --help' to learn it, then use it to do X.`

---

## 8. Gérer le contexte activement

### Les commandes essentielles

| Commande | Effet | Quand l'utiliser |
|----------|-------|-----------------|
| `/compact` | Compacte l'historique en gardant un résumé | En cours de feature, quand vous approchez de la limite |
| `/clear` | Repart de zéro, contexte vide | Entre deux features distinctes |
| `/context` | Vue d'ensemble : taille, catégories, répartition | Pour diagnostiquer ce qui consomme du contexte |

> **Règle pratique :** `/compact` en cours de feature, `/clear` entre features. Ce que vous voulez que Claude retienne d'une session à l'autre → dans votre `CLAUDE.md`.

### Stratégies pour économiser du contexte

**Soyez précis dans vos prompts.** Un prompt vague oblige Claude à explorer plus largement pour deviner votre intention — ce qui consomme bien plus de contexte qu'un prompt détaillé.

**Désactivez les serveurs MCP inutilisés.** Chaque serveur MCP connecté charge ses définitions d'outils dans le contexte, même si vous ne l'utilisez pas. Vérifiez avec `/mcp`.

**Utilisez des sous-agents pour les recherches.** Quand vous avez besoin d'explorer votre codebase pour une question précise, un sous-agent fait le travail dans son propre contexte et retourne uniquement la réponse — sans polluer votre contexte principal.

```
Use subagents to investigate how our authentication system handles token
refresh, and whether we have any existing OAuth utilities I should reuse.
```

---

## Partie 4 — Vérification et qualité

## 9. Donner à Claude un moyen de vérifier son travail

> Donnez à Claude un test qu'il peut exécuter : une suite de tests, un build, une capture d'écran à comparer. C'est la différence entre une session que vous surveillez et une session à laquelle vous pouvez déléguer en confiance.

Claude s'arrête quand le travail *semble* terminé. Sans vérification automatique, vous devenez la boucle de vérification — chaque erreur attend que vous la remarquiez. La vérification peut être n'importe quoi qui retourne un résultat lisible : une suite de tests, un code de sortie de build, un linter, un script comparant la sortie à une fixture.

### Niveaux de vérification

| Niveau | Mécanisme | Mise en place | Pour |
|--------|-----------|---------------|------|
| Dans le prompt | Demander à Claude de lancer le test et d'itérer | Aucune | N'importe quelle tâche |
| Pendant la session | Condition `/goal` — un évaluateur séparé revérifie après chaque tour | Faible | Tâches longues |
| Garde déterministe | Stop hook — bloque la fin du tour jusqu'à ce que le test passe | Moyen | Exécutions sans surveillance |
| Second avis | Subagent de vérification — un modèle frais relit le résultat | Moyen | Code critique |

### Demander des preuves, pas des affirmations

Demandez à Claude de vous montrer la sortie des tests, la commande exécutée, ou une capture d'écran — plutôt que de simplement dire "c'est fait". Relire des preuves est plus rapide que de re-lancer la vérification vous-même, et cela fonctionne pour les sessions que vous n'avez pas surveillées.

---

## 10. Revue de code avec Claude

### Revue par sous-agent : regard neuf garanti

Avant de pousser une PR, déléguez la revue à un sous-agent. L'avantage clé : le sous-agent démarre avec un contexte vierge, sans le biais accumulé pendant la session de codage.

**Configuration recommandée :**
- Outils en lecture seule uniquement — un reviewer signale des problèmes, il ne corrige pas
- Versionner la configuration dans le repo pour que toute l'équipe utilise le même reviewer

### Pattern Writer / Reviewer

Un contexte frais améliore la revue de code — Claude ne sera pas biaisé vers le code qu'il vient d'écrire.

| Session A (Writer) | Session B (Reviewer) |
|--------------------|----------------------|
| `Implement a rate limiter for our API endpoints` | |
| | `Review the rate limiter in @src/middleware/rateLimiter.ts. Look for edge cases, race conditions, and consistency with our existing middleware.` |
| `Here's the review feedback: [résultat]. Address these issues.` | |

Même principe avec les tests : un Claude écrit les tests, un autre écrit le code pour les faire passer.

### Revue adversariale

Avant de considérer une tâche comme terminée, faites relire le résultat par un subagent dans un contexte frais.

```
Use a subagent to review the rate limiter diff against PLAN.md. Check that
every requirement is implemented, the listed edge cases have tests, and
nothing outside the task's scope changed. Report gaps, not style preferences.
```

> Un reviewer invité à trouver des lacunes en trouvera souvent, même quand le travail est solide. Dites-lui de ne signaler que les gaps qui affectent la correction ou les exigences déclarées.

### Commit, push et PR en une commande

La skill `/commit-push-pr` enchaîne commit, push et création de PR en une seule étape. Si vous avez un serveur MCP Slack configuré avec vos canaux dans `CLAUDE.md`, elle peut aussi poster automatiquement le lien de la PR.

### Reprendre le travail sur une PR existante

```bash
claude --from-pr <NUMÉRO_DE_PR>
```

Utile pour répondre à des commentaires de review ou corriger un build cassé.

---

## Partie 5 — Personnaliser Claude Code

## 11. CLAUDE.md : la mémoire persistante

Le `CLAUDE.md` est un fichier Markdown à la racine de votre projet. Claude le lit automatiquement à chaque démarrage de session. Pensez-y comme à un **document d'onboarding pour Claude** — il évite de redécouvrir les mêmes choses à chaque session.

### Structure minimale recommandée

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

### Ce qu'il faut inclure — et ce qu'il faut couper

`CLAUDE.md` est chargé à chaque session. Un fichier trop long pousse Claude à ignorer les règles noyées dans le bruit.

| ✅ À inclure | ❌ À exclure |
|-------------|------------|
| Commandes Bash que Claude ne peut pas deviner | Ce que Claude peut inférer en lisant le code |
| Règles de style différentes des conventions par défaut | Conventions standard du langage |
| Runner de tests et instructions de test | Documentation d'API détaillée (mettre un lien) |
| Convention de nommage des branches et des PR | Informations qui changent fréquemment |
| Décisions d'architecture spécifiques au projet | Longues explications ou tutoriels |
| Particularités de l'environnement (variables d'env requises) | Descriptions fichier par fichier de la codebase |

**Règle d'or :** pour chaque ligne, demandez *"Supprimer cette ligne ferait-il faire des erreurs à Claude ?"* Si non, coupez-la.

### Hiérarchie des fichiers mémoire

| Fichier | Emplacement | Portée |
|---------|-------------|--------|
| `CLAUDE.md` projet | Racine du repo | Toute l'équipe (à versionner dans git) |
| `CLAUDE.local.md` | Racine du repo | Personnel uniquement (dans `.gitignore`) |
| `CLAUDE.md` personnel | `~/.claude/CLAUDE.md` | Vous uniquement, tous vos projets |
| `CLAUDE.md` sous-dossiers | Chargé à la demande lors de la lecture de fichiers dans ce dossier | Contexte local |

### Conseils pratiques

**Laissez émerger le contenu.** Commencez sans `CLAUDE.md` et observez où vous corrigez souvent Claude. Ces corrections méritent d'être mémorisées. Utilisez `/init` pour une première version générée automatiquement.

**Référencez votre documentation existante :**
```markdown
## Architecture
Pour plus de contexte : @docs/architecture.md
```

**Mémorisez les corrections récurrentes.** Si vous vous surprenez à redire la même chose, dites à Claude : *"Sauvegarde cette règle dans le CLAUDE.md du projet."*

**Si une règle est ignorée** malgré sa présence dans `CLAUDE.md`, le fichier est probablement trop long. Convertissez les règles critiques en hooks pour une application déterministe.

---

## 12. Les modes de permission

Par défaut, Claude demande une approbation avant chaque écriture de fichier ou commande. Après la dixième approbation, vous ne relisez plus vraiment — vous cliquez mécaniquement.

| Mode | Comment | Quand l'utiliser |
|------|---------|-----------------|
| **Approbation (défaut)** | Demande avant chaque action | Nouvelles tâches, codebases inconnues |
| **Auto-accept** | Modifie les fichiers sans demander | Tâches de routine bien comprises |
| **Auto mode** | Un classifier bloque les actions risquées ; le reste passe sans prompts | Quand vous faites confiance à la direction générale |
| **Listes blanches** | Autoriser des commandes sûres spécifiques : `npm run lint`, `git commit` | Opérations connues et sûres |
| **Sandboxing** | Isolation OS qui restreint filesystem et réseau | Exécutions sans surveillance avec inputs externes |

Configurez via `/permissions` ou directement dans `.claude/settings.json`.

---

## 13. Les hooks

Les hooks sont des commandes shell qui s'exécutent automatiquement à des moments précis du cycle de vie de Claude Code. Leur force : ils sont **déterministes**. Mettre une règle dans `CLAUDE.md` → Claude la respecte *la plupart du temps*. Un hook → la règle s'applique **à chaque fois, sans exception**.

### Événements disponibles

| Événement | Déclenchement |
|-----------|--------------|
| `PreToolUse` | Avant l'exécution d'un outil |
| `PostToolUse` | Après l'exécution d'un outil |
| `UserPromptSubmit` | Quand vous soumettez un prompt, avant traitement |
| `Stop` | Quand Claude termine sa réponse |
| `Notification` | Quand Claude envoie une notification |

### Cas d'usage typiques

- **Auto-formatage** : `PostToolUse` sur `Edit|MultiEdit|Write` → Prettier, `gofmt`, etc.
- **Audit** : logger toutes les commandes exécutées par Claude
- **Guardrails** : bloquer les modifications de config prod, les `rm -rf`, les commits directs sur `main`
- **Vérification automatique** : Stop hook qui lance les tests et bloque la fin du tour si ils échouent
- **Notifications** : alerte Slack ou desktop quand une longue tâche se termine

### Bloquer une action avec PreToolUse

Un hook `PreToolUse` peut bloquer une action avant qu'elle s'exécute. Il reçoit le nom de l'outil et ses paramètres en JSON sur `stdin` :

| Code de sortie | Comportement |
|---------------|-------------|
| `0` | L'action continue normalement |
| `2` | Action bloquée — le message `stderr` est transmis à Claude pour qu'il comprenne pourquoi |
| Autre | Erreur non-bloquante, visible par vous uniquement |

### Partager les hooks avec l'équipe

Commitez `.claude/settings.json` dans votre repo. Toute l'équipe bénéficie automatiquement des mêmes guardrails. Utilisez `CLAUDE_PROJECT_DIR` pour référencer des scripts du projet dans vos commandes de hook.

> **Principe :** ce qui doit arriver à chaque fois sans exception → un hook. Ce qui est une préférence → le `CLAUDE.md`.

---

## 14. Les sous-agents

Un sous-agent est une instance de Claude qui tourne en parallèle avec son propre contexte isolé. Il reçoit une tâche, la traite, et retourne uniquement son résultat — sans polluer votre contexte principal.

### Pourquoi c'est utile

Quand vous demandez à Claude d'explorer votre codebase pour localiser quelque chose, Claude lit des dizaines de fichiers — tout ça consomme du contexte. Si vous n'avez besoin que de la réponse finale, tout ce travail exploratoire est du bruit. Un sous-agent fait la même exploration dans sa propre bulle et vous retourne juste la conclusion.

### Créer un sous-agent

```
/agents → "Créer un nouvel agent"
```

Vous définissez : le périmètre (personnel ou projet), l'objectif, les outils accessibles, et quand Claude doit l'appeler automatiquement. Les sous-agents sont des fichiers Markdown avec frontmatter YAML — versionnables dans votre repo.

**Exemple — sous-agent de revue de sécurité :**
```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities
tools: Read, Grep, Glob, Bash
model: opus
---
You are a senior security engineer. Review code for:
- Injection vulnerabilities (SQL, XSS, command injection)
- Authentication and authorization flaws
- Secrets or credentials in code

Provide specific line references and suggested fixes.
```

### Invoquer un sous-agent

```
Use a subagent to investigate how our authentication system handles token refresh.
Use a subagent to review this code for edge cases.
```

---

## 15. MCP — Model Context Protocol

MCP est un standard ouvert qui connecte Claude Code à des outils et sources de données externes — Linear, Slack, GitHub, documentation de dépendances, bases de données...

### Ajouter un serveur MCP

```bash
claude mcp add <nom-du-serveur>
```

Deux types : **HTTP** (services distants) et **Stdio** (processus locaux).

### Portée des serveurs

| Portée | Fichier de config | Partage |
|--------|------------------|---------|
| Local | Settings personnel | Vous uniquement |
| Utilisateur | Settings global | Tous vos projets |
| Projet | `.mcp.json` dans le repo | Toute l'équipe automatiquement |

> Pour partager les MCP avec l'équipe : commitez `.mcp.json` à la racine. Tout le monde obtient les mêmes serveurs au prochain `git pull`.

### Attention au contexte

Chaque serveur MCP connecté charge ses définitions d'outils dans le contexte, même quand vous ne l'utilisez pas. Si un outil a un équivalent CLI (`gh`, `aws`...), le CLI est plus économe en contexte. Vérifiez avec `/mcp` et désactivez les serveurs non pertinents à la tâche en cours.

---

## 16. Les Skills

### Le problème qu'elles résolvent

Chaque fois que vous expliquez vos conventions de PR à Claude, vous vous répétez. Les Skills permettent d'enseigner ces choses à Claude **une seule fois**. Claude les applique ensuite automatiquement, sans que vous ayez à les réécrire.

### Comment les Skills se positionnent

| Mécanisme | Chargement | Idéal pour |
|-----------|-----------|------------|
| `CLAUDE.md` | À chaque conversation | Standards permanents du projet |
| **Skills** | À la demande, selon le contexte | Expertise spécifique à une tâche |
| Slash commands | Invocation explicite | Actions ponctuelles déclenchées manuellement |
| Hooks | Événements (sauvegarde, commit...) | Automatisations déterministes |

L'avantage clé des Skills : elles ne consomment du contexte que quand elles sont pertinentes. Votre checklist de revue de PR n'a pas besoin d'être en mémoire quand vous déboguez.

### Anatomie d'une Skill

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

| Champ | Requis | Rôle |
|-------|--------|------|
| `name` | ✅ | Identifiant (minuscules, tirets, max 64 chars) |
| `description` | ✅ | Critère de déclenchement — ce que Claude lit pour décider si la skill est pertinente |
| `allowed-tools` | Non | Restreint les outils disponibles quand la skill est active |
| `model` | Non | Spécifie le modèle à utiliser |

> **Règle clé :** une description bien rédigée = une skill qui se déclenche au bon moment. Une description vague = une skill invisible.

### Créer sa première Skill

**Exemple — Skill de description de PR :**

```bash
mkdir -p ~/.claude/skills/pr-description
```

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
```

Redémarrez votre session Claude Code (les Skills sont chargées au démarrage), puis dites : *"écris une description de PR pour mes changements"*.

### Où vivent les Skills

| Type | Chemin | Portée |
|------|--------|--------|
| Personnelle | `~/.claude/skills/` | Tous vos projets |
| Projet | `.claude/skills/` à la racine du repo | Tout le monde sur ce repo |

### Configuration avancée

**Restreindre les outils avec `allowed-tools` :**
```yaml
---
name: exploration-codebase
description: Aide à comprendre l'architecture. À utiliser pour l'onboarding.
allowed-tools: Read, Grep, Glob, Bash
model: sonnet
---
```

Quand cette Skill est active, Claude ne peut pas modifier de fichiers.

**Progressive disclosure pour les grandes Skills :**
```
.claude/skills/ma-skill/
├── SKILL.md              # Instructions essentielles (< 500 lignes)
├── references/           # Documentation chargée à la demande
└── scripts/              # Scripts exécutables
```

Dans `SKILL.md`, indiquez quand charger les fichiers annexes : `"Pour les questions d'architecture, lire references/guide-archi.md."`

### Partager les Skills avec son équipe

| Niveau | Méthode | Pour |
|--------|---------|------|
| Repo Git | Committer `.claude/skills/` | Standards de l'équipe, workflows projet |
| Plugins | Marketplace Claude Code | Skills utiles à la communauté |
| Enterprise | Managed Settings | Standards obligatoires, conformité |

> Les Skills Enterprise ont la **priorité absolue** sur les Skills personnelles, projet et plugins de même nom.

### Skills et sous-agents

Les sous-agents ne voient **pas** automatiquement les Skills disponibles — ils démarrent avec un contexte vide. Pour créer un sous-agent avec des Skills :

```yaml
---
name: frontend-reviewer
description: "Utiliser pour la revue de code frontend, accessibilité et perf."
tools: Bash, Glob, Grep, Read
model: sonnet
skills: accessibility-audit, performance-check
---
```

### Diagnostiquer les problèmes

| Symptôme | Cause probable | Solution |
|----------|---------------|----------|
| La Skill ne se déclenche pas | Description trop vague | Ajouter des formulations que vous utilisez réellement |
| La Skill n'apparaît pas | Mauvaise structure de fichier | Vérifier que `SKILL.md` est dans un sous-dossier nommé |
| Mauvaise Skill utilisée | Descriptions trop similaires | Rendre les descriptions plus distinctes |
| Skill personnelle ignorée | Conflit de priorité | Renommer avec un nom plus spécifique |

Lancez `claude --debug` pour voir les erreurs de chargement.

---

## Partie 6 — Maîtrise avancée

## 17. Laisser Claude vous interviewer

> Pour les grandes fonctionnalités, laissez Claude vous interroger en premier — il pose des questions sur ce que vous n'avez peut-être pas encore considéré.

Au lieu d'écrire une spec complète vous-même, commencez par une description minimale et laissez Claude extraire les besoins par des questions structurées.

```
I want to build [brief description]. Interview me in detail using the AskUserQuestion tool.

Ask about technical implementation, UI/UX, edge cases, concerns, and tradeoffs.
Don't ask obvious questions — dig into the hard parts I might not have considered.

Keep interviewing until we've covered everything, then write a complete spec to SPEC.md.
```

Une fois la spec complète, **commencez une nouvelle session** pour l'implémenter. La nouvelle session a un contexte propre entièrement focalisé sur l'implémentation, et vous avez une référence écrite à laquelle vous comparer.

**Ce qui fait une bonne spec :** elle nomme les fichiers et interfaces concernés, dit explicitement ce qui est hors périmètre, et se termine par une étape de vérification bout-en-bout.

---

## 18. Gérer sa session comme un pro

### Corriger tôt et souvent

Les meilleurs résultats viennent de boucles de feedback serrées. Corrigez Claude dès que vous remarquez qu'il part dans la mauvaise direction.

| Action | Comment | Quand |
|--------|---------|-------|
| Stopper en plein milieu | `Esc` | Claude fait la mauvaise chose |
| Annuler et rediriger | `Esc + Esc` ou `/rewind` → restaurer | Mauvaise approche, essayer autrement |
| Revenir en arrière sur le code | "Undo that" | Garder la conversation, annuler le code |
| Repartir à zéro | `/clear` | Nouvelle tâche non liée, ou après deux corrections échouées |

> **Règle :** si vous avez corrigé Claude plus de deux fois sur le même problème, lancez `/clear` et écrivez un meilleur prompt initial. Une session propre avec un meilleur prompt surpasse presque toujours une longue session avec des corrections accumulées.

### Checkpoints et rembobinage

Chaque prompt que vous envoyez crée un checkpoint. Claude prend un instantané des fichiers avant chaque modification.

- `Esc + Esc` ou `/rewind` — ouvre le menu de rembobinage
- Options : restaurer la conversation seulement, le code seulement, les deux, ou résumer depuis un message
- Les checkpoints persistent entre sessions — fermez votre terminal et rembobinez plus tard

Vous pouvez donc dire à Claude d'essayer quelque chose de risqué. Si ça ne fonctionne pas, rembobinez et essayez une approche différente.

> Les checkpoints ne suivent que les modifications faites *par Claude*, pas les processus externes. Ce n'est pas un substitut à git.

### Reprendre des sessions

```bash
claude --continue        # Reprendre la session la plus récente
claude --resume          # Choisir dans une liste de sessions sauvegardées
```

Utilisez `/rename` pour donner aux sessions des noms descriptifs comme `migration-oauth` pour les retrouver plus tard. Traitez les sessions comme des branches : chaque workstream obtient son propre contexte persistant.

### Questions de côté avec `/btw`

Pour les questions rapides dont vous n'avez pas besoin dans le contexte, utilisez `/btw`. La réponse apparaît dans un overlay qu'on peut fermer et n'entre jamais dans l'historique de conversation.

```
/btw que fait le flag --no-ff dans git merge ?
```

---

## 19. Automatiser et passer à l'échelle

### Mode non-interactif (headless)

Utilisez `claude -p "prompt"` pour lancer Claude sans session — dans la CI, des pre-commit hooks, ou des scripts.

```bash
# Requête ponctuelle
claude -p "Explain what this project does"

# Sortie structurée pour des scripts
claude -p "List all API endpoints" --output-format json

# Streaming pour traitement en temps réel
claude -p "Analyze this log file" --output-format stream-json --verbose
```

Utilisez `--allowedTools` pour restreindre ce que Claude peut faire dans les exécutions automatisées :

```bash
claude -p "Fix all lint errors" --allowedTools "Edit,Bash(npm run lint)"
```

### Exécuter plusieurs sessions en parallèle

| Approche | Comment | Pour |
|----------|---------|------|
| Worktrees | Sessions CLI séparées dans des checkouts git isolés | Fonctionnalités parallèles sans collisions d'édition |
| Desktop app | Plusieurs sessions locales, chacune dans son worktree | Gestion visuelle de workstreams parallèles |
| Web (claude.ai/code) | Sessions sur infrastructure cloud dans des VMs isolées | Travail distant sur des repos GitHub |
| Agent teams | Coordination automatisée avec tâches partagées et team lead | Workflows multi-agents complexes |

### Fan-out sur des fichiers à grande échelle

Pour les grandes migrations ou analyses, distribuez le travail en plusieurs invocations Claude parallèles.

**Étape 1 — Générer une liste de tâches :**
```
liste tous les 200 fichiers Python qui doivent migrer de requests vers httpx
```

**Étape 2 — Écrire une boucle :**
```bash
for file in $(cat files.txt); do
  claude -p "Migrate $file from requests to httpx. Return OK or FAIL." \
    --allowedTools "Edit,Bash(git commit *)"
done
```

**Étape 3 — Tester sur 2-3 fichiers, puis passer à l'échelle.** Affinez votre prompt en fonction de ce qui ne fonctionne pas d'abord, puis lancez sur l'ensemble.

### Auto mode pour les exécutions autonomes

Pour des exécutions sans interruption avec vérification en arrière-plan :

```bash
claude --permission-mode auto -p "fix all lint errors"
```

Un classifier bloque les actions risquées (escalade de scope, infrastructure inconnue, actions pilotées par du contenu hostile) tout en laissant le travail de routine avancer sans prompts.

---

## 20. Les patterns d'échec courants

Reconnaître ces patterns tôt fait gagner beaucoup de temps.

### La session "tout dans le même sac"

Vous commencez avec une tâche, posez une question non liée, revenez à la première tâche. Le contexte est rempli d'informations non pertinentes et la qualité des réponses baisse.

**Solution :** `/clear` entre les tâches non liées.

### La correction en boucle

Claude fait quelque chose de faux, vous corrigez, c'est toujours faux, vous recorrigez. Le contexte est pollué par des approches échouées et Claude continue d'essayer des variations de la même erreur.

**Solution :** après deux corrections échouées, `/clear` et écrivez un meilleur prompt initial qui intègre ce que vous avez appris.

### Le CLAUDE.md surchargé

CLAUDE.md est trop long, les règles importantes se noient dans le bruit. Claude en ignore la moitié.

**Solution :** élagage impitoyable. Si Claude fait déjà quelque chose correctement sans l'instruction, supprimez-la. Convertissez les règles critiques en hooks pour une application déterministe.

### L'écart "faire confiance puis vérifier"

Claude produit une implémentation qui semble plausible mais ne gère pas les cas limites — et vous la livrez sans vérifier.

**Solution :** fournissez toujours une vérification (tests, scripts, captures d'écran). Si vous ne pouvez pas vérifier, ne livrez pas.

### L'exploration infinie

Vous demandez à Claude "d'explorer" quelque chose sans cadrer la recherche. Claude lit des centaines de fichiers, remplissant le contexte avec un travail exploratoire que vous n'aviez pas besoin de voir.

**Solution :** cadrez les explorations étroitement, ou utilisez des subagents pour que l'exploration ne consomme pas votre contexte principal.

---

## Récapitulatif global

| Concept | À retenir |
|---------|-----------|
| **Agent** | Claude Code agit directement sur vos fichiers et commandes — pas de copier-coller |
| **Boucle agentique** | Collecte → Action → Vérification → recommence si nécessaire |
| **Fenêtre de contexte** | Ressource limitée — tout le reste en découle |
| **Plan Mode** | `Shift+Tab` — explore et planifie avant de coder. Idéal pour les tâches complexes |
| **Workflow principal** | Explore → Plan → Code → Commit |
| **Bons prompts** | Précis, avec contexte riche, critère de succès, références `@` |
| **Vérification** | Donnez à Claude un test à lancer — preuves, pas assertions |
| **CLAUDE.md** | Court, actionnable, pour ce que Claude ne peut pas deviner |
| **Hooks** | Guardrails déterministes — règles critiques, pas préférences |
| **Sous-agents** | Contexte isolé pour déléguer sans polluer la session principale |
| **MCP** | Connexion à des outils externes (Linear, Slack, GitHub...) |
| **Skills** | Expertise à la demande — chargée uniquement quand pertinente |
| **Checkpoints** | `Esc+Esc` ou `/rewind` — essayez des choses risquées, rembobinez si besoin |
| **Sessions** | `--continue`, `--resume`, `/rename` — reprenez le contexte sans le réexpliquer |
| **Scale** | `claude -p` en boucle, sessions parallèles, pattern Writer/Reviewer |
