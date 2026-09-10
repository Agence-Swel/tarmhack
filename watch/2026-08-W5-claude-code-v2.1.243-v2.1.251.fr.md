---
title: "Claude Code : ce qui change cette semaine (v2.1.243 → v2.1.251)"
date: 2026-08-31
period: "Semaine W5 · v2.1.243 → v2.1.251"
sources:
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
  - https://code.claude.com/docs/en/changelog
  - https://registry.npmjs.org/-/package/@anthropic-ai/claude-code/dist-tags
lang: fr
---

# Claude Code : ce qui change cette semaine

> Période : semaine W5 (août 2026) · Versions v2.1.243 → v2.1.251, publiées du 24 au 28 août 2026 (7 versions sur npm)
> Sources : [CHANGELOG.md GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [code.claude.com](https://code.claude.com/docs/en/changelog) · [dist-tags npm](https://registry.npmjs.org/-/package/@anthropic-ai/claude-code/dist-tags)
> Généré le 2026-08-31

> **Statut de stabilité, vérifié le 10 septembre 2026 : tout ce qui suit est encore en préparation.** Les dist-tags npm donnent `stable 2.1.236` / `latest 2.1.267`. La version la plus basse couverte ici, la v2.1.243, est déjà au-dessus du canal stable : si vous suivez stable, rien de tout cela ne vous est encore parvenu. Vérifiez `claude --version` et votre canal avant de déployer quoi que ce soit.
>
> **Deux trous dans la numérotation, et une version muette.** Les v2.1.244 et v2.1.249 n'existent ni au changelog ni sur npm. La v2.1.242, juste en amont de cette plage, existe sur npm mais n'a aucune entrée publique au changelog. Et la v2.1.250 ne porte qu'une ligne, « Bug fixes and reliability improvements ». Cette page couvre ce qui est documenté, pas ce qui est publié : la distinction compte le jour où vous cherchez à comprendre un changement de comportement entre deux versions.

---

## Ce qui compte vraiment

### Un mode restreint, en un seul drapeau, v2.1.248 (27 août 2026)

Quatre garanties d'un coup, sans assembler la moindre règle de permission.

> *"Added `--restricted` (or `CLAUDE_CODE_RESTRICTED=1`): removes the built-in tools that run commands or code and `WebFetch` (unless named in `--tools`), keeps file tools inside the working directory, refuses `bypassPermissions`, and ignores user, project and local settings files"*

**Ce que ça change concrètement** : plus d'exécution de commandes ni de code, plus de récupération de pages web, les outils de fichier confinés au dossier de travail, le contournement des permissions refusé. La clause la plus intéressante est la dernière : les fichiers de settings utilisateur, projet et locaux ne sont plus lus du tout. Jusqu'ici, verrouiller Claude Code voulait dire empiler des règles à la main en espérant n'avoir rien oublié. Ce mode ne durcit pas les règles, il coupe la source des règles : personne ne peut réactiver quoi que ce soit depuis un fichier committé dans le dépôt. Le cas d'usage évident, c'est faire lire du code que vous ne connaissez pas, la PR d'un contributeur externe ou un dépôt fraîchement cloné, sans lui donner de quoi exécuter la moindre chose. Lancez `claude --restricted` et posez vos questions.

---

### Deux hooks pour encadrer les changements de modèle, v2.1.251 (28 août 2026)

Une règle d'équipe cesse d'être un document et devient un refus.

> *"Added `PreModelSwitch` and `PostModelSwitch` hook events (block, confirm, or annotate a model switch); `SessionStart` resume hooks now receive session staleness and the estimated re-cache cost"*

**Ce que ça change concrètement** : vous pouvez intercepter un changement de modèle, le bloquer, demander confirmation, ou simplement le consigner. Si votre équipe a une règle sur les modèles autorisés, pour le coût, pour la conformité ou pour la reproductibilité, elle vivait jusqu'ici dans une page que personne ne relit. Elle peut désormais vivre dans un hook qui dit non. La seconde moitié de la phrase est plus discrète et tout aussi utile : les hooks `SessionStart` de reprise reçoivent l'ancienneté de la session et le coût estimé de reconstruction du cache. Un script peut donc trancher, sans vous, si reprendre une vieille session vaut le prix ou s'il vaut mieux repartir de zéro.

---

### Les chemins se resserrent, quatre fois dans la même version, v2.1.251 (28 août 2026)

Quatre correctifs de la même famille, et un même coupable : le lien symbolique.

> *"Fixed file tools (Read, Write, Edit) following a symlink swapped inside the working directory after the permission check, which could read or write outside the approved location"*
>
> *"Fixed Grep and Glob not applying `Read(...)` deny rules to files reached through a symlinked search path"*

**Ce que ça change concrètement** : un lien peut être échangé entre le moment où l'autorisation est accordée et le moment où le fichier est réellement ouvert. Vos règles `deny` étaient donc contournables par quelque chose d'aussi banal qu'un lien posé dans votre propre dossier de travail. Deux autres correctifs de la même version vont dans le même sens : le Workflow tool lisait un `scriptPath` situé hors de ce que la session a le droit de lire, avant que le contrôle ne s'exécute, et les commandes déclarées par une entrée de marketplace pouvaient pointer hors du répertoire de leur plugin. Si vous vous appuyez sur des règles `Read(...)` en refus pour protéger des fichiers sensibles, prenez cette version. C'est le genre de correctif que personne ne remarque, et qui change ce que vos règles garantissaient vraiment.

---

### Un fichier de dépôt ne redirige plus votre configuration, v2.1.251 (28 août 2026)

Une frontière de confiance qu'on n'avait pas vue passer.

> *"Changed project-level `.claude/settings.json` `env` to no longer set `CLAUDE_CONFIG_DIR`, `CLAUDE_CODE_TMPDIR`, or `TMPDIR`/`TMP`/`TEMP`; set them in your shell, user, or managed settings instead"*

**Ce que ça change concrètement** : `.claude/settings.json` vit dans le dépôt, donc n'importe qui disposant d'un droit de merge pouvait, en théorie, déplacer votre dossier de configuration ou votre dossier temporaire. Ces trois variables déterminent trop de choses pour être laissées à un fichier de projet. Si vous posiez `CLAUDE_CONFIG_DIR` de cette façon, déplacez-le dans votre shell ou dans vos settings utilisateur : ça marchera toujours, simplement plus depuis le dépôt. À noter, la même version ferme une porte voisine : des settings de projet pouvaient activer un traçage bêta détaillé ou la journalisation brute des corps d'API.

---

## Améliorations notables

**Installation et mémoire, sérieusement allégées (v2.1.243, v2.1.251)** Le binaire natif est désormais compressé en zstd, soit *"about 75 MB instead of 340 MB on Linux x64"*, et le code est chargé à la demande au lieu de rester entièrement résident, ce qui vaut *"roughly 40–70 MB less memory per session"*. S'y ajoutent un binaire environ 5 Mo plus petit et le retrait de six langages de coloration syntaxique peu utilisés. Si vous faites tourner plusieurs sessions côte à côte, c'est le changement le plus tangible du lot.

**Un cache de prompt qui tient une heure (v2.1.243, v2.1.248)** Deux réglages arrivent, `promptCacheTtl` et `subagentPromptCacheTtl`, *"so API-key and cloud-provider users can keep a 1-hour prompt cache on the main conversation while subagents stay at 5 minutes"*. Le qualificatif compte : ça vise les accès par clé API et par fournisseur cloud. La v2.1.248 descend d'un cran plus bas, avec `experimental.cacheTtl` (`"5m"` ou `"1h"`) posé dans le frontmatter d'un agent. Sur une session longue et chargée en contexte, c'est exactement là que se joue la facture.

**Savoir quels settings s'appliquent réellement (v2.1.243, v2.1.248)** `/status` liste maintenant les sources de managed settings *"present but not applied because a higher-precedence managed source is active"*, et `/doctor` explique pourquoi un chargement a échoué. Si vous avez déjà passé vingt minutes à chercher pourquoi un réglage « ne prend pas », vous savez précisément pourquoi cette ligne existe.

**Une alerte sur une classe de règle Bash trompeuse (v2.1.246)** Un avertissement au démarrage signale les règles d'autorisation dont le joker précède la sous-commande, comme `Bash(git * main)`, *"since they also match options inserted before the subcommand"*. Autrement dit : votre règle est plus large que vous ne le pensiez. Allez relire vos règles Bash, celle-ci s'écrit par accident.

**`/permissions` gagne un onglet Auto mode (v2.1.246)** pour consulter et modifier les règles du classifieur du mode automatique. Le mode auto cesse d'être une boîte noire.

**Les sous-agents ont un modèle par défaut, plus un modèle imposé (v2.1.251)** `CLAUDE_CODE_SUBAGENT_MODEL` sert désormais *"to set the default subagent model rather than override everything"* : le `model:` déclaré dans une définition d'agent, et un modèle passé explicitement au lancement, l'emportent. C'est l'ordre de priorité auquel on s'attendait, et c'est enfin celui qui s'applique.

---

## Corrections importantes

**Un crash au démarrage sur glibc 2.44 (v2.1.245)** La seule ligne de cette version, et elle compte.

> *"Fixed a crash on startup on Linux distributions that ship glibc 2.44 (for example Arch Linux, CachyOS and Fedora Rawhide)"*

**Ce que ça change concrètement** : si Claude Code refuse de démarrer sur Arch, CachyOS ou Fedora Rawhide, c'est la version qu'il vous faut, la v2.1.245 au minimum. Corrigé.

**Des transcriptions de session écrasées en silence (v2.1.251)** Un changement de répertoire pouvait relocaliser une session sur une transcription existante portant le même identifiant, aussitôt écrasée sans un mot. Votre historique de conversation disparaissait sans message d'erreur. Corrigé.

**Un défaut de cache de prompt environ une fois par heure (v2.1.248)** Dans les sessions longues, un rafraîchissement de jeton OAuth provoquait un nouveau rendu des définitions d'outils, donc un défaut de cache complet et la perte du contexte de réflexion étendue. C'est du coût et de la latence que vous payiez sans le savoir. Corrigé.

**Des sessions Claude Desktop et Cowork disparues au bout de 30 jours (v2.1.248)** Le nettoyage des transcriptions conserve désormais les sessions écrites par l'application de bureau tant qu'elles y figurent, avec un nouveau réglage `desktopSessionCleanupPeriodDays` pour plafonner cette exemption. Corrigé.

**Des worktrees Git supprimés par erreur (v2.1.246)** Le balayage de rétention des sessions en arrière-plan pouvait supprimer des worktrees que vous aviez créés vous-même sous `.claude/worktrees/`, dès lors qu'un ancien enregistrement de session pointait dessus. Si vous travaillez à plusieurs worktrees, c'est la correction à ne pas manquer. Corrigé.

**Des commandes Bash approuvées automatiquement à tort (v2.1.251)** Les contrôles de permission approuvaient sans rien demander les commandes qui affectent une expression arithmétique à une variable entière, du type `OPTIND=1/0` ou `RANDOM=2+2`. Elles demandent maintenant une approbation. Corrigé.

**Des identifiants qui partaient avec les sessions cloud (v2.1.246, v2.1.248)** Deux correctifs de même nature : les requêtes de télémétrie vers Anthropic transportaient la clé API configurée pour une passerelle tierce, *"a credential is now only sent to its own host"* ; et `/ultrareview` téléversait les modifications non committées de fichiers de type `prod.env` et `*.tfvars`, ainsi que les copies temporaires, d'échange et de sauvegarde de fichiers d'identifiants comme `key.pem.tmp` ou `id_rsa.swo`. Elles restent maintenant sur votre machine. Corrigé.

---

## En bref (mineur / cosmétique)

- v2.1.246 : mode plein écran, transcription blanche après redimensionnement, défilement erratique et focus clavier volé par le pointeur.
- v2.1.246 : ralentissement sévère de la transcription quand un diff contenait une ligne unique très longue, par exemple une chaîne base64. Elle est maintenant tronquée avec un marqueur.
- v2.1.246 : rendu markdown désactivé à tort pour tout un message quand ses 500 premiers caractères n'en contenaient aucun.
- v2.1.246 : heatmap d'activité de `/stats` décalée d'une case dans les fuseaux à l'est d'UTC ; `/fork` depuis une session déjà forkée ou passée en arrière-plan démarrait avec une conversation vide.
- v2.1.246 : les couleurs de diff d'un thème personnalisé étaient ignorées, et `/rename` écrasait la couleur de bordure du thème.
- v2.1.243 : `/resume` ne listait que les 50 sessions les plus récentes, le sélecteur charge maintenant à la demande ; la recherche d'historique et la flèche haut cassaient sur une entrée malformée dans `~/.claude/history.jsonl`.
- v2.1.243, v2.1.247 : nouveaux réglages, `modelPicker` pour curer la liste du sélecteur de modèles, `modelPricing` pour les tarifs contractuels d'une organisation, `spinnerTipsOverride` et `feedbackDrafts`.
- v2.1.247 : nouvel outil `SendFeedback`, Claude peut rédiger un rapport que vous relisez avant de l'envoyer depuis `/feedback`. Sur Sonnet 5, l'auto-compactage utilise désormais la fenêtre complète de 1 M, soit environ 967 K tokens au lieu de 934 K.
- v2.1.243, v2.1.251 : `/usage` gagne une répartition par Loop et une barre de plafond de dépense ; `/cost` reçoit une ligne de cache de prompt par session.
- v2.1.248 : la description du Workflow tool passe d'environ 5,7 k à 1 k tokens, la référence d'écriture de scripts partant dans un skill dédié.
- v2.1.251 : cinq commandes rejoignent `claude --help`, `attach`, `logs`, `stop`, `respawn` et `rm`.


---

**Pour aller plus loin** : quatre correctifs de containment dans une seule release, et ils racontent tous la même histoire. Une règle `deny` qu'un lien symbolique contourne, un chemin dont les parenthèses font tomber la règle en silence, une commande approuvée à tort parce qu'elle affecte une expression arithmétique. Le contrôle de commande est un filet, pas un mur, et la parade ne consiste pas à écrire de meilleurs motifs : elle consiste à partir du refus, puis à vérifier ce qui s'applique réellement. Comment poser ce plancher, et pourquoi une règle qu'on n'a jamais vue s'appliquer ne protège personne : notre guide [Harden your own Claude Code permissions](../resources/guides/harden-claude-code-permissions.fr.md).

---

*Veille générée à partir du changelog officiel Anthropic.*
*Les interprétations et cas d'usage sont explicitement indiqués comme tels.*
