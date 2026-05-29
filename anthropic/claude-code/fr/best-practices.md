# Claude Code — Bonnes pratiques

> Patterns concrets pour tirer le meilleur de Claude Code, du prompting efficace à la mise à l'échelle en sessions parallèles.

**Source :** [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) — Documentation officielle  
**Prérequis :** [Claude Code 101](./claude-code-101.md) · [Introduction aux Agent Skills](./introduction-to-agent-skills.md)

---

## Table des matières

1. [La contrainte centrale](#1-la-contrainte-centrale)
2. [Donner à Claude un moyen de vérifier son travail](#2-donner-à-claude-un-moyen-de-vérifier-son-travail)
3. [Fournir un contexte riche et précis](#3-fournir-un-contexte-riche-et-précis)
4. [Configurer son environnement](#4-configurer-son-environnement)
5. [Laisser Claude vous interviewer](#5-laisser-claude-vous-interviewer)
6. [Gérer sa session activement](#6-gérer-sa-session-activement)
7. [Automatiser et passer à l'échelle](#7-automatiser-et-passer-à-léchelle)
8. [Les patterns d'échec courants](#8-les-patterns-déchec-courants)

---

## 1. La contrainte centrale

Presque toutes les bonnes pratiques de ce guide découlent d'une seule contrainte : **la fenêtre de contexte de Claude se remplit vite, et les performances se dégradent à mesure qu'elle se remplit.**

Tout ce que Claude "voit" pendant une session — vos messages, les fichiers lus, les sorties de commandes — occupe de l'espace dans sa fenêtre de contexte. Une seule session de débogage peut consommer des dizaines de milliers de tokens. À mesure que la fenêtre se remplit, Claude peut oublier des instructions précédentes ou faire davantage d'erreurs.

**La fenêtre de contexte est la ressource la plus importante à gérer.** Toutes les sections suivantes reviennent à ce point.

---

## 2. Donner à Claude un moyen de vérifier son travail

> Donnez à Claude un test qu'il peut exécuter : une suite de tests, un build, une capture d'écran à comparer. C'est la différence entre une session que vous surveillez et une session à laquelle vous pouvez déléguer en confiance.

Claude s'arrête quand le travail *semble* terminé. Sans vérification automatique, vous devenez la boucle de vérification — chaque erreur attend que vous la remarquiez.

La vérification peut être n'importe quoi qui retourne un résultat lisible : une suite de tests, un code de sortie de build, un linter, un script qui compare la sortie à un fichier de référence.

### Exemples avant / après

| Stratégie | Prompt faible | Prompt fort |
|-----------|---------------|-------------|
| **Critère de vérification** | *"implémente une fonction validateEmail"* | *"écris validateEmail. cas de test : user@example.com → true, invalid → false, user@.com → false. lance les tests après l'implémentation"* |
| **Vérification visuelle UI** | *"améliore l'apparence du dashboard"* | *"[capture d'écran] implémente ce design. prends une capture du résultat et compare-la à l'original. liste les différences et corrige-les"* |
| **Cause racine** | *"le build plante"* | *"le build échoue avec cette erreur : [coller l'erreur]. corrige-le et vérifie que le build réussit. traite la cause racine, ne supprime pas l'erreur"* |

### Niveaux de vérification

| Niveau | Mécanisme | Mise en place | Pour |
|--------|-----------|---------------|------|
| Dans le prompt | Demander à Claude de lancer le test et d'itérer | Aucune | N'importe quelle tâche |
| Pendant la session | Condition `/goal` — un évaluateur séparé revérifie après chaque tour | Faible | Tâches longues |
| Garde déterministe | Stop hook — bloque la fin du tour jusqu'à ce que le test passe | Moyen | Exécutions sans surveillance |
| Second avis | Subagent de vérification — un modèle frais relit le résultat | Moyen | Code critique |

### Demander des preuves, pas des affirmations

Demandez à Claude de vous montrer la sortie des tests, la commande exécutée, ou une capture d'écran — plutôt que de simplement dire "c'est fait". Relire des preuves est plus rapide que de re-lancer la vérification vous-même.

---

## 3. Fournir un contexte riche et précis

> Plus vos instructions sont précises, moins vous aurez de corrections à faire.

Claude peut inférer l'intention, mais ne peut pas lire dans vos pensées. Référencez des fichiers spécifiques, mentionnez les contraintes, et pointez vers des patterns existants dans votre codebase.

### Patterns de prompting

| Stratégie | Faible | Fort |
|-----------|--------|------|
| **Cadrer la tâche** | *"ajoute des tests pour foo.py"* | *"écris un test pour foo.py couvrant le cas où l'utilisateur est déconnecté. évite les mocks."* |
| **Pointer vers les sources** | *"pourquoi ExecutionFactory a une API aussi bizarre ?"* | *"regarde l'historique git de ExecutionFactory et résume comment son API en est arrivée là"* |
| **Référencer les patterns** | *"ajoute un widget calendrier"* | *"regarde HotDogWidget.php pour comprendre notre pattern de widget, puis implémente un widget calendrier qui permet de sélectionner un mois et paginer par année. pas de nouvelles bibliothèques."* |
| **Décrire le symptôme** | *"corrige le bug de login"* | *"les utilisateurs rapportent que le login échoue après expiration de session. vérifie le flux auth dans src/auth/, notamment le token refresh. écris un test qui reproduit le bug, puis corrige-le"* |

### Façons de fournir du contexte riche

- **`@filename`** — référencer un fichier directement ; Claude le lit avant de répondre
- **Coller des images** — copier-coller ou glisser des captures d'écran, maquettes, ou diagrammes
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

## 4. Configurer son environnement

### CLAUDE.md : ce qu'il faut inclure et ce qu'il faut couper

`CLAUDE.md` est chargé à chaque session. Restez concis — un fichier trop long pousse Claude à ignorer les règles noyées dans le bruit.

| ✅ À inclure | ❌ À exclure |
|-------------|------------|
| Commandes Bash que Claude ne peut pas deviner | Ce que Claude peut inférer en lisant le code |
| Règles de style différentes des conventions par défaut | Conventions standard du langage |
| Runner de tests et instructions de test | Documentation d'API détaillée (mettre un lien) |
| Convention de nommage des branches et des PR | Informations qui changent fréquemment |
| Décisions d'architecture spécifiques au projet | Longues explications ou tutoriels |
| Particularités de l'environnement (variables d'env requises) | Descriptions fichier par fichier de la codebase |

**Règle d'or :** pour chaque ligne, demandez *"Supprimer cette ligne ferait-il faire des erreurs à Claude ?"* Si non, coupez-la.

**Conseil :** si Claude continue d'enfreindre une règle malgré sa présence dans `CLAUDE.md`, le fichier est probablement trop long. Convertissez les règles critiques en hooks pour une application déterministe (voir [Claude Code 101 — Hooks](./claude-code-101.md#84-les-hooks)).

### Modes de permission

Par défaut, Claude demande une approbation avant chaque écriture de fichier ou commande. Après la dixième approbation, vous ne relisez plus vraiment — vous cliquez mécaniquement.

| Mode | Comment | Quand l'utiliser |
|------|---------|-----------------|
| **Approbation (défaut)** | Demande avant chaque action | Nouvelles tâches, codebases inconnues |
| **Auto mode** | Un classifier bloque les actions risquées ; le reste passe sans prompts | Quand vous faites confiance à la direction générale |
| **Listes blanches** | Autoriser des commandes sûres spécifiques : `npm run lint`, `git commit` | Opérations connues et sûres |
| **Sandboxing** | Isolation OS qui restreint filesystem et réseau | Exécutions sans surveillance avec inputs externes |

Configurer via `/permissions` ou directement dans `.claude/settings.json`.

---

## 5. Laisser Claude vous interviewer

> Pour les grandes fonctionnalités, laissez Claude vous interroger en premier — il pose des questions sur ce que vous n'avez peut-être pas encore considéré.

Au lieu d'écrire une spec complète vous-même, commencez par une description minimale et laissez Claude extraire les besoins par des questions structurées.

```
I want to build [brief description]. Interview me in detail using the AskUserQuestion tool.

Ask about technical implementation, UI/UX, edge cases, concerns, and tradeoffs.
Don't ask obvious questions — dig into the hard parts I might not have considered.

Keep interviewing until we've covered everything, then write a complete spec to SPEC.md.
```

Une fois la spec complète, **commencez une nouvelle session** pour l'implémenter. La nouvelle session a un contexte propre entièrement focalisé sur l'implémentation, et vous avez une référence écrite à laquelle vous comparer.

**Ce qui fait une bonne spec :** elle nomme les fichiers et interfaces concernés, dit explicitement ce qui est hors périmètre, et se termine par une étape de vérification bout-en-bout qui prouve que la fonctionnalité fonctionne.

---

## 6. Gérer sa session activement

### Corriger tôt et souvent

Les meilleurs résultats viennent de boucles de feedback serrées. Corrigez Claude dès que vous remarquez qu'il part dans la mauvaise direction — attendre n'est jamais plus rapide.

| Action | Comment | Quand |
|--------|---------|-------|
| Stopper en plein milieu | `Esc` | Claude fait la mauvaise chose |
| Annuler et rediriger | `Esc + Esc` ou `/rewind` → restaurer | Mauvaise approche, essayer autrement |
| Revenir en arrière sur le code | "Undo that" | Garder la conversation, annuler le code |
| Repartir à zéro | `/clear` | Nouvelle tâche non liée, ou après deux corrections échouées |

> **Règle :** si vous avez corrigé Claude plus de deux fois sur le même problème, lancez `/clear` et écrivez un meilleur prompt initial qui intègre ce que vous avez appris. Une session propre avec un meilleur prompt surpasse presque toujours une longue session avec des corrections accumulées.

### Checkpoints et rembobinage

Chaque prompt que vous envoyez crée un checkpoint. Claude prend un instantané des fichiers avant chaque modification.

- `Esc + Esc` ou `/rewind` — ouvre le menu de rembobinage
- Options : restaurer la conversation seulement, le code seulement, les deux, ou résumer depuis un message
- Les checkpoints persistent entre sessions — fermez votre terminal et rembobinez plus tard

Cela signifie que vous pouvez dire à Claude d'essayer quelque chose de risqué. Si ça ne fonctionne pas, rembobinez et essayez une approche différente.

> Les checkpoints ne suivent que les modifications faites *par Claude*, pas les processus externes. Ce n'est pas un substitut à git.

### Reprendre des sessions

Claude Code sauvegarde les conversations localement. Vous n'avez pas à ré-expliquer le contexte quand vous reprenez une tâche.

```bash
claude --continue        # Reprendre la session la plus récente
claude --resume          # Choisir dans une liste de sessions sauvegardées
```

Utilisez `/rename` pour donner aux sessions des noms descriptifs comme `migration-oauth` pour les retrouver plus tard.

### Questions de côté avec `/btw`

Pour les questions rapides dont vous n'avez pas besoin dans le contexte, utilisez `/btw`. La réponse apparaît dans un overlay qu'on peut fermer et n'entre jamais dans l'historique de conversation.

```
/btw que fait le flag --no-ff dans git merge ?
```

### Utiliser des subagents pour l'exploration

Quand Claude explore une codebase, il lit des dizaines de fichiers — tout cela consomme du contexte. Les subagents explorent dans une fenêtre de contexte séparée et ne retournent que le résultat.

```
Use subagents to investigate how our authentication system handles token
refresh, and whether we have any existing OAuth utilities I should reuse.
```

Le subagent lit tout, vous n'obtenez que la conclusion.

---

## 7. Automatiser et passer à l'échelle

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

### Pattern Writer / Reviewer

Un contexte frais améliore la revue de code — Claude ne sera pas biaisé vers le code qu'il vient d'écrire.

| Session A (Writer) | Session B (Reviewer) |
|--------------------|----------------------|
| `Implement a rate limiter for our API endpoints` | |
| | `Review the rate limiter implementation in @src/middleware/rateLimiter.ts. Look for edge cases, race conditions, and consistency with our existing middleware patterns.` |
| `Here's the review feedback: [résultat]. Address these issues.` | |

Même principe avec les tests : un Claude écrit les tests, un autre écrit le code pour les faire passer.

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

**Étape 3 — Tester sur 2-3 fichiers, puis passer à l'échelle.** Affinez votre prompt en fonction de ce qui ne fonctionne pas d'abord.

### Revue adversariale

Avant de considérer une tâche comme terminée, faites relire le résultat par un subagent dans un contexte frais.

```
Use a subagent to review the rate limiter diff against PLAN.md. Check that
every requirement is implemented, the listed edge cases have tests, and
nothing outside the task's scope changed. Report gaps, not style preferences.
```

Parce que le reviewer tourne comme subagent, la session principale reçoit les gaps directement et peut les corriger et re-relire sans que vous n'ayez à copier les résultats entre fenêtres.

> Un reviewer invité à trouver des lacunes en trouvera souvent, même quand le travail est solide. Dites-lui de ne signaler que les gaps qui affectent la correction ou les exigences déclarées — pas les préférences de style.

---

## 8. Les patterns d'échec courants

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

## Référence rapide

| Pattern | Mécanisme |
|---------|-----------|
| Vérifier le travail automatiquement | Stop hook, `/goal`, ou subagent vérificateur |
| Éviter le gonflement du contexte lors de recherches | Utiliser des subagents |
| Garder les règles critiques appliquées | Hooks, pas CLAUDE.md |
| Récupérer d'une mauvaise direction | `Esc+Esc` ou `/rewind` |
| Reprendre entre sessions | `claude --continue` ou `claude --resume` |
| Question rapide sans gonfler le contexte | `/btw` |
| Lancer Claude dans la CI | `claude -p "prompt" --output-format json` |
| Workstreams parallèles | Worktrees + plusieurs sessions |
| Revue de code non biaisée | Pattern Writer/Reviewer |
| Migration à grande échelle | Boucle `claude -p` avec `--allowedTools` |
