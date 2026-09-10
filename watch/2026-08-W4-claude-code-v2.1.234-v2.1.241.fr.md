---
title: "Claude Code : ce qui change cette semaine (v2.1.234 → v2.1.241)"
date: 2026-09-10
period: "Semaine W4 · v2.1.234 → v2.1.241"
sources:
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
  - https://registry.npmjs.org/-/package/@anthropic-ai/claude-code/dist-tags
lang: fr
---

# Claude Code : ce qui change cette semaine

> Période : semaine W4 (août 2026) · Versions v2.1.234 → v2.1.241 (8 versions, publiées du 17 au 22 août 2026)
> Sources : [CHANGELOG.md GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [code.claude.com](https://code.claude.com/docs/en/changelog) · [dist-tags npm](https://registry.npmjs.org/-/package/@anthropic-ai/claude-code/dist-tags)
> Généré le 2026-09-10

> **Statut de stabilité, vérifié le 10 septembre 2026.** Les dist-tags npm donnent `stable 2.1.236` / `latest 2.1.267`. Trois des huit versions décrites ici sont donc passées sur le canal stable : v2.1.234, v2.1.235 et v2.1.236. Les cinq autres, v2.1.237 à v2.1.241, restent au-dessus de stable et peuvent encore bouger. Deux d'entre elles, v2.1.240 et v2.1.241, ne portent au changelog qu'une seule ligne : « Bug fixes and reliability improvements ». Cette page couvre ce qui est documenté, elle ne prétend pas couvrir ce qui est publié. Vérifiez `claude --version` et votre canal avant de déployer.

---

## Ce qui compte vraiment

### Vos sessions se parlent enfin sous Windows, v2.1.239 (21 août 2026)

La messagerie entre sessions existait sur macOS et Linux. Elle arrive sous Windows, accompagnée de la petite ligne sans laquelle elle restait inutilisable.

> *"Windows: cross-session messaging is now available, so Claude Code sessions across your machines can message each other with `SendMessage` and find each other with `ListAgents`, as on macOS and Linux"*
>
> *"`ListAgents` now tells a session its own name (the one peers use to message it), and `SendMessage` to your own name says so instead of \"no agent named …\""*

**Ce que ça change concrètement** : jusqu'ici, faire tourner plusieurs sessions, c'était faire tourner plusieurs solitaires. Chacune travaillait dans son coin, et le lien entre elles, c'était vous : copier un résultat d'un terminal vers un autre, se souvenir de qui attend quoi. Une session connaît désormais son propre nom, sait qui d'autre est vivant, et peut écrire à un pair. Commencez petit : deux sessions, une tâche longue dans la première, un `SendMessage` vers la seconde à la fin. Vous verrez très vite si votre façon de travailler gagne quelque chose à ce que les agents se coordonnent, ou si vous préférez rester l'aiguilleur. Les deux réponses sont légitimes. Ce qui a changé, c'est que la question se pose.

**Note** : la pièce complémentaire, arrivée trois jours plus tôt, n'a pas suivi sur Windows.

> *"Added `notify_when_idle` to cross-session `SendMessage`: ask another Claude Code session on this machine to send one notice when it next goes idle — opt-in, one-shot, no polling (macOS and Linux)"* (v2.1.236)

La ligne d'origine porte elle-même la restriction, « (macOS and Linux) », et aucune entrée de la fenêtre ne l'étend à Windows. Sur la foi du changelog seul : sous Windows vous avez la messagerie, le réveil sur inactivité n'y est pas annoncé.

---

### Un modèle par défaut qui ne confisque pas le choix de chacun, v2.1.236 (19 août 2026)

Une variable d'environnement de plus, et une différence qui se lit en une ligne mais se paie tous les jours.

> *"Added `ANTHROPIC_DEFAULT_MODEL` environment variable: sets the model new sessions start on, while a `/model` pick still overrides it and persists across restarts (unlike `ANTHROPIC_MODEL`)"*

**Ce que ça change concrètement** : l'ancienne variable, `ANTHROPIC_MODEL`, imposait le modèle. Celle-ci pose un point de départ. C'est exactement le réglage qu'il fallait pour une équipe : vous normalisez sur quoi démarrent les nouvelles sessions, et chacun garde le droit de basculer sur autre chose pour une tâche précise, sans que ce choix soit écrasé au prochain lancement. Si vous administrez des postes, c'est cette variable qu'il faut poser, pas l'autre. Si vous travaillez seul, elle vous évite un `/model` à chaque nouveau projet.

---

### Le coût du contexte devient un sujet d'ingénierie, y compris chez Anthropic, v2.1.234 (17 août 2026)

Anthropic divise par huit le coût de chargement de son propre skill, et dit exactement comment.

> *"Reduced the context cost of loading the built-in `claude-api` skill from ~200k+ tokens to ~25k by loading reference docs on demand"*

**Ce que ça change concrètement** : c'est la démonstration la plus nette qu'on ait lue de ce qu'est un bon skill. Un skill de 200 000 tokens n'est pas un skill riche, c'est une documentation collée dans le contexte, qui se charge intégralement même quand on n'en utilise qu'une phrase. Le passage à 25 000 tokens ne retire rien : les documents de référence sont toujours là, ils s'ouvrent quand on en a besoin. Si votre propre skill pèse plusieurs dizaines de milliers de tokens, la question n'est pas « qu'est-ce que je coupe ? » mais « qu'est-ce qui n'avait pas besoin d'être là au chargement ? ». Écrivez votre `SKILL.md` comme un index, pas comme un manuel : il dit quoi faire, quand, et où trouver le détail. Le détail vit dans des fichiers voisins que le modèle ouvre lui-même.

---

### La confiance d'un dossier devient un prérequis, même en mode non interactif, v2.1.238 (20 août 2026)

Un `.mcp.json` de projet peut faire exécuter une commande. Cloner un dépôt ne suffit plus à la déclencher.

> *"MCP `headersHelper` in a project `.mcp.json`, and inline MCP servers in project or `--add-dir` agent files, now require that folder's trust dialog to have been accepted (also under `claude -p`)"*
>
> *"`claude plugin install/update` ask `[y/N]` (or pass `-y`)"*

**Ce que ça change concrètement** : le raisonnement est solide. Un `headersHelper` fabrique des en-têtes HTTP, typiquement un jeton court, en lançant une commande de votre machine. La rendre conditionnelle à une confiance explicite est la bonne décision. Ce qu'il faut retenir, c'est le « also under `claude -p` » : le mode non interactif ne contourne plus la confiance. La conséquence est directe et silencieuse. Vos scripts d'intégration continue qui lancent `claude -p` sur des dépôts jamais ouverts à la main vont démarrer avec moins de serveurs MCP que vous ne le croyez, sans erreur bruyante. Vérifiez explicitement ce qui est chargé au lieu de le supposer. Et dans un pipeline, le `-y` de `claude plugin install` n'est plus une coquetterie : sans lui, la commande attend une réponse que personne ne donnera.

---

## Améliorations notables

**Un style de sortie « Concise » livré d'origine (v2.1.237)** : *"Added a built-in \"Concise\" output style: Claude leads with results and skips preamble and narration, while doing the work just as thoroughly."* À activer sous « Output style » dans `/config`. Si vous avez déjà écrit trois lignes de CLAUDE.md pour demander à Claude d'arrêter de raconter ce qu'il s'apprête à faire, vous pouvez les supprimer.

**Les raccourcis clavier peuvent se comporter comme dans Bash (v2.1.238, v2.1.239)** : la clé `keybindingFlavor: "readline"` fait que `Ctrl+W` efface jusqu'à l'espace précédent, comme votre shell. La v2.1.239 étend le traitement à `Alt+F`, `Ctrl/Option+→`, `Alt+D` et `Ctrl+Y`, et fait de la ponctuation une frontière de mot. Le défaut, `"classic"`, ne change pas. Petit réglage, gros confort si vos doigts viennent du terminal.

**Les règles de refus en lecture ne se contournent plus par un renommage (v2.1.236, macOS)** : *"wildcard read-deny rules (e.g. `**/.env`) now take precedence inside allowed read regions, cover matched directories' contents, and can't be bypassed by renaming the denied file"*. Si vous vous appuyiez sur `**/.env` pour tenir vos secrets à l'écart, la règle fait maintenant ce que vous pensiez qu'elle faisait.

**`/goal` cesse de vous relancer toutes les demi-heures (v2.1.236, v2.1.239)** : les relances sur du travail de fond long passent à 30 minutes, puis 1 heure, puis toutes les 2 heures. À noter, une correction voisine mais antérieure : depuis la v2.1.234, un objectif *"now clears itself with a notice when a turn dies on an unrecoverable error"*, au lieu de rester armé dans le vide.

**Un correcteur orthographique dans le prompt (v2.1.235)** : la clé `spellcheck` souligne les fautes en s'appuyant sur l'`aspell`, `hunspell` ou `ispell` déjà installé sur votre machine. Optionnel, désactivé par défaut.

**Les dépôts GitLab ont enfin leur badge (v2.1.234)** : un dépôt avec un remote GitLab et un `glab` authentifié affiche `MR !N` dans le pied de page et la statusline, avec les états brouillon, en attente et vert. La v2.1.233 avait déjà ouvert `--worktree` aux URL de merge request.

---

## Corrections importantes

**Un proxy pouvait doubler votre facture API, sans rien signaler (v2.1.239)** : c'est le correctif le plus coûteux de la fenêtre.

> *"Fixed Bedrock streaming behind proxies that strip the response Content-Type header, which silently doubled billed API calls by re-running every turn non-streaming"*

**Ce que ça change concrètement** : derrière un proxy d'entreprise qui retire l'en-tête `Content-Type`, chaque tour repartait en mode non streamé, donc était facturé deux fois. Si vous êtes sur Bedrock et que vos coûts vous paraissaient élevés sans explication, vous venez peut-être de la trouver.

**Un fichier commençant par un BOM UTF-8 n'est plus ignoré en silence (v2.1.239)** : *"Fixed agents, skills, and commands whose `.md` file starts with a UTF-8 BOM being silently ignored"*. Le BOM est ce marqueur invisible que plusieurs éditeurs Windows ajoutent en tête de fichier. Un skill parfaitement écrit pouvait être purement et simplement absent, sans message d'erreur.

**Deux projets aux noms voisins ne partagent plus leurs sessions (v2.1.239)** : *"Fixed `claude -c`/resume picking up sessions from a different directory whose path differed only by characters like `_`, `-`, or `.`"*. `mon_app` et `mon-app` pouvaient retomber sur le même historique. Si vous avez un jour repris une session qui parlait d'un autre projet, ce n'était pas vous.

**Les sessions longues ne gonflent plus indéfiniment en mémoire (v2.1.238)** : *"Fixed unbounded memory growth in long interactive sessions: subagent tool results are now released once they leave the recent display window"*. Concerne directement ceux qui laissent une session ouverte toute la journée.

**Les traces OpenTelemetry ne se coupent plus en deux sur un hook (v2.1.239)** : une exécution d'outil différée par un hook `PreToolUse` reprend dans la trace du tour d'origine au lieu d'en ouvrir une nouvelle. Si vous mesurez Claude Code par OTEL et que vous utilisez des hooks, vos chiffres antérieurs sont à considérer avec prudence.

**Les hooks ne plantent plus quand le dossier de travail a disparu (v2.1.239)** : au lieu d'un `posix_spawn ENOENT`, ils s'exécutent depuis la racine du projet ou le répertoire personnel. Même version, même famille : le bac à sable Linux ne rend plus illisible un `.git/config.worktree` inexistant, ce qui cassait toute commande git en bac à sable dans les dépôts réglés avec `extensions.worktreeConfig`.

**La fermeture du vecteur NTLM se poursuit (v2.1.234)** : les lectures de fichiers distants, la restauration de session, les inclusions de CLAUDE.md, les scripts de workflow et les envois de fichiers rejettent désormais les chemins Windows en espace de noms NT (`\??\`). C'est la suite directe du correctif de la semaine précédente, appliquée cette fois aux accès fichiers qui précèdent toute approbation.

---

## En bref (mineur / cosmétique)

- v2.1.234 : `/permissions` et `/add-dir` s'ouvrent désormais pendant que Claude travaille, et un changement de règle s'applique au reste du tour en cours.
- v2.1.234 : la reprise automatique de session à la réinitialisation d'une limite d'usage claude.ai est activée, désactivable dans `/config`. Les diagnostics MCP n'impriment plus de secrets résolus : les avertissements montrent la forme `${VAR}` configurée.
- v2.1.234 : le réglage « Default teammate model » disparaît de `/config`, les coéquipiers utilisent le modèle du leader sauf mention au lancement.
- v2.1.236 : une commande slash mal orthographiée est signalée au lieu de lancer la plus proche par approximation. `/usage` affiche la ligne de crédits d'usage pour les membres Team et Enterprise.
- v2.1.238 : `Ctrl+L` et `Cmd+K` en plein écran ne font plus que repeindre, le raccourci `/clear` par double appui a disparu. `claude mcp list` et `claude mcp get` affichent les serveurs désactivés comme `⊘ Disabled` au lieu de les sonder. `claude self-hosted-runner` gagne `--defer-shutdown-max-min` et deux options d'autorisation de proxy.
- v2.1.239 : `/cost`, la statusline et `--max-budget-usd` intègrent la prime de 1,1× d'inférence US-only des espaces à résidence de données. Sur les builds Alpine et musl, collage d'image natif, presse-papiers et capture audio fonctionnent enfin.
- v2.1.236 à v2.1.239 : le renderer plein écran est proposé sur Bedrock, Vertex et Foundry, il se replie sur le renderer classique après un démarrage raté au lieu de sortir à chaque lancement, et l'invite s'arrête après trois propositions.
- v2.1.234 à v2.1.239 : confort et affichage, avec le mode vim, tmux et iTerm2, le lecteur d'écran VS Code, l'alignement du pied de page, la troncature au milieu des chemins longs, le retour à la ligne des diffs contenant des emoji, et les yeux de la mascotte Clawd.


---

**Pour aller plus loin** : le fait durable de cette fenêtre tient dans une bascule discrète. Jusqu'ici, `claude -p` n'affichait jamais de dialogue de confiance, et beaucoup en avaient déduit que le mode non interactif échappait à la question. La v2.1.238 referme la porte : un `headersHelper` déclaré dans `.mcp.json`, comme les serveurs MCP inline, exige désormais un dossier approuvé, y compris quand personne ne regarde l'écran. Un pipeline qui tournait hier peut donc se taire aujourd'hui, et un dépôt cloné ne devient pas inoffensif parce que l'exécution est automatisée. Ce qu'une approbation couvre vraiment, ce qui s'exécute avant elle, et une checklist manuelle gratuite : notre guide [Treat a cloned repo's Claude Code setup as untrusted code](../resources/guides/untrusted-repo-config.md).

---

*Veille générée à partir du changelog officiel Anthropic.*
*Les interprétations et cas d'usage sont explicitement indiqués comme tels.*
