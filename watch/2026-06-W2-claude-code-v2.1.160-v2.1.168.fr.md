---
title: "Claude Code — Ce qui change cette semaine (v2.1.160 → v2.1.168)"
date: 2026-06-08
period: "Semaine W24 · v2.1.160 → v2.1.168"
sources:
  - https://docs.claude.com/en/docs/claude-code/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
lang: fr
---

# Claude Code — Ce qui change cette semaine

> Période : semaine W24 · 2 au 6 juin 2026 · Versions v2.1.160 → v2.1.168 (8 releases)
> Sources : [CHANGELOG.md GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [docs.claude.com](https://docs.claude.com/en/docs/claude-code/changelog)
> Généré le 2026-06-08

---

## Ce qui compte vraiment

### Un modèle de repli configurable, jusqu'à trois niveaux — v2.1.166 (6 juin 2026)

Claude Code introduit un paramètre `fallbackModel` qui permet de configurer jusqu'à trois modèles de repli, essayés dans l'ordre quand le modèle principal est surchargé ou indisponible. Le drapeau `--fallback-model` s'applique désormais aussi aux sessions interactives, et non plus seulement aux exécutions non interactives.

> *"configure up to three fallback models tried in order when the primary model is overloaded or unavailable"*

**Ce que ça change concrètement** : fini la session qui échoue net quand le modèle principal est saturé — on définit une cascade de secours et le travail continue. En pratique : ajoutez `fallbackModel` dans votre `settings.json`, ou lancez avec `--fallback-model` même en session interactive.

---

### Savoir sur quoi une session est bloquée — v2.1.162 (3 juin 2026)

`claude agents --json` expose maintenant un champ `waitingFor` qui indique sur quoi une session en attente est bloquée (par exemple une demande de permission).

> *"`claude agents --json` now includes `waitingFor` showing what a waiting session is blocked on (e.g. permission prompt)"*

**Ce que ça change concrètement** : quand on pilote plusieurs agents en arrière-plan, une session « en attente » sans explication oblige à aller voir manuellement. Avec `waitingFor`, on sait immédiatement laquelle attend une permission et laquelle attend autre chose. Cas d'usage : parser la sortie de `claude agents --json` pour afficher la cause d'attente dans un tableau de bord.

---

### Gouvernance de version pour les équipes — v2.1.163 (4 juin 2026)

Deux nouveaux paramètres managés font leur apparition : `requiredMinimumVersion` et `requiredMaximumVersion`. Claude Code refuse de démarrer si sa version est hors de la plage autorisée et oriente l'utilisateur vers une version approuvée.

> *"Added `requiredMinimumVersion` and `requiredMaximumVersion` managed settings — Claude Code refuses to start if its version is outside the allowed range"*

**Ce que ça change concrètement** : les organisations peuvent imposer une fenêtre de versions homologuées — utile pour éviter qu'un poste tourne sur une version trop ancienne ou non validée. À fixer dans les *managed settings* de l'organisation.

---

## Améliorations notables

**Refus global d'outils via glob (v2.1.166)** — les règles `deny` acceptent désormais un glob en position nom d'outil : `"*"` refuse tous les outils. Les règles `allow` rejettent les globs non-MCP, et un nom d'outil inconnu dans une règle `deny` déclenche un avertissement au démarrage. Une syntaxe compacte pour une posture « refuser par défaut ».

**`/plugin list` (v2.1.163)** — nouvelle commande pour lister les plugins installés, avec les filtres `--enabled` / `--disabled`. Pratique pour faire le point sur ce qui est réellement actif.

**Hooks `Stop` et `SubagentStop` plus souples (v2.1.163)** — ces hooks peuvent renvoyer `hookSpecificOutput.additionalContext` pour donner un retour à Claude et poursuivre le tour sans être marqués comme une erreur de hook. On peut injecter du contexte sans interrompre le travail.

**Appels d'outils en parallèle plus robustes (v2.1.161)** — une commande Bash en échec n'annule plus les autres appels du même lot ; chaque outil retourne son propre résultat indépendamment.

**Écritures sensibles plus prudentes (v2.1.160)** — Claude Code demande maintenant confirmation avant d'écrire dans des fichiers de démarrage shell (`.zshenv`, `.zlogin`, `.bash_login`) et dans `~/.config/git/`. Le mode `acceptEdits` invite également avant d'écrire des fichiers de configuration d'outils de build qui accordent l'exécution de code (`.npmrc`, `.yarnrc*`, `bunfig.toml`, `.bazelrc`, `.pre-commit-config.yaml`, `.devcontainer/`).

---

## Corrections importantes

**Permissions Windows (v2.1.162)** — correction de règles de permission qui ne matchaient jamais quand elles étaient écrites avec des backslashs (`~\`, `\\server\share`) ou des variantes de casse ; les règles `deny` en lecture cachent désormais aussi les fichiers des résultats Glob/Grep.

**Chemins personnels et `$HOME` (v2.1.163)** — les règles `deny` sur des chemins du répertoire personnel (ex. `Read(~/Desktop/**)`) bloquent maintenant les commandes Bash qui référencent le chemin via `$HOME`.

**Condition de hook `if: "Bash(...)"` (v2.1.163)** — cette condition ne se déclenche plus sur toute commande Bash contenant `$()` ou `$VAR` ; le motif matche désormais aussi les commandes dans les sous-shells et les backticks.

**Régression `$TMPDIR` (v2.1.163)** — correction de commandes Bash échouant sous bazel et dans les workflows Go protégés par EDR : `$TMPDIR` était surchargé pour toutes les commandes au lieu des seules commandes en sandbox (régression introduite en v2.1.154). Corrigé.

**Précédence des règles WebFetch (v2.1.162)** — les règles de permission `WebFetch(domain:...)` explicites (deny/ask/allow) priment désormais sur l'auto-autorisation des domaines préapprouvés.

---

## En bref (mineur / cosmétique)

- v2.1.166 : durcissement de la messagerie inter-sessions — les messages relayés via `SendMessage` depuis d'autres sessions ne portent plus l'autorité de l'utilisateur (les destinataires refusent les demandes de permission relayées).
- v2.1.166 : `MAX_THINKING_TOKENS=0`, `--thinking disabled` et le bouton de réflexion par modèle désactivent le « thinking » sur les modèles qui le font par défaut (fournisseurs tiers inchangés).
- v2.1.166 : `claude update` annonce la version cible avant de télécharger ; correctifs JetBrains (flicker 2026.1+), Kitty (Shift+non-ASCII), PowerShell (validation qui traînait sur Windows).
- v2.1.162 : les timeouts MCP par serveur sous 1000 ms sont désormais ignorés (retour à `MCP_TOOL_TIMEOUT` ou au défaut) ; nombreuses améliorations d'affichage de `claude agents` (largeur de colonnes, troncature des noms).
- v2.1.161 : `OTEL_RESOURCE_ATTRIBUTES` ajouté comme labels sur les métriques ; `claude mcp` list/get/add ne fuit plus les secrets (`${VAR}` non expansé, en-têtes redigés).
- v2.1.160 : le mot-clé déclencheur des workflows dynamiques passe de `workflow` à `ultracode` ; le teardown des sessions background envoie SIGTERM avant SIGKILL ; suppression de la variable `CLAUDE_CODE_OPUS_4_6_FAST_MODE_OVERRIDE` (devenue sans effet).
- v2.1.165 / v2.1.167 / v2.1.168 : corrections de bugs et améliorations de fiabilité.

---

**Pour aller plus loin** — Cette semaine resserre la gouvernance des permissions de Claude Code : glob `deny` `"*"` pour tout refuser, et confirmation avant d'écrire dans des fichiers qui accordent l'exécution de code (`.zshenv`, `.npmrc`, configs de build). Le pendant côté lecture — traiter la configuration d'un dépôt cloné comme du code non fiable — est détaillé dans notre guide : [Treat a cloned repo's Claude Code setup as untrusted code](../resources/guides/untrusted-repo-config.md).

---

*Veille générée à partir du changelog officiel Anthropic.*
*Les interprétations et cas d'usage sont explicitement indiqués comme tels.*
