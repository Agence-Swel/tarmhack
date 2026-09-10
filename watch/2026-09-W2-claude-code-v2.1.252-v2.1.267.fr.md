---
title: "Claude Code : ce qui change cette semaine (v2.1.252 → v2.1.267)"
date: 2026-09-10
period: "Semaine W2 · v2.1.252 → v2.1.267"
sources:
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
  - https://registry.npmjs.org/-/package/@anthropic-ai/claude-code/dist-tags
lang: fr
---

# Claude Code : ce qui change cette semaine

> Période : semaine W2 (septembre 2026) · Publications du 31 août au 9 septembre 2026 · Versions v2.1.252 → v2.1.267 (16 numéros dans l'intervalle, 10 seulement portent une entrée publique : 253, 254, 255, 256, 262 et 264 n'en ont aucune. On n'apprend l'existence de la v2.1.255 que par le correctif de la v2.1.258, qui la nomme comme la version ayant cassé le lancement sur macOS 12)
> Sources : [CHANGELOG.md GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [code.claude.com](https://code.claude.com/docs/en/changelog) · [dist-tags npm](https://registry.npmjs.org/-/package/@anthropic-ai/claude-code/dist-tags)
> Généré le 2026-09-10

> **Statut de stabilité : la totalité de cette édition est en préparation.** Les dist-tags npm donnent `stable 2.1.236` / `latest 2.1.267`. Le canal stable n'a pas bougé depuis le 19 août, et c'est en soi l'information la plus utile de la semaine : vingt-trois versions ont été publiées au-dessus de lui depuis, dont les dix que couvre cette édition. Si vous suivez stable, rien de ce qui suit ne vous a atteint. Trois semaines d'écart méritent qu'on vérifie où l'on se trouve : `claude --version`, puis votre canal, avant de déployer.

---

## Ce qui compte vraiment

### Fable change de génération, sans que vous ayez rien fait, v2.1.257 (1er septembre 2026)

Ce n'est pas un modèle de plus dans le menu : c'est celui qui répond par défaut qui a changé.

> *"Added Claude Fable 5.1 (`claude-fable-5-1`), now the default Fable model — 1M context, $10/$50 per Mtok with $0.25/Mtok cache reads"*

**Ce que ça change concrètement** : un flux qui demande « Fable » sans plus de précision tourne désormais sur la 5.1, fenêtre de contexte d'un million de jetons, tarif annoncé à 10 / 50 dollars par million, lectures de cache à 0,25. Ceux qui épinglent `claude-fable-5` par son nom exact restent où ils sont. Deux nuances comptent plus que l'annonce.

> *"Changed `fable` and `best` in Claude apps gateway sessions to keep resolving to Fable 5 for now, since gateways not yet configured for Fable 5.1 reject it; pick Fable 5.1 in `/model` to use it"*

> *"Fixed prompt caching on Claude Fable 5.1 not covering the context attached after tool results, so it was re-sent as uncached input on every tool-call turn"* (v2.1.260, 3 septembre 2026)

**Note** : « par défaut » n'est donc pas universel, les sessions passant par une passerelle Claude apps restant sur Fable 5. Et pendant les deux premiers jours du modèle, le contexte rattaché après un résultat d'outil échappait au cache et repartait en entrée pleine à chaque tour. Si vous avez basculé le 1er septembre, la facture du 1er au 3 vaut un coup d'œil.

---

### Une organisation peut désormais imposer ses serveurs MCP à tout le monde, v2.1.259 (2 septembre 2026)

Jusqu'ici, un serveur MCP se configurait projet par projet, ou poste par poste.

> *"Added `managedMcpServers` managed setting: organizations can provide HTTP/SSE MCP servers to every user (same entry shape as `.mcp.json`); entries that name a command to run are skipped"*

**Ce que ça change concrètement** : c'est le levier qui manquait pour standardiser l'accès à un outil interne sans dépendre de la discipline de chaque poste, au format `.mcp.json` déjà connu. Et la restriction est bien choisie : une entrée qui nomme une commande à exécuter est purement ignorée, donc un réglage poussé depuis le serveur ne peut pas lancer de binaire local. HTTP et SSE, rien d'autre.

> *"Changed `allowedMcpServers` to govern only servers users add: a literal `managed-mcp.json` server your allowlist used to filter out now loads on upgrade; use `deniedMcpServers` to keep it off"*

**Note** : le même lot déplace une frontière sans le dire très fort. Une liste d'autorisation qui filtrait jusqu'ici un serveur `managed-mcp.json` cesse de le filtrer à la mise à jour. Rien ne casse visiblement, et ce qui tourne chez vos utilisateurs n'est plus tout à fait ce que vous aviez décidé. Si vous administrez un parc, c'est la ligne à relire avant de promouvoir cette version.

---

### Un diagnostic qui vous dit ce que vos skills coûtent, v2.1.261 (4 septembre 2026)

Chaque skill chargé consomme du contexte, qu'il serve ou non dans la session en cours.

> *"Added `/skill-doctor` to show which loaded skills go unused and what they cost in context, so you can prune them"*

**Ce que ça change concrètement** : savoir lesquels étaient du poids mort relevait de la mémoire ou de la chance. La commande donne les deux moitiés de la décision d'un coup : ce qui ne sert pas, et ce que ça coûte. C'est le genre d'outil modeste qui change une habitude, parce qu'on élague ce qu'on mesure. Passez-la sur un projet installé depuis six mois : le résultat est rarement flatteur.

---

### Le confinement des plugins se fait reprendre trois fois en neuf jours, v2.1.267 (9 septembre 2026)

Trois correctifs, trois versions, la même famille de faille : un chemin déclaré par un plugin ou par une entrée de marketplace qui sort du répertoire où il était censé rester enfermé.

> *"Fixed plugins being able to read files outside their own directory through a declared command, agent, skill, hooks or other component path that is a symlink; such paths are now refused with an error"* (v2.1.257, 1er septembre 2026)

> *"Fixed a plugin path containing a backslash bypassing the symlink containment check on macOS and Linux"* (v2.1.265, 8 septembre 2026)

> *"Fixed a case where a marketplace entry path containing a backslash could bypass the containment check for fetched marketplaces on macOS and Linux"* (v2.1.267, 9 septembre 2026)

**Ce que ça change concrètement** : aucun des trois cas n'est présenté comme exploité, et rien n'indique une urgence. Ce qui parle, c'est la cadence : un lien symbolique le 1er, un antislash dans un chemin de plugin le 8, le même antislash dans une entrée de marketplace le 9. Une même erreur de raisonnement corrigée à trois endroits, ce qui est la signature d'une classe de risque qu'on ratisse, pas d'un incident isolé. La conclusion tient en une ligne : un plugin tiers s'exécute avec vos droits, et la barrière qui le tient à sa place est en cours de réparation. Privilégiez les marketplaces dont vous connaissez l'auteur, et prenez ces versions dès qu'elles atteignent stable.

---

## Améliorations notables

**Un panneau de diff à côté de la conversation (v2.1.260).** `/diff` ouvre en mode plein écran un panneau qui montre vos changements non commités au fil des éditions de Claude, sans basculer vers un terminal ou votre éditeur.

> *"Added a diff panel that opens beside the conversation in fullscreen mode and shows your uncommitted changes as Claude edits; toggle it with `/diff`"*

**Le mode auto se durcit (v2.1.257, v2.1.261).** Une règle d'évasion de confinement retire de l'auto-approbation les récupérations d'identifiants de métadonnées cloud, les contournements de filtrage sortant et les accès inter-locataires, *"unless your environment marks them expected"*. Une invite unique apparaît avant la première lecture de fichier hors des répertoires de travail (`permissions.blockReadsOutsideWorkingDirectories` permet de les bloquer). Et un lien qui encode du contenu dans l'URL d'un générateur de diagrammes public est traité pour ce qu'il est, un envoi vers ce site.

**GitLab cesse d'être un citoyen de seconde zone (v2.1.257, v2.1.259, v2.1.260).** Les commandes `glab mr create/merge/close/reopen/note/update` sont reconnues et affichent la merge request en `MR !N` dans le résumé d'outil ; la détection de dépôt couvre les sous-groupes imbriqués ; une référence `owner/repo#123` pointe vers l'issue gitlab.com au lieu de github.com ; et `/code-review --comment` poste ses constats via `glab mr note`.

---

## Corrections importantes

**Une campagne entière contre les ratés de cache de prompt (v2.1.265, v2.1.267).** Le fil le plus dense du lot : une douzaine de lignes, sur deux versions, qui traquent le même symptôme. Un élément du préfixe de conversation changeait en cours de route, le cache tombait, et la facture montait sans que rien ne le signale.

> *"Fixed switching models with /model re-sending every tool definition (a prompt-cache miss); commit and PR attribution text now arrives as a conversation note that updates on model changes"*

> *"Improved prompt-cache stability: subagents and sessions started with `--system-prompt` or `--append-system-prompt` now record the system prompt and tool definitions once instead of re-rendering them"*

**Ce que ça change concrètement** : les déclencheurs corrigés sont ceux du quotidien, changer de modèle en cours de session, reprendre une conversation, voir un serveur MCP se reconnecter, lancer un sous-agent. Si vous surveillez vos coûts, comparez votre taux de lecture de cache avant et après ces versions plutôt que de croire l'un ou l'autre sur parole.

**Des règles de permission qui laissaient un dossier « lecture seule » inscriptible (v2.1.260).** La correction la plus sérieuse de la semaine, et elle porte sur des fichiers de réglages qu'on écrit une fois puis qu'on oublie.

> *"Fixed `Edit`/`Write`/`Read` permission rules whose path contains parentheses being dropped as invalid or ignored by the Bash sandbox, which left \"read-only\" folders writable"*

**Ce que ça change concrètement** : une parenthèse dans un chemin suffisait à faire tomber la règle, silencieusement. Allez relire vos règles maintenant : celles qui contiennent une parenthèse, un crochet ou une accolade sont exactement celles qui pouvaient ne rien protéger du tout.

**Des sessions concurrentes qui s'écrasaient l'une l'autre (v2.1.259).** Travailler avec plusieurs sessions ouvertes en parallèle pouvait faire perdre en silence des changements écrits par une autre.

> *"Fixed concurrent sessions silently reverting each other's `~/.claude.json` changes — workspace trust no longer resets and MCP/project state is no longer lost when running many sessions at once"*

**Un réglage managé mal formé bloque désormais le démarrage (v2.1.259).** Si le fichier de réglages managés, un fichier déposé à côté, le plist MDM ou la valeur de registre Windows ne se laisse pas analyser, Claude Code refuse de démarrer et nomme la source. Avant, il démarrait sans appliquer des règles de gouvernance qu'on croyait actives.

**Un aller-retour sur les règles `Read()` appliquées à Bash (v2.1.259 puis v2.1.260).** La v2.1.259 étendait les règles de refus `Read()` aux arguments de commandes Bash. La v2.1.260 **révoque** ce changement : il refusait `npm run build` sous une règle `Read(./**/build/**)` dans tous les modes, et faisait demander confirmation sur un simple `cd … && grep` même en mode auto. Si le mode auto vous a paru inutilisable début septembre, c'était ça.

---

## En bref (mineur / cosmétique)

- v2.1.267 : `maxEffortLevel` plafonne le niveau d'effort sur tous les fournisseurs, Bedrock, Vertex et Foundry compris. `--system-prompt-snapshot off` force un rendu frais du prompt système à chaque requête, utile en itération de prompt. Les réglages managés `allowedHttpHookUrls`, `httpHookAllowedEnvVars` et `allowedChannelPlugins` n'admettent plus rien, au lieu de tout, quand ils sont illisibles : en cas de doute, on ferme.
- v2.1.266 : correctif de régression pour les passerelles LLM et proxys tiers, la variable non documentée `CLAUDE_CODE_USE_GATEWAY` s'étant mise à forcer seule une connexion à la passerelle Cloud. Publié le jour même que la version fautive.
- v2.1.265 : `--plugin-dir` accepte un dossier de plugins, ajouts et retraits à chaud compris ; les résultats d'outil sauvegardés sur disque sont plafonnés à 1 Go.
- v2.1.263 : entrée publique réduite à « Bug fixes and reliability improvements ». Nous n'en inventerons pas davantage.
- v2.1.261 : `bashOutputMaxChars` et `taskOutputMaxChars` relèvent, jusqu'à 128 K caractères, la quantité de sortie de commande ou de tâche de fond que Claude reçoit dans la conversation avant qu'elle ne bascule sur disque. Les touches d'édition par mot du prompt s'alignent sur Bash et `keybindingFlavor` n'a plus aucun effet. `--append-subagent-system-prompt-file` charge un prompt système de sous-agent depuis un fichier. Une ligne « Organization policy » dans `/status` dit pourquoi la politique d'organisation n'a pas chargé.
- v2.1.260 : `/reload-plugins` et une forme textuelle d'`/advisor` arrivent en session headless. `ctrl+l` / `cmd+k` en plein écran nettoie la vue comme un `clear`. La limite d'une heure sur les commandes de fond lancées par un sous-agent disparaît.
- v2.1.259 : `--permission-prompts none` pour les hôtes headless sans surveillance, tout ce qui demanderait confirmation étant refusé automatiquement. `--json` sur `claude plugin validate`.
- v2.1.258 : Claude Code ne démarrait plus sur macOS 12 (Monterey), une régression de la v2.1.255.
- v2.1.257 : réglages `timeZone` et `timeFormat`. `defaultMode: "bypassPermissions"` posé dans les réglages de projet est désormais ignoré, comme `"auto"` l'était déjà : passez par les réglages utilisateur, managés, ou `--permission-mode`.
- v2.1.252 à v2.1.267 : rendu terminal et reprise de session, emoji composés coupés en fin de ligne, texte de droite à gauche mêlé à de l'anglais, transcripts de plus de 5 Mo et appels d'outils parallèles perdus à la reprise, « always allow » qui ne s'enregistrait pas dans un projet sans `.claude/settings.local.json`. Sous Windows, `Read`, `Write` et `Edit` refusaient tous les fichiers dans un bac à sable AppContainer.


---

**Pour aller plus loin** : trois correctifs de containment en trois semaines sur les plugins et les marketplaces, un chemin de composant passé par un lien symbolique puis deux orthographes d'antislash, tous refermés après coup. La leçon n'est pas qu'un correctif manquait, c'est que la surface d'entrée d'un dépôt tiers est plus large qu'elle n'en a l'air : ce qui se charge depuis un marketplace, un plugin ou un skill s'exécute chez vous, avec vos droits. La checklist manuelle, gratuite et sans outil, pour savoir ce qu'un dépôt cloné apporte réellement avec lui : notre guide [Treat a cloned repo's Claude Code setup as untrusted code](../resources/guides/untrusted-repo-config.fr.md).

---

*Veille générée à partir du changelog officiel Anthropic.*
*Les interprétations et cas d'usage sont explicitement indiqués comme tels.*
