---
title: "Claude Code — Ce qui change cette semaine (v2.1.178 → v2.1.195)"
date: 2026-06-29
period: "Semaine W5 · v2.1.178 → v2.1.195"
sources:
  - https://docs.claude.com/en/docs/claude-code/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
lang: fr
---

# Claude Code — Ce qui change cette semaine

> Période : semaine W5 · du 15 au 26 juin 2026 · Versions v2.1.178 → v2.1.195 (11 releases)
> Sources : [docs.claude.com](https://docs.claude.com/en/docs/claude-code/changelog) · [CHANGELOG.md GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
> Généré le 2026-06-29

---

## Ce qui compte vraiment

### Interdire une action selon ses paramètres, pas seulement l'outil — v2.1.178

Jusqu'ici, une règle de permission disait « autorise ou bloque cet outil ». Désormais elle peut viser un paramètre précis : un modèle, un argument, une valeur.

> *"Added `Tool(param:value)` syntax for permission rules to match a tool's input parameters (with `*` wildcard), e.g. `Agent(model:opus)` to block Opus subagents"*

**Ce que ça change concrètement** : `Agent(model:opus)` empêche tout sous-agent de tourner sur Opus, même si l'outil Agent reste autorisé. C'est plus fin, et c'est utile dès que vous laissez des agents se lancer entre eux. Pour vous en servir, ajoutez la règle dans votre `permissions` (allow / deny / ask) comme un pattern classique. La v2.1.186 corrige au passage l'application des règles `Agent(type)` et `Agent(x,y)` sur les sous-agents nommés — elles étaient ignorées, elles ne le sont plus.

---

### Couper les commandes sandboxées de vos secrets — v2.1.187

Si vous faites tourner des commandes dans le sandbox, un nouveau réglage les empêche de lire vos fichiers de credentials et vos variables d'environnement sensibles.

> *"Added `sandbox.credentials` setting to block sandboxed commands from reading credential files and secret environment variables"*

**Ce que ça change concrètement** : c'est exactement le réglage qu'on oublie d'activer jusqu'au jour où on en aurait eu besoin. Activez-le par défaut ; vous le réactiverez au cas par cas si une commande légitime a vraiment besoin d'un secret.

---

### Un effet de bord de télémétrie à surveiller avant de monter de version — v2.1.193

Si vous journalisiez déjà les prompts via OpenTelemetry, la mise à jour vous fait aussi exporter le texte des réponses du modèle — sans rien changer dans votre config.

> *"Added `claude_code.assistant_response` OpenTelemetry log event… when that var is unset it follows `OTEL_LOG_USER_PROMPTS`, so deployments that already log prompt content will start receiving response content on upgrade — set `OTEL_LOG_ASSISTANT_RESPONSES=0` to keep prompts-only"*

**Ce que ça change concrètement** : ce n'est pas un bug, c'est un héritage de réglage. Si l'export des réponses n'est pas voulu, posez `OTEL_LOG_ASSISTANT_RESPONSES=0` avant de monter de version. Vérifiez-le.

---

## Améliorations notables

**L'auto-mode plus prudent par défaut (v2.1.183)** — il bloque les commandes destructrices que vous n'avez pas demandées : `git reset --hard`, `git checkout -- .`, `git clean -fd`, `git stash drop`, et `terraform destroy` / `pulumi destroy` / `cdk destroy` sauf si vous avez explicitement visé la stack concernée. `git commit --amend` est bloqué si le commit n'a pas été fait par l'agent dans la session. La même version avertit quand le modèle demandé est déprécié ou redirigé.

**Vous voyez pourquoi une action a été refusée (v2.1.193)** — les raisons de refus de l'auto-mode sont ajoutées au transcript, au toast et à l'onglet « refusés récemment » de `/permissions`. Plus de refus muet.

**Configurer en une ligne (v2.1.181)** — `/config key=value` règle n'importe quel paramètre depuis le prompt (par exemple `/config thinking=false`), en interactif, en `-p` et en Remote Control. v2.1.183 ajoute `/config --help` et change le comportement du toggle (Entrée et Espace modifient, Échap enregistre et ferme).

---

## Corrections importantes

**Les matchers de hooks se comportent enfin comme on les lit (v2.1.195 / v2.1.191)** — deux corrections à connaître si vous utilisez des hooks. v2.1.195 : les identifiants à tiret (`code-reviewer`, `mcp__brave-search`) ne matchent plus par sous-chaîne mais en correspondance exacte — pour couvrir tous les outils d'un serveur MCP à tiret, écrivez désormais `mcp__brave-search__.*`. Si un de vos hooks comptait sur l'ancien comportement, il ne se déclenche peut-être plus : relisez vos matchers. v2.1.191 : les matchers séparés par virgule (`"Bash,PowerShell"`) ne se déclenchaient jamais en silence — corrigé.

**Arrêter un agent de fond, c'est maintenant définitif (v2.1.191 / v2.1.195)** — un agent stoppé depuis le panneau des tâches ne « ressuscite » plus (v2.1.191). La v2.1.195 corrige aussi des jobs de fond qui pouvaient disparaître ou perdre leurs données quand ils étaient écrits par une version plus récente de Claude Code — utile si vous jonglez entre plusieurs versions installées.

**Une rafale de stabilité** — réponses partielles préservées sur coupure réseau (v2.1.179), fichiers 0-octet sur lecteurs réseau/cloud corrigés (v2.1.181), collage CJK/coréen illisible corrigé (v2.1.187/181), dictée vocale sur macOS et langues sans espaces (v2.1.195), et CPU en streaming réduit d'environ 37 % (v2.1.191).

---

## En bref (mineur / cosmétique)

- v2.1.181 : runtime Bun bundlé passé en 1.4 ; streaming des longs paragraphes ligne par ligne.
- v2.1.191 : `/rewind` peut reprendre une conversation d'avant un `/clear`.
- v2.1.186 : `claude mcp login <name>` / `claude mcp logout <name>` pour authentifier un serveur MCP sans passer par `/mcp` (option `--no-browser` pour SSH).
- v2.1.187 : support de la souris dans les menus de sélection en fullscreen ; restrictions de modèle configurées par l'organisation affichées dans le sélecteur.
- v2.1.181 : `CLAUDE_CLIENT_PRESENCE_FILE` pour couper les notifications push mobiles quand vous êtes à la machine.
- v2.1.190 : « corrections de bugs et améliorations de fiabilité » (sans détail).

---

**Pour aller plus loin** — Cette semaine fait passer les permissions du niveau « outil » au niveau « paramètre » (`Agent(model:opus)`, v2.1.178) et ajoute `sandbox.credentials` (v2.1.187) pour couper les commandes sandboxées de vos secrets. Comment en faire une posture durable de moindre privilège (deny-by-default, écritures sensibles, fenêtre de versions, allowlist de modèles) : notre guide [Harden your own Claude Code permissions: go deny-by-default](../resources/guides/harden-claude-code-permissions.md).

---

*Veille générée à partir du changelog officiel Anthropic.*
*Les interprétations et cas d'usage sont explicitement indiqués comme tels.*
