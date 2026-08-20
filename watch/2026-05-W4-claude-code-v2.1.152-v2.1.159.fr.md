---
title: "Claude Code — Ce qui change cette semaine (v2.1.152 → v2.1.159)"
date: 2026-06-01
period: "Semaine W4 · v2.1.152 → v2.1.159"
sources:
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
lang: fr
---

# Claude Code — Ce qui change cette semaine

> Période : semaine W4 (4ᵉ semaine de mai) · 27 au 31 mai 2026 · Versions v2.1.152 → v2.1.159
> Sources : [CHANGELOG.md GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [code.claude.com](https://code.claude.com/docs/en/changelog)
> Généré le 2026-06-01

---

## Ce qui compte vraiment

### Opus 4.8 est disponible — v2.1.154 (28 mai 2026)

Le nouveau modèle phare d'Anthropic est désormais accessible dans Claude Code. Il devient le modèle par défaut pour les tâches à effort élevé (`/effort xhigh`). Un mode *fast* est proposé à 2× le tarif standard pour 2,5× la vitesse par rapport au mode normal.

> *"Opus 4.8 is here! Now defaults to high effort · /effort xhigh for your hardest tasks"*
> *"Fast mode on Opus 4.8 is now available at a fraction of its previous cost: 2x the standard rate for 2.5x the speed"*

**Ce que ça change concrètement** : si vous utilisez Claude Code en mode abonnement Max ou Team, Opus 4.8 devient accessible via `/model`. Le slider `/effort` gagne une nouvelle position utile pour les tâches longues et complexes (refactorings massifs, analyses de base de code entière).

**Note** : `CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` est déprécié et sera supprimé le 01/06. Si vous l'utilisiez, passez à `/model claude-opus-4-6[1m]` puis `/fast on`.

---

### Dynamic workflows : orchestration de masse en background — v2.1.154 (28 mai 2026)

Claude Code peut maintenant décomposer une tâche complexe et orchestrer lui-même des dizaines à centaines d'agents en arrière-plan. Commande : `/workflows` pour visualiser les runs en cours.

> *"Introducing dynamic workflows: ask Claude to create a workflow and it orchestrates work across tens to hundreds of agents in the background, so you can take on larger, more complex tasks. Run `/workflows` to view your runs"*

**Ce que ça change concrètement** : vous pouvez confier à Claude Code une tâche ambitieuse — migrer une API, générer une suite de tests sur l'ensemble d'un projet, auditer une documentation — et le laisser travailler sans rester présent. Le résultat est consolidé quand les agents se terminent. Complément utile : `claude agents` affiche maintenant les workflows aux côtés des sessions classiques.

**Lancer un agent shell en background** (même version) : `! <commande>` dans `claude agents` ouvre une session background attachable/détachable, ou `claude --bg --exec '<commande>'` en ligne de commande.

---

### Nouveau hook `MessageDisplay` — personnaliser l'affichage des réponses — v2.1.152 (27 mai 2026)

Un nouvel événement hook est disponible : `MessageDisplay`. Il permet à un script de transformer ou masquer le texte d'une réponse assistant *au moment où elle s'affiche* dans le terminal.

> *"Added a `MessageDisplay` hook event that lets hooks transform or hide assistant message text as it is displayed"*

**Ce que ça change concrètement** : cas d'usage typiques — filtrer du contenu sensible avant affichage, ajouter un préfixe contextuel à chaque réponse, masquer des blocs verbeux automatiquement. Ce hook ne modifie pas la réponse stockée dans l'historique, uniquement ce que vous voyez.

---

### Les plugins se chargent depuis `.claude/skills` sans marketplace — v2.1.157 (29 mai 2026)

Les plugins placés dans `.claude/skills` d'un projet sont maintenant chargés automatiquement, sans nécessiter d'installation via une marketplace. La commande `claude plugin init <nom>` crée le scaffold d'un nouveau plugin directement dans `.claude/skills/<nom>/`.

> *"Plugins in `.claude/skills` directories are now automatically loaded, no marketplace required"*
> *"Added `claude plugin init <name>` to scaffold a new plugin in `.claude/skills`"*

**Ce que ça change concrètement** : vous pouvez créer et partager des plugins Claude Code en versionant simplement un dossier dans votre dépôt. Fini les étapes d'installation : un `git clone` suffit pour qu'un collègue bénéficie du même environnement. L'autocomplete de `/plugin` couvre maintenant les sous-commandes, les noms de plugins installés, et les plugins des marketplaces connues.

---

## Améliorations notables

**`SessionStart` enrichi (v2.1.152)** — les hooks `SessionStart` peuvent maintenant retourner `reloadSkills: true` pour que les skills installés par le hook soient disponibles *dans la même session*, sans relancer Claude Code. Ils peuvent aussi définir le titre de session via `hookSpecificOutput.sessionTitle`.

**`disallowed-tools` dans le frontmatter des skills (v2.1.152)** — en ajoutant `disallowed-tools:` dans le frontmatter YAML d'un skill ou d'une slash command, vous retirez des outils spécifiques du modèle pendant l'exécution de ce skill. Utile pour contraindre un skill à ne pas utiliser Bash, par exemple.

**`/reload-skills` (v2.1.152)** — re-scanne les répertoires de skills sans redémarrer la session. Pratique pendant le développement d'un skill.

**`/model` sauvegarde désormais le choix par défaut (v2.1.153)** — appuyer sur `Entrée` dans le sélecteur de modèle définit le modèle pour les prochaines sessions (comportement aligné avec les extensions IDE). Appuyer sur `s` restreint le changement à la session courante uniquement. Si vous aviez personnalisé le raccourci `modelPicker:setAsDefault`, renommez-le en `modelPicker:thisSessionOnly` dans `keybindings.json`.

**`/code-review --fix` applique les corrections (v2.1.152)** — la commande effectue maintenant la review *et* applique les suggestions (réutilisation, simplification, efficacité) directement dans votre arbre de travail. `/simplify` devient un alias de `/code-review --fix`.

**Auto mode sans opt-in (v2.1.152)** — le mode `auto` ne nécessite plus de consentement préalable. Il est disponible directement dans le sélecteur de mode.

**`agent` dans `settings.json` (v2.1.154/v2.1.157)** — un nouveau champ `agent` de premier niveau dans `settings.json` permet de définir l'agent par défaut pour les sessions dispatchées via `claude agents`. Surchargeable avec `--agent <nom>`.

**Lean system prompt par défaut (v2.1.154)** — pour Opus 4.8 et les modèles futurs, le prompt système allégé est activé par défaut (les modèles Haiku, Sonnet et Opus 4.7 et antérieurs conservent le comportement précédent).

**MCP : `CLAUDE_CODE_SESSION_ID` pour les subprocesses stdio (v2.1.154)** — les serveurs MCP stdio reçoivent maintenant `CLAUDE_CODE_SESSION_ID` et `CLAUDECODE=1` dans leur environnement, permettant aux serveurs MCP de savoir depuis quelle session Claude Code ils sont appelés.

**Auto mode sur Bedrock, Vertex et Foundry (v2.1.158)** — disponible pour Opus 4.7 et 4.8, via `CLAUDE_CODE_ENABLE_AUTO_MODE=1`.

---

## Corrections importantes

**Fix : blocage CLI en mode stream-json (v2.1.153)** — un bug pouvait empêcher le CLI de se terminer quand stdin était fermé sans EOF en mode `--output-format stream-json`, laissant un marqueur de session obsolète. Corrigé.

**Fix : l'Agent tool écrivait dans un worktree temporaire (v2.1.153)** — `Agent` avec `subagent_type: 'claude'` pouvait écrire dans un worktree temporaire non documenté, ce qui causait la perte silencieuse d'outputs dans des chemins gitignorés. Corrigé.

**Fix : serveurs MCP stateful en reconnexion infinie (v2.1.153)** — les serveurs MCP stateful sans flux SSE GET optionnel se reconnectaient en boucle sur `tools/list` (régression v2.1.147). Corrigé.

**Fix : subagents ignoraient `--strict-mcp-config` (v2.1.153)** — les subagents définis par frontmatter ignoraient les politiques MCP gérées. Corrigé ; les serveurs MCP bloqués génèrent maintenant un avertissement visible.

**Fix : orphelins de worktrees après le sweep 30 jours (v2.1.157)** — les worktrees d'agents background dans `.claude/worktrees/` pouvaient être orphelinisés après le nettoyage automatique mensuel. Corrigé.

**Fix : sessions background re-attachées sans date correcte (v2.1.157)** — après une mise en veille/réveil, les sessions réattachées ne transmettaient pas la bonne date au modèle. Corrigé.

---

## En bref (mineur / cosmétique)

- v2.1.159 : améliorations internes uniquement, aucun changement visible.
- v2.1.156 : fix stabilité thinking blocks Opus 4.8.
- v2.1.157 : `EnterWorktree` peut changer de worktree géré en cours de session.
- v2.1.157 : le statut de claim "Feature of the Week" apparaît en notification (plus en ligne au-dessus du prompt).
- v2.1.157 : bannière "bash sera sandboxé" supprimée du démarrage ; l'info reste disponible via `/status`.
- v2.1.153 : `claude doctor` affiche le résultat de la dernière tentative de mise à jour.
- v2.1.153 : Windows — si une mise à jour échoue, Claude Code restaure l'exécutable précédent et indique comment récupérer.
- v2.1.152 : `pluginSuggestionMarketplaces` — nouveau champ managed settings pour les admins (Enterprise).
- v2.1.154 : `←←` pour ouvrir la vue agents fonctionne maintenant sur Bedrock, Vertex, Foundry.

---

**Pour aller plus loin** — Cette semaine touche à la confiance qu'on accorde à la configuration d'un projet (plugins désormais auto-chargés depuis `.claude/skills`). Notre guide associé revient sur le bon réflexe : [Treat a cloned repo's Claude Code setup as untrusted code](../resources/guides/untrusted-repo-config.md).

---

*Veille générée à partir du changelog officiel Anthropic.*
*Les interprétations et cas d'usage sont explicitement indiqués comme tels.*
