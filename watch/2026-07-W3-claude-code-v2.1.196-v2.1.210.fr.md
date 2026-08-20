---
title: "Claude Code — Ce qui change cette semaine (v2.1.196 → v2.1.210)"
date: 2026-07-15
period: "Semaine W3 · v2.1.196 → v2.1.210"
sources:
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
lang: fr
---

# Claude Code — Ce qui change cette semaine

> Période : semaine W3 (juillet 2026) · Versions v2.1.196 → v2.1.210 (15 versions)
> Sources : [CHANGELOG.md GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [code.claude.com](https://code.claude.com/docs/en/changelog)
> Généré le 2026-07-15

> **Statut de stabilité — non confirmé ce run.** Les dist-tags npm renvoient des numéros (`stable 2.1.142` / `latest 2.1.150`) inférieurs aux versions déjà publiées dans le changelog officiel — ils sont donc désynchronisés cette semaine. Par prudence, on n'appose aucun badge « stable / en préparation » sur les items ci-dessous : vérifiez `claude --version` et votre canal de mise à jour avant de déployer en production.

---

## Ce qui compte vraiment

### Sonnet 5 devient le modèle par défaut — v2.1.197

Dès la mise à jour, vos sessions tournent sur Sonnet 5, avec une fenêtre de contexte native de 1M tokens.

> *"Introducing Claude Sonnet 5: now the default model in Claude Code, with a native 1M-token context window and promotional pricing of $2/$10 per Mtok through August 31. Update to version 2.1.197 for access."*

**Ce que ça change concrètement** : à partir de 2.1.197, vos sessions utilisent Sonnet 5 sauf si vous avez explicitement fixé un autre modèle. Le contexte natif passe à 1M tokens — vous pouvez lui donner beaucoup plus de code d'un coup sans le tronquer, et le prix promotionnel court jusqu'au 31 août. Sur Bedrock, Vertex et Claude Platform on AWS, c'est Opus 4.8 qui devient le défaut (v2.1.207). Vérifiez ce que `/model` affiche après mise à jour : si vous comptiez sur un défaut implicite, il a peut-être bougé.

---

### Le mode de permission « par défaut » s'appelle maintenant « Manual » — v2.1.200

C'est le même mode qu'avant — celui où Claude vous demande avant chaque action sensible — mais renommé pour dire clairement ce qu'il fait.

> *"Changed the "default" permission mode to "Manual" across the CLI, `--help`, VS Code, and JetBrains; `--permission-mode manual` and `"defaultMode": "manual"` are accepted alongside `default`"*

**Ce que ça change concrètement** : vos configs existantes ne cassent pas, `default` reste accepté. Si vous scriptez ou documentez des workflows, adoptez `manual` : c'est le terme qui restera. Un badge gris ⏸ apparaît aussi dans le pied de page quand ce mode est actif (v2.1.202), pour qu'on sache toujours où on en est.

---

### L'auto-mode devient disponible sans opt-in sur les providers cloud — v2.1.207

Sur Bedrock, Vertex et Foundry, l'auto-mode (Claude décide seul quelles commandes sont sûres) est là par défaut — et il se durcit.

> *"Auto mode is now available without `CLAUDE_CODE_ENABLE_AUTO_MODE` opt-in on Bedrock, Vertex AI, and Foundry; disable via `disableAutoMode` in settings"*

**Ce que ça change concrètement** : pour le couper, posez `disableAutoMode` dans vos settings. Deux garde-fous notables : les notifications de tâches de fond énoncent explicitement qu'aucune intervention humaine n'a eu lieu, pour empêcher qu'une fausse approbation glissée dans le transcript soit exécutée (v2.1.205) ; et l'auto-mode demande maintenant avant de lancer `rm -rf` sur une variable qu'il ne peut pas résoudre (v2.1.205). Détail qui compte : l'auto-mode ne lit plus sa config depuis le `.claude/settings.local.json` du dépôt — seul `~/.claude/settings.json` (ou les settings managés) est pris en compte (v2.1.207). Si vous croyiez le configurer par projet, ce n'est plus le cas.

---

### Les subagents tournent en arrière-plan par défaut, et vont jusqu'à la PR — v2.1.198

Quand Claude délègue à un sous-agent, il ne vous fait plus attendre — et un agent de fond qui termine un travail de code va désormais jusqu'à ouvrir une PR.

> *"Subagents now run in the background by default, so Claude keeps working while they run and is notified when they finish"*
>
> *"Background agents launched from `claude agents` now commit, push, and open a draft PR when they finish code work in a worktree, instead of stopping to ask"*

**Ce que ça change concrètement** : Claude continue et vous notifie à la fin, au lieu de vous faire patienter. Et un agent de fond ne s'arrête plus pour demander : il committe, pousse et ouvre une draft PR. Gain de fluidité réel, mais surveillez votre liste `claude agents` : il s'y passe désormais plus de choses sans vous.

---

## Améliorations notables

**Sortie structurée plus fiable — et plus stricte (v2.1.205)** — `--json-schema` produisait parfois une sortie non structurée en silence quand le schéma était invalide : c'est corrigé, et les schémas qui utilisent le mot-clé `format` sont désormais rejetés explicitement. Si vous générez du JSON structuré via un schéma, revérifiez qu'il ne s'appuie pas sur `format` — sinon il faut le retravailler.

**Beaucoup moins de mémoire et de latence sur les longues sessions (v2.1.208)** — une release de performance à elle seule : les matchers de règles sont compilés une fois et mis en cache (fin des ralentissements de plusieurs secondes quand vous avez beaucoup de règles deny/ask), l'assemblage des outils MCP est caché (jusqu'à 7× plus rapide en print/SDK), plusieurs fuites mémoire sont bouchées, et la taille des transcripts d'édition chute jusqu'à 79×. Si vos sessions longues ramaient, la mise à jour se sent.

**`AskUserQuestion` ne répond plus à votre place (v2.1.200)** — les dialogues de question n'auto-continuent plus par défaut ; vous pouvez réactiver un délai d'inactivité via `/config`. Une question posée attend maintenant vraiment votre réponse.

---

## Corrections importantes

**Les worktrees isolés le sont vraiment (v2.1.210)** — correction de subagents en `isolation: 'worktree'` qui pouvaient exécuter des commandes git modifiant le checkout principal au lieu de leur propre worktree — un vrai risque de pollution du dépôt. La même version libère les verrous `git worktree` laissés derrière par une session tuée (un balayage périodique les relâche quand le processus propriétaire a disparu). Si vous utilisiez des agents de fond sur worktrees, c'est un correctif à prendre.

**Les agents de fond survivent mieux aux mises à jour et aux redémarrages (v2.1.200 / 205 / 208 / 210)** — un daemon plus ancien ne peut plus reprendre la main sur des workers lancés par une version plus récente ; les réponses tapées à un agent de fond ne sont plus perdues si la livraison échoue (elles sont sauvegardées et délivrées au redémarrage, v2.1.208) ; et `claude attach` attend que le daemon se stabilise au lieu d'échouer en « job not found » pendant une transition (v2.1.210).

**Sécurité MCP : fin de l'auto-approbation par un dépôt (v2.1.196)** — `claude mcp list`/`get` ne lancent plus les serveurs `.mcp.json` qu'un dépôt s'est auto-approuvés via un `.claude/settings.json` committé ; un workspace non fiable affiche `⏸ Pending approval`. Si vous ouvrez du code inconnu, ses serveurs MCP ne démarrent plus dans votre dos.

---

## En bref (mineur / cosmétique)

- v2.1.198 : **Claude in Chrome** passe en disponibilité générale ; nouveau skill `/dataviz` (design de graphiques + validateur de palette).
- v2.1.208 : mode **lecteur d'écran** (`claude --ax-screen-reader` / `CLAUDE_AX_SCREEN_READER=1` / `"axScreenReader": true`) ; réglage `vimInsertModeRemaps` (mapper `jj` → Échap en mode vim) ; `CLAUDE_CODE_PROCESS_WRAPPER` pour imposer un lanceur d'entreprise.
- v2.1.196 : modèle par défaut d'organisation configurable par l'admin, affiché « Org default » dans `/model` ; noms de session lisibles au démarrage.
- v2.1.202 : attributs OpenTelemetry `workflow.run_id` / `workflow.name` sur les agents lancés par un workflow ; réglage « Dynamic workflow size » dans `/config`.
- v2.1.206 : `/commit-push-pr` auto-autorise `git push` vers le remote configuré (`remote.pushDefault`) ; suggestions de chemins de dossier dans `/cd`.
- v2.1.210 : compteur de temps écoulé en direct sur les appels d'outils longs ; l'écriture d'un index MEMORY.md au-delà de la limite de lecture renvoie une erreur explicite au lieu d'une troncature silencieuse.
- v2.1.209 : correctif — `/model` et autres dialogues n'étaient plus bloqués dans les sessions `claude agents` de fond.

---

**Pour aller plus loin** — Cette semaine, `claude mcp list`/`get` cessent de démarrer les serveurs MCP qu'un dépôt s'est auto-approuvés via un `.claude/settings.json` committé (v2.1.196) : le CLI applique nativement le réflexe qu'on recommande depuis le départ — traiter la configuration d'un repo cloné comme du code non fiable. Le réflexe durable, avec une checklist gratuite : notre guide [Treat a cloned repo's Claude Code setup as untrusted code](../resources/guides/untrusted-repo-config.md).

---

*Veille générée à partir du changelog officiel Anthropic.*
*Les interprétations et cas d'usage sont explicitement indiqués comme tels.*
