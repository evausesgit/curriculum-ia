# CLI Agentique — Guide fondamental

> Ce guide couvre les concepts universels qui s'appliquent à tout CLI agentique : Claude Code, OpenCode, Cursor, Copilot CLI, et les outils à venir. Lisez-le avant d'aborder la documentation d'un outil spécifique.

---

## Table des matières

**Partie 1 — Comprendre un CLI agentique**
1. [Agent vs assistant : la différence fondamentale](#1-agent-vs-assistant--la-différence-fondamentale)
2. [La contrainte centrale : la fenêtre de contexte](#2-la-contrainte-centrale--la-fenêtre-de-contexte)

**Partie 2 — Travailler avec un agent**
3. [Le workflow Explore → Plan → Code → Commit](#3-le-workflow-explore--plan--code--commit)
4. [Écrire de bons prompts](#4-écrire-de-bons-prompts)
5. [Gérer le contexte activement](#5-gérer-le-contexte-activement)

**Partie 3 — Vérification et qualité**
6. [Donner à l'agent un moyen de vérifier son travail](#6-donner-à-lagent-un-moyen-de-vérifier-son-travail)
7. [Revue de code avec un agent](#7-revue-de-code-avec-un-agent)

**Partie 4 — Configurer son environnement**
8. [Le fichier de configuration du projet](#8-le-fichier-de-configuration-du-projet)
9. [Les modes de permission](#9-les-modes-de-permission)
10. [Les automatisations déterministes](#10-les-automatisations-déterministes)
11. [Les modules d'instructions réutilisables](#11-les-modules-dinstructions-réutilisables)
12. [Les agents parallèles isolés](#12-les-agents-parallèles-isolés)
13. [Les connecteurs d'outils externes](#13-les-connecteurs-doutils-externes)

**Partie 5 — Maîtrise avancée**
14. [Laisser l'agent vous interviewer](#14-laisser-lagent-vous-interviewer)
15. [Gérer sa session activement](#15-gérer-sa-session-activement)
16. [Automatiser et passer à l'échelle](#16-automatiser-et-passer-à-léchelle)
17. [Les patterns d'échec courants](#17-les-patterns-déchec-courants)

[Récapitulatif](#récapitulatif)

---

## Partie 1 — Comprendre un CLI agentique

## 1. Agent vs assistant : la différence fondamentale

### Ce que vous avez probablement déjà utilisé

La plupart des développeurs ont utilisé un assistant IA en copiant-collant du code dans une interface de chat. L'assistant répond, vous copiez, vous collez, vous testez, vous revenez. C'est un aller-retour manuel.

### Ce qu'un CLI agentique fait différemment

Un CLI agentique a un **accès direct** à votre environnement de développement. Il n'a pas besoin que vous lui montriez le code — il le lit lui-même, l'édite, et exécute des commandes. Vous décrivez un objectif ; l'agent trouve comment l'atteindre.

Concrètement, un agent peut :

- **Lire et comprendre votre codebase** — explorer les fichiers, tracer un bug, comprendre une architecture
- **Modifier des fichiers** — refactoriser une fonction et mettre à jour tous les fichiers qui l'utilisent
- **Exécuter des commandes** — lancer les tests, installer des dépendances, lire les logs, et utiliser les résultats pour décider quoi faire ensuite
- **Chercher des informations** — consulter de la documentation ou des références d'API

### La boucle agentique

À chaque fois que vous envoyez un message, l'agent suit cette boucle :

```
Votre prompt
    ↓
Collecte du contexte (lecture de fichiers, recherche...)
    ↓
Action (modification de fichier, exécution de commande...)
    ↓
Vérification (les résultats répondent-ils à la demande ?)
    ↓
Si oui → attend votre prochain message
Si non → recommence la boucle
```

Vous pouvez interrompre ou orienter l'agent à n'importe quelle étape.

### Trois choses à garder en tête

**La fenêtre de contexte, c'est sa mémoire de travail.** L'agent peut tenir beaucoup d'informations en tête, mais pas tout votre projet à la fois. Il explore stratégiquement votre codebase pour trouver ce dont il a besoin.

**Il demande votre accord avant d'agir.** Par défaut, la plupart des agents attendent votre validation avant de modifier un fichier ou d'exécuter une commande. Vous gardez le contrôle.

**Il peut se tromper.** Comme tout outil, il peut mal interpréter une demande ou introduire un bug. Rester dans la boucle permet d'attraper ces erreurs tôt.

---

## 2. La contrainte centrale : la fenêtre de contexte

Presque toutes les bonnes pratiques de ce guide découlent d'une seule contrainte : **la fenêtre de contexte de l'agent se remplit vite, et les performances se dégradent à mesure qu'elle se remplit.**

Tout ce que l'agent "voit" pendant une session — vos messages, les fichiers lus, les sorties de commandes — occupe de l'espace dans sa fenêtre de contexte. Une seule session de débogage peut consommer des dizaines de milliers de tokens. Quand la fenêtre est pleine, l'agent peut oublier des instructions précédentes ou faire davantage d'erreurs.

**La fenêtre de contexte est la ressource la plus importante à gérer.**

Retenez cette idée : chaque technique de ce guide — les prompts précis, les agents parallèles, les instructions à la demande, les commandes de réinitialisation — vise, directement ou indirectement, à préserver cet espace pour ce qui compte vraiment.

---

## Partie 2 — Travailler avec un agent

## 3. Le workflow Explore → Plan → Code → Commit

C'est le workflow central à adopter. Il évite le principal piège : demander du code directement sans établir de contexte, ce qui entraîne des corrections coûteuses en cours de route.

### Explore

Avant de toucher du code, donnez à l'agent le contexte dont il a besoin. La plupart des outils proposent un mode lecture seule pour cette phase. Vous pouvez aussi déclencher l'exploration explicitement pour obtenir un résumé d'architecture sans intention immédiate de modifier quoi que ce soit.

```
Je dois ajouter la conversion WebP dans notre pipeline d'upload d'images.
Dis-moi où dans le pipeline ça devrait s'insérer, quelles dépendances
seraient nécessaires, et comment tu envisages l'implémentation.
```

### Plan

L'agent analyse les fichiers pertinents, fait des recherches si besoin, et vous soumet un plan d'action. C'est le **meilleur moment pour corriger le tir** — avant que du code soit écrit. Posez des questions, demandez des révisions. Un plan bien validé mène à une implémentation propre.

> **Conseil :** demandez à l'agent de sauvegarder le plan dans un fichier (`PLAN.md`, `SPEC.md`…). Vous aurez une référence écrite pour la vérification en fin de tâche.

### Code

Une fois le plan validé, l'agent travaille la liste d'étapes. Conseils pour cette phase :

- **Critère de succès explicite** — indiquez ce que "correct" signifie (tests verts, comportement attendu, capture d'écran conforme au design)
- **Suite de tests** — donnez-la à l'agent comme source de vérité continue ; il peut aussi en écrire une pour vous
- **Mémorisez les corrections** — si l'agent répète les mêmes erreurs, ajoutez la règle dans votre fichier de configuration de projet

### Commit

Avant de pousser votre code :

1. **Lancez une revue par un agent parallèle** — regard neuf, sans le biais de la session de codage
2. **Demandez à l'agent de générer le message de commit** dans le style de votre équipe

---

## 4. Écrire de bons prompts

Le niveau de précision de votre prompt détermine directement la qualité du résultat et la quantité de contexte consommée. Un prompt vague force l'agent à explorer plus largement pour deviner votre intention.

### Patterns de prompting

| Stratégie | Prompt faible | Prompt fort |
|-----------|---------------|-------------|
| **Cadrer la tâche** | *"ajoute des tests pour foo.py"* | *"écris un test pour foo.py couvrant le cas où l'utilisateur est déconnecté. évite les mocks."* |
| **Pointer vers les sources** | *"pourquoi ExecutionFactory a une API bizarre ?"* | *"regarde l'historique git de ExecutionFactory et résume comment son API en est arrivée là"* |
| **Référencer les patterns** | *"ajoute un widget calendrier"* | *"regarde HotDogWidget.php pour comprendre notre pattern de widget, puis implémente un widget calendrier qui permet de sélectionner un mois et paginer par année. pas de nouvelles bibliothèques."* |
| **Décrire le symptôme** | *"corrige le bug de login"* | *"les utilisateurs rapportent que le login échoue après expiration de session. vérifie le flux auth dans src/auth/, notamment le token refresh. écris un test qui reproduit le bug, puis corrige-le"* |
| **Critère de vérification** | *"implémente validateEmail"* | *"écris validateEmail. cas de test : user@example.com → true, invalid → false, user@.com → false. lance les tests après."* |

### Fournir du contexte riche

La plupart des CLI agentiques permettent d'enrichir vos prompts au-delà du texte :

- **Référencer des fichiers** — pointez vers un fichier spécifique plutôt que de décrire son contenu
- **Coller des images** — captures d'écran, maquettes, diagrammes
- **Donner des URLs** — liens de documentation, issues GitHub, specs d'API
- **Piper des données** — envoyer directement le contenu d'un fichier (logs, JSON, CSV)
- **Laisser l'agent chercher** — lui demander d'utiliser ses outils pour récupérer ce dont il a besoin

### Utiliser les outils CLI existants

Les agents sont efficaces avec les outils CLI comme `gh` (GitHub), `aws`, `gcloud`. Ces outils sont plus économes en contexte que leurs équivalents d'intégration directe, et la plupart des agents les connaissent déjà.

Pour les outils inconnus de l'agent : *"utilise `foo-cli --help` pour apprendre l'outil, puis fais X avec."*

---

## 5. Gérer le contexte activement

### Les opérations fondamentales

Chaque outil agentique propose des équivalents de ces trois opérations :

| Opération | Effet | Quand l'utiliser |
|-----------|-------|-----------------|
| **Compacter** | Résume et compresse l'historique pour libérer de la place | En cours de feature, quand on approche de la limite |
| **Effacer** | Repart de zéro, contexte vide | Entre deux features distinctes |
| **Inspecter** | Affiche l'utilisation actuelle du contexte | Pour diagnostiquer ce qui consomme de l'espace |

> **Règle pratique :** compactez en cours de feature, effacez entre features. Ce que vous voulez que l'agent retienne d'une session à l'autre → dans votre fichier de configuration de projet.

### Stratégies pour économiser du contexte

**Soyez précis dans vos prompts.** Un prompt vague oblige l'agent à explorer plus largement pour deviner votre intention.

**Désactivez les intégrations inutilisées.** Chaque outil ou connecteur activé charge ses définitions dans le contexte, même quand vous ne l'utilisez pas.

**Déléguez les recherches à un agent parallèle.** Quand vous avez besoin d'explorer votre codebase pour une question précise, un agent parallèle fait le travail dans son propre contexte et retourne uniquement la réponse.

---

## Partie 3 — Vérification et qualité

## 6. Donner à l'agent un moyen de vérifier son travail

> Donnez à l'agent un test qu'il peut exécuter : une suite de tests, un build, une capture d'écran à comparer. C'est la différence entre une session que vous surveillez et une session que vous pouvez déléguer.

L'agent s'arrête quand le travail *semble* terminé. Sans vérification automatique, vous devenez la boucle de vérification — chaque erreur attend que vous la remarquiez.

### Niveaux de vérification

| Niveau | Mécanisme | Mise en place | Pour |
|--------|-----------|---------------|------|
| **Dans le prompt** | Demander à l'agent de lancer le test et d'itérer | Aucune | N'importe quelle tâche |
| **Condition de session** | Objectif défini en début de session — l'agent revérifie après chaque action | Faible | Tâches longues |
| **Garde déterministe** | Automatisation qui bloque la fin du tour jusqu'à ce que le test passe | Moyen | Exécutions sans surveillance |
| **Second avis** | Agent parallèle de vérification dans un contexte frais | Moyen | Code critique |

### Demander des preuves, pas des affirmations

Demandez à l'agent de vous montrer la sortie des tests, la commande exécutée, ou une capture d'écran — plutôt que de simplement dire "c'est fait". Relire des preuves est plus rapide que de re-lancer la vérification vous-même.

---

## 7. Revue de code avec un agent

### Le principe du contexte frais

Avant de pousser une PR, déléguez la revue à un agent parallèle. L'avantage clé : l'agent parallèle démarre avec un contexte vierge, sans le biais accumulé pendant la session de codage.

**Configuration recommandée :**
- Outils en lecture seule uniquement — un reviewer signale des problèmes, il ne corrige pas
- Versionner la configuration dans le repo pour que toute l'équipe utilise le même reviewer

### Pattern Writer / Reviewer

Un contexte frais améliore la revue de code — l'agent ne sera pas biaisé vers le code qu'il vient d'écrire.

| Session A (Writer) | Session B (Reviewer) |
|--------------------|----------------------|
| *Implémente le rate limiter* | |
| | *Relis l'implémentation dans [fichier]. Cherche les edge cases, les race conditions, la cohérence avec le middleware existant.* |
| *Voici le feedback : [résultat]. Corrige ces points.* | |

Même principe avec les tests : un agent écrit les tests, un autre écrit le code pour les faire passer.

### Revue adversariale

Avant de considérer une tâche comme terminée, faites relire le résultat par un agent parallèle dans un contexte frais, en lui donnant le plan original comme référence.

```
Relis le diff par rapport à PLAN.md. Vérifie que chaque exigence est
implémentée, que les edge cases listés ont des tests, et que rien hors
du périmètre n'a changé. Signale les lacunes, pas les préférences de style.
```

> Un reviewer invité à trouver des lacunes en trouvera souvent, même quand le travail est solide. Dites-lui de ne signaler que les gaps qui affectent la correction ou les exigences déclarées.

---

## Partie 4 — Configurer son environnement

## 8. Le fichier de configuration du projet

Chaque CLI agentique majeur propose un fichier de configuration que l'agent lit automatiquement à chaque démarrage de session. C'est l'**onboarding document pour l'agent** — il évite de redécouvrir les mêmes choses à chaque fois.

### Équivalents selon les outils

| Outil | Fichier de configuration |
|-------|------------------------|
| Claude Code | `CLAUDE.md` |
| Cursor | `.cursorrules` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| OpenCode | `AGENTS.md` |

### Ce qu'il faut inclure

| ✅ À inclure | ❌ À exclure |
|-------------|------------|
| Commandes de build, test, lint | Ce que l'agent peut inférer en lisant le code |
| Règles de style différentes des conventions par défaut | Conventions standard du langage |
| Convention de nommage des branches et des PR | Documentation d'API détaillée (mettre un lien) |
| Décisions d'architecture spécifiques au projet | Informations qui changent fréquemment |
| Particularités de l'environnement (variables d'env requises) | Longues explications ou tutoriels |

**Règle d'or :** pour chaque ligne, demandez *"Supprimer cette ligne ferait-il faire des erreurs à l'agent ?"* Si non, coupez-la. Un fichier trop long pousse l'agent à ignorer les règles noyées dans le bruit.

### Conseils pratiques

**Laissez émerger le contenu.** Commencez sans fichier de configuration et observez où vous corrigez souvent l'agent. Ces corrections méritent d'être mémorisées.

**Plusieurs niveaux de portée.** La plupart des outils permettent un fichier global (tous vos projets) et un fichier projet (toute l'équipe via git). Utilisez-les de façon complémentaire.

**Mémorisez les corrections récurrentes.** Si vous vous surprenez à redire la même chose, ajoutez la règle au fichier de configuration.

---

## 9. Les modes de permission

Par défaut, l'agent demande une approbation avant chaque écriture de fichier ou commande. Après la dixième approbation, vous ne relisez plus vraiment — vous cliquez mécaniquement.

### Spectrum de contrôle

```
Contrôle maximum ←————————————————→ Autonomie maximum
     Approbation    Listes blanches    Auto    Sandbox
     (défaut)       sélectives         mode
```

| Mode | Comportement | Quand l'utiliser |
|------|-------------|-----------------|
| **Approbation** | Demande avant chaque action | Nouvelles tâches, codebases inconnues |
| **Listes blanches** | Certaines commandes autorisées automatiquement | Opérations connues et répétitives |
| **Auto / sans surveillance** | L'outil gère les approbations avec une classification de risque | Tâches longues dans une direction de confiance |
| **Sandbox** | Isolation OS, filesystem et réseau restreints | Exécutions avec inputs externes ou non vérifiés |

---

## 10. Les automatisations déterministes

La plupart des CLI agentiques permettent de définir des scripts qui s'exécutent automatiquement à des moments précis du workflow. Leur force : **ils sont déterministes**. Mettre une règle dans le fichier de configuration → l'agent la respecte *la plupart du temps*. Une automatisation → la règle s'applique **à chaque fois, sans exception**.

### Points de déclenchement courants

| Événement | Exemples d'usage |
|-----------|-----------------|
| Avant une action | Bloquer les modifications de fichiers sensibles |
| Après une action | Formater automatiquement le code édité |
| À la soumission d'un prompt | Enrichir ou valider le prompt |
| À la fin d'une réponse | Lancer les tests, envoyer une notification |

### Cas d'usage typiques

- **Auto-formatage** : après chaque édition de fichier → Prettier, `gofmt`, etc.
- **Audit** : logger toutes les commandes exécutées par l'agent
- **Guardrails** : bloquer les modifications de config prod, les `rm -rf`, les commits directs sur `main`
- **Vérification** : lancer les tests en fin de session et bloquer si ils échouent
- **Notifications** : alerte Slack ou desktop quand une longue tâche se termine

> **Principe :** ce qui doit arriver à chaque fois sans exception → une automatisation déterministe. Ce qui est une préférence → le fichier de configuration.

---

## 11. Les modules d'instructions réutilisables

La plupart des CLI agentiques permettent de définir des ensembles d'instructions spécialisées qui ne sont chargés qu'à la demande, quand ils sont pertinents.

### Le problème qu'ils résolvent

Votre checklist de revue de PR n'a pas besoin d'être en mémoire quand vous déboguez un problème de performance. Les modules d'instructions réutilisables sont chargés **uniquement quand le contexte le justifie**, préservant ainsi l'espace de la fenêtre de contexte.

### Équivalents selon les outils

| Outil | Mécanisme |
|-------|-----------|
| Claude Code | Skills (`SKILL.md`) |
| Cursor | Rules by context |
| OpenCode | Instructions personnalisées |

### Bonnes pratiques de conception

- **La description est cruciale** — c'est ce que l'agent lit pour décider si le module est pertinent. Une description vague = un module qui ne se déclenche jamais.
- **Gardez-les focalisés** — un module = un domaine d'expertise. Évitez les modules "fourre-tout".
- **Partage via git** — versionner les modules dans le repo permet à toute l'équipe d'en bénéficier.

---

## 12. Les agents parallèles isolés

Un agent parallèle est une instance de l'agent qui tourne avec son propre contexte isolé. Il reçoit une tâche, la traite, et retourne uniquement son résultat — sans polluer votre contexte principal.

### Quand les utiliser

| Cas d'usage | Pourquoi un agent parallèle |
|-------------|---------------------------|
| Exploration de codebase | Lit des dizaines de fichiers sans consommer votre contexte |
| Revue de code | Contexte frais = pas de biais envers le code qu'il vient d'écrire |
| Vérification | Un agent différent de celui qui a implémenté évalue le résultat |
| Tâches indépendantes | Plusieurs workstreams en parallèle |

### Équivalents selon les outils

| Outil | Mécanisme |
|-------|-----------|
| Claude Code | Subagents (`.claude/agents/`) |
| OpenCode | Agents spécialisés |
| Cursor | Composer agents |

---

## 13. Les connecteurs d'outils externes

La plupart des CLI agentiques modernes supportent un standard de connexion à des outils et sources de données externes : Linear, Slack, GitHub, bases de données, documentation...

### MCP — Model Context Protocol

MCP est le standard ouvert émergent, adopté par de nombreux outils. Il permet à l'agent d'interagir avec des services externes sans que vous ayez à copier-coller les données.

```bash
# Exemple générique d'ajout d'un connecteur
<outil> mcp add <nom-du-serveur>
```

### Portée et partage

La plupart des outils permettent de définir les connecteurs à plusieurs niveaux :

| Portée | Partage |
|--------|---------|
| Personnel | Vous uniquement |
| Global utilisateur | Tous vos projets |
| Projet (fichier versionné) | Toute l'équipe automatiquement |

> Pour partager les connecteurs avec l'équipe : committez le fichier de configuration MCP à la racine du repo.

### Attention au contexte

Chaque connecteur activé charge ses définitions d'outils dans le contexte, même quand vous ne l'utilisez pas. Si un outil a un équivalent CLI (`gh`, `aws`...), le CLI est souvent plus économe en contexte.

---

## Partie 5 — Maîtrise avancée

## 14. Laisser l'agent vous interviewer

> Pour les grandes fonctionnalités, laissez l'agent vous interroger en premier — il pose des questions sur ce que vous n'avez peut-être pas encore considéré.

Au lieu d'écrire une spec complète vous-même, commencez par une description minimale et laissez l'agent extraire les besoins par des questions structurées.

```
I want to build [brief description]. Interview me in detail.

Ask about technical implementation, UI/UX, edge cases, concerns, and tradeoffs.
Don't ask obvious questions — dig into the hard parts I might not have considered.

Keep interviewing until we've covered everything, then write a complete spec to SPEC.md.
```

Une fois la spec complète, **commencez une nouvelle session** pour l'implémenter. La nouvelle session a un contexte propre entièrement focalisé sur l'implémentation.

**Ce qui fait une bonne spec :** elle nomme les fichiers et interfaces concernés, dit explicitement ce qui est hors périmètre, et se termine par une étape de vérification bout-en-bout.

---

## 15. Gérer sa session activement

### Corriger tôt et souvent

Les meilleurs résultats viennent de boucles de feedback serrées. Corrigez l'agent dès que vous remarquez qu'il part dans la mauvaise direction.

| Action | Quand |
|--------|-------|
| **Stopper** l'agent en cours d'action | Il fait la mauvaise chose |
| **Annuler** les modifications et rediriger | Mauvaise approche, essayer autrement |
| **Réinitialiser** le contexte | Nouvelle tâche non liée, ou après deux corrections échouées |

> **Règle :** si vous avez corrigé l'agent plus de deux fois sur le même problème, réinitialisez et écrivez un meilleur prompt initial qui intègre ce que vous avez appris. Une session propre avec un meilleur prompt surpasse presque toujours une longue session avec des corrections accumulées.

### Checkpoints et récupération

La plupart des CLI agentiques créent automatiquement des points de sauvegarde avant chaque modification. Cela vous permet de :

- **Revenir en arrière** sur les fichiers modifiés sans toucher à la conversation
- **Reprendre** une session interrompue sans re-expliquer le contexte
- **Expérimenter** sans risque — essayez quelque chose de risqué, annulez si ça ne marche pas

> Les checkpoints ne suivent que les modifications faites *par l'agent*, pas les processus externes. Ce n'est pas un substitut à git.

### Reprendre des sessions

La plupart des outils sauvegardent les conversations localement. Vous n'avez pas à ré-expliquer le contexte quand vous reprenez une tâche. Donnez des noms descriptifs à vos sessions (`migration-oauth`, `refacto-auth`…) pour les retrouver facilement.

---

## 16. Automatiser et passer à l'échelle

### Mode non-interactif

La plupart des CLI agentiques proposent un mode headless pour s'intégrer dans des pipelines CI/CD, des pre-commit hooks, ou des scripts.

```bash
# Principe général (syntaxe variable selon l'outil)
<outil> --non-interactive "votre prompt" --output-format json
```

Utilisez ce mode pour :
- Analyser automatiquement du code à chaque PR
- Lancer des migrations en batch
- Intégrer l'agent dans vos pipelines existants

### Sessions parallèles

Pour accélérer le développement, plusieurs approches :

| Approche | Pour |
|----------|------|
| **Worktrees git** | Fonctionnalités parallèles dans des checkouts isolés — les modifications ne se chevauchent pas |
| **Sessions multiples** | Workstreams indépendants gérés visuellement |
| **Agent teams** | Coordination automatisée de plusieurs agents avec partage de tâches |

### Fan-out à grande échelle

Pour les grandes migrations ou analyses, distribuez le travail en invocations parallèles :

1. **Générez une liste de fichiers** à traiter
2. **Écrivez une boucle** qui invoque l'agent en mode non-interactif pour chaque fichier
3. **Testez sur 2-3 fichiers**, affinez le prompt, puis lancez à l'échelle

```bash
# Principe (adapter la syntaxe à votre outil)
for file in $(cat files.txt); do
  <outil> "Migrate $file from X to Y. Return OK or FAIL."
done
```

---

## 17. Les patterns d'échec courants

Reconnaître ces patterns tôt fait gagner beaucoup de temps. Ils sont universels — ils s'appliquent quel que soit l'outil utilisé.

### La session "tout dans le même sac"

Vous commencez avec une tâche, posez une question non liée, revenez à la première tâche. Le contexte est rempli d'informations non pertinentes et la qualité des réponses baisse.

**Solution :** réinitialisez le contexte entre les tâches non liées.

### La correction en boucle

L'agent fait quelque chose de faux, vous corrigez, c'est toujours faux, vous recorrigez. Le contexte est pollué par des approches échouées et l'agent continue d'essayer des variations de la même erreur.

**Solution :** après deux corrections échouées, réinitialisez et écrivez un meilleur prompt initial qui intègre ce que vous avez appris.

### Le fichier de configuration surchargé

Le fichier de configuration est trop long, les règles importantes se noient dans le bruit. L'agent en ignore la moitié.

**Solution :** élagage impitoyable. Si l'agent fait déjà quelque chose correctement sans l'instruction, supprimez-la. Convertissez les règles critiques en automatisations déterministes.

### L'écart "faire confiance puis vérifier"

L'agent produit une implémentation qui semble plausible mais ne gère pas les cas limites — et vous la livrez sans vérifier.

**Solution :** fournissez toujours une vérification (tests, scripts, captures d'écran). Si vous ne pouvez pas vérifier, ne livrez pas.

### L'exploration infinie

Vous demandez à l'agent "d'explorer" quelque chose sans cadrer la recherche. L'agent lit des centaines de fichiers, remplissant le contexte avec un travail exploratoire que vous n'aviez pas besoin de voir.

**Solution :** cadrez les explorations étroitement, ou déléguez-les à un agent parallèle pour que l'exploration ne consomme pas votre contexte principal.

### Le plan ignoré

Vous validez un plan, puis laissez l'agent travailler sans vérification intermédiaire. En fin de session, l'implémentation s'est éloignée du plan sans que vous vous en rendiez compte.

**Solution :** donnez à l'agent le plan comme référence à chaque étape importante. Demandez-lui de vérifier sa conformité avant de passer à l'étape suivante.

---

## Récapitulatif

| Concept | À retenir |
|---------|-----------|
| **Agent vs assistant** | L'agent agit directement sur votre environnement — pas de copier-coller |
| **Boucle agentique** | Collecte → Action → Vérification → recommence si nécessaire |
| **Fenêtre de contexte** | La ressource centrale — tout le reste en découle |
| **Workflow principal** | Explore → Plan → Code → Commit |
| **Bons prompts** | Précis, avec contexte riche, critère de succès, références de fichiers |
| **Vérification** | Donnez à l'agent un test à lancer — preuves, pas assertions |
| **Fichier de config** | Court, actionnable, pour ce que l'agent ne peut pas deviner |
| **Automatisations** | Pour les règles critiques — déterministes, sans exception |
| **Agents parallèles** | Contexte isolé pour déléguer sans polluer la session principale |
| **Connecteurs** | Connexion aux outils externes — attention au contexte chargé |
| **Modules réutilisables** | Expertise à la demande — chargée uniquement quand pertinente |
| **Checkpoints** | Expérimentez sans risque — annulez si besoin |
| **Sessions** | Nommez, reprenez, ne ré-expliquez pas |
| **Scale** | Mode headless en boucle, sessions parallèles, pattern Writer/Reviewer |

---

*Pour aller plus loin, consultez la documentation spécifique à votre outil :*
- *[Claude Code](../../anthropic/claude-code/fr/parcours-complet.md)*
- *[OpenCode](../../opencode/)*
