---
title: "Traitez la configuration Claude Code d'un dépôt cloné comme du code non fiable"
description: "Quand vous ouvrez le projet de quelqu'un d'autre dans Claude Code, sa configuration cachée peut s'exécuter sur votre machine. Voici le réflexe durable, et une checklist gratuite, pour garder la main."
last_revalidated: 2026-09-10
claude_code_ref: v2.1.267
lang: fr
sources:
  - https://research.checkpoint.com/2026/rce-and-api-token-exfiltration-through-claude-code-project-files-cve-2025-59536/
  - https://flatt.tech/research/posts/poisoning-claude-code-one-github-issue-to-break-the-supply-chain/
  - https://code.claude.com/docs/en/security
  - https://code.claude.com/docs/en/permissions
  - https://code.claude.com/docs/en/skills
  - https://code.claude.com/docs/en/settings
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
  - https://www.npmjs.com/package/@anthropic-ai/claude-code?activeTab=versions
---

# Traitez la configuration Claude Code d'un dépôt cloné comme du code non fiable

## Pourquoi le sujet tombe maintenant

Fin mai 2026, Claude Code a rendu le partage d'environnements encore plus simple : les plugins
déposés dans le répertoire `.claude/skills` d'un projet sont désormais **chargés
automatiquement, sans étape de marketplace** (v2.1.157). Un seul `git clone` suffit pour qu'un
collègue, ou un inconnu, vous remette son environnement complet.

C'est réellement pratique. C'est aussi le bon moment pour ancrer un réflexe :

> **La configuration Claude Code d'un dépôt n'est pas une métadonnée. C'est du code exécutable
> auquel vous vous apprêtez à faire confiance.**

Et cette surface n'a pas cessé de bouger. En août 2026, Anthropic a dû corriger des dépôts git
imbriqués qui héritaient de la confiance accordée à un répertoire parent (v2.1.232), ce qui en
dit long sur la portée réelle qu'avait un simple clic sur **trust**. Dans les quatre semaines
qui ont suivi, jusqu'à la **v2.1.267**, deux choses se sont produites : la barrière censée
maintenir un plugin ou un marketplace à l'intérieur de son propre dossier a été réparée quatre
fois de suite, et la confiance accordée à un dossier a pris du mordant à un endroit précis où
le dialogue ne vous est jamais montré. Les deux points sont détaillés plus bas.

### D'abord : de quelle « version courante » parlez-vous ?

Avant toute chose, et cela ne concerne le produit d'aucun éditeur. Le **10 septembre 2026**,
`npm dist-tag ls @anthropic-ai/claude-code` renvoie deux réponses très différentes :

| Canal | Version | Publiée le |
|---|---|---|
| `stable` | **2.1.236** | 19 août 2026 |
| `latest` | **2.1.267** | 9 septembre 2026 |

Le canal `stable` n'a pas bougé depuis trois semaines. Comptez précisément, parce que c'est
le chiffre rond qui trompe : sur les quatorze versions citées dans cette page, **huit sont
arrivées après le 19 août**. Un lecteur qui « garde consciencieusement Claude Code à jour »
sur `stable` n'a aucune de ces huit. Suivre un canal plus lent est un choix
légitime, c'est même sa raison d'être. Mais cela fait dire deux choses différentes à deux
phrases qui sonnent pareil : « je suis à la dernière version » et « j'ai ce correctif ».

**La vérification gratuite** : lancez `claude doctor`, lisez le numéro de version, comparez-le
aux versions citées dans cette page. En dessous du correctif, vous n'avez pas le correctif,
quel que soit le canal que vous suivez.

## La menace, en clair

Quand vous ouvrez un projet, Claude Code lit sa configuration locale : `.claude/settings.json`,
hooks, serveurs MCP, surcharges d'environnement, et maintenant plugins chargés
automatiquement. N'importe qui ayant un droit de commit sur un dépôt peut modifier ces
fichiers, et ils n'obtiennent pas l'examen qu'on réserve au code applicatif en revue, parce que
nous les lisons instinctivement comme « juste des réglages ».

Ce n'est pas une hypothèse d'école. Deux affaires documentées en 2026 l'ont montré :

- **CVE-2025-59536** (Check Point Research, CVSS 8.7) : un hook planté dans le fichier de
  réglages d'un dépôt pouvait exécuter des commandes shell **avant même que le dialogue de
  confiance n'apparaisse**. Vous ouvriez le dossier, du code tournait.
- **CVE-2026-21852** (CVSS 5.3) : un projet pouvait surcharger `ANTHROPIC_BASE_URL` dans sa
  configuration, amenant Claude Code à envoyer une requête authentifiée, porteuse de votre clé
  d'API, vers le serveur d'un attaquant, **avant** que vous ayez confirmé faire confiance au
  projet.

Par ailleurs, GMO Flatt Security a démontré un chemin d'attaque sur la chaîne
d'approvisionnement où **une seule issue GitHub** pouvait suffire à empoisonner un
environnement en aval.

**Point important, et c'est la raison pour laquelle cette fiche ne joue pas sur la peur** :
tout cela est **corrigé** dans les versions actuelles de Claude Code. Les failles ont disparu.
Ce qui reste utile, c'est l'*habitude* qu'elles révèlent, parce que la surface sous-jacente,
une configuration capable d'agir en votre nom, est permanente, elle.

## Le principe durable

Traitez l'invite de confiance comme une vraie décision de sécurité, pas comme un dialogue à
faire disparaître. Avant de laisser Claude Code agir dans un dépôt que vous ne connaissez pas,
partez du principe que sa configuration peut lancer des commandes, atteindre le réseau et
reconfigurer vos outils, puis décidez délibérément.

**Ensuite, sachez ce que cette décision couvre.** Une décision de confiance a une *portée*, et
cette portée n'est pas « tout, pour toujours, partout sous ce dossier ». Deux axes, tous deux
tirés de la référence officielle sur les permissions :

- **Où elle s'applique.** La confiance est enregistrée par espace de travail, *"keyed on the
  git repository root or, outside a repository, the directory you started Claude Code from."*
  Cela compte dès qu'un dépôt en contient d'autres : sous-modules, dépendances vendorées,
  sous-dépôts d'un monorepo. Jusqu'à la **v2.1.232**, ces dépôts imbriqués héritaient de la
  confiance du parent. Chacun demande désormais pour lui-même. Sur un CLI à jour, attendez-vous
  à *plus* d'invites, pas à moins.
- **Ce qu'elle couvre à l'intérieur du dossier.** Tout n'est pas conditionné par elle. Le
  tableau de la documentation *What runs before you trust a folder* est explicite : quand vous
  n'avez fait confiance qu'à un dossier parent, les hooks des fichiers de réglages, le bloc
  `env`, les commandes assistantes comme `apiKeyHelper`, ainsi que les hooks et les
  `allowed-tools` d'un skill de projet sont **utilisés**, tandis que les règles
  `permissions.allow` et `additionalDirectories` sont retenues jusqu'à votre acceptation. Sur
  les skills, la documentation va plus loin : *"A skill can grant itself broad tool access, so
  review the `allowed-tools` of skills checked into a repository before you run Claude Code
  there."*

Et un troisième point, qui est en réalité un piège : **on ne vous demande pas toujours**, et
depuis août, « on ne vous demande pas » ne veut plus dire la même chose partout. Le dialogue
reste réservé à l'interactif : *"A `claude -p` run or an SDK session never shows it."* Mais
l'absence de réponse à ce dialogue a désormais des conséquences propres, et une commande
supposément interactive sautait purement et simplement l'invite. Les deux cas sont plus bas.

## Une checklist manuelle et gratuite (aucun outil requis)

Ouvrez ces fichiers dans un **éditeur de texte**, et non en lançant Claude Code à l'intérieur
du dépôt, quand la source n'est pas quelqu'un à qui vous faites déjà confiance :

1. **`.claude/settings.json`** et **`.claude/settings.local.json`** : lisez-les entièrement, et
   cherchez d'abord `defaultMode`, la chose la plus bruyante qu'un dépôt puisse committer.
   Jusqu'à la **v2.1.257**, un fichier de réglages de projet pouvait le poser à
   `"bypassPermissions"` et être obéi ; cette version *"Changed `defaultMode:
   "bypassPermissions"` in `.claude/settings.json` or `.claude/settings.local.json` to be
   ignored, like `"auto"`"*. Sur un CLI plus ancien, une seule ligne de JSON committée éteint
   toutes les invites sur lesquelles vous comptez.
2. **Les hooks** : pour chaque hook, lisez la commande shell réellement exécutée. Tout ce qui
   part vers un appel réseau, touche à des identifiants ou lance un script opaque est un signal
   d'arrêt.
3. **Les surcharges d'environnement et d'en-têtes** : surveillez précisément
   `ANTHROPIC_BASE_URL`, `apiKeyHelper`, et toute variable qui détourne la destination de votre
   trafic ou de vos clés. Deux de ces portes se sont refermées en **v2.1.251**, ce qui vous dit
   qu'elles étaient ouvertes : le bloc `env` d'un projet peut *"no longer set
   `CLAUDE_CONFIG_DIR`, `CLAUDE_CODE_TMPDIR`, or `TMPDIR`/`TMP`/`TEMP`"*, et
   `ANTHROPIC_CUSTOM_HEADERS` venu des réglages de projet *"require[s] approval when it sets a
   credential, org/tenant, routing, or API-behavior header (e.g. `Authorization`, `Host`)."* Ce
   qui ne figure pas sur ces deux listes reste ce que le dépôt en dit.
4. **Les serveurs MCP** (`.mcp.json` et serveurs configurés) : quelle commande chacun
   lance-t-il, et d'où vient ce binaire ? Lisez un `headersHelper` avec le même œil qu'un hook,
   c'est une commande qui s'exécute pour fabriquer des en-têtes HTTP.
5. **Les plugins chargés automatiquement et `.claude/skills`** : depuis la v2.1.157, un dossier
   déposé là qui porte un manifeste `.claude-plugin/plugin.json` se charge comme plugin, sans
   marketplace et sans étape d'installation, dès que vous acceptez le dialogue de confiance de
   l'espace de travail. Lisez ce qui s'y trouve, en commençant par les `allowed-tools`, puis par
   les **chemins** déclarés dans le manifeste (voir *La surface d'entrée est plus large qu'elle
   n'en a l'air*, plus bas). Depuis la **v2.1.233**, vous disposez d'une première passe
   gratuite : `claude plugin validate <chemin>` contrôle *"a bare `.claude/skills` directory,
   reporting SKILL.md files whose frontmatter fails to parse"*, et depuis la **v2.1.259** il
   accepte *"`--json` … for a machine-readable validation report"* si vous le passez sur de
   nombreux dépôts. Prenez-le pour ce qu'il est : il vous dit qu'un skill est bien *formé*, pas
   qu'il est inoffensif.
6. **Les fichiers de contexte livrés par le dépôt** : `CLAUDE.md`, ses inclusions `@`,
   `.claude/rules`. Ils se lisent comme de la documentation, ils sont chargés avant que vous
   ne fassiez quoi que ce soit, et cela en fait aussi une surface d'accès aux fichiers. La
   **v2.1.234** a livré un correctif *"Security"* pour que *"remote file reads, session restore,
   CLAUDE.md includes, workflow scripts and file uploads now reject Windows NT-namespace
   (`\??\`) paths, hardening the remaining pre-approval file accesses against the NTLM
   credential-leak vector."* Lisez les inclusions, et suivez où elles pointent.
7. **La configuration qui n'est jamais venue du dépôt.** Autre porte, et les points 1 à 6 ne
   l'attraperont pas : les skills que vous avez activés sur votre compte claude.ai. Claude Code
   les télécharge dans `~/.claude/skills/synced/` quand `CLAUDE_CODE_SYNC_SKILLS` est posée
   dans une exécution non interactive, et chaque session locale ultérieure les charge depuis
   là. Auditez ce dossier comme n'importe quel autre. Et sachez que les noms entrent en
   collision d'une source à l'autre : `/skills` et `/context` regroupent les skills synchronisés
   sous `claude.ai sync`, et le menu `/` les étiquette. Servez-vous de cette étiquette quand une
   commande familière se met à se comporter de façon inhabituelle. Les plugins synchronisés ont
   reçu le même traitement en **v2.1.239** : ils *"now show as `name@synced` … and never
   override a same-named plugin you installed."*

Si quelque chose vous paraît anormal, n'ouvrez pas le projet dans Claude Code depuis ce
répertoire tant que vous ne l'avez pas nettoyé ou supprimé.

## Le CLI commence à soutenir ce réflexe

Un petit signe que la direction est la bonne. Extrait du changelog officiel, **v2.1.196** :

> *"Security: `claude mcp list`/`get` no longer spawn `.mcp.json` servers that a repo
> self-approved via a committed `.claude/settings.json`; untrusted workspaces show
> `⏸ Pending approval`."*

Dépliez la phrase, elle recouvre exactement le point 4 ci-dessus. Un dépôt pouvait approuver
*ses propres* serveurs MCP en committant `enableAllProjectMcpServers: true` (ou une liste
`enabledMcpjsonServers`) dans `.claude/settings.json`, et alors même un innocent
`claude mcp list` *lançait* ces processus serveurs. C'est de l'exécution de code déclenchée par
une commande qui se lit comme une inspection. Désormais, dans un dossier non approuvé, ces deux
commandes ignorent une approbation arrivée par un fichier committé ; elle ne compte que si elle
vit dans un fichier de réglages qui vous appartient et que vous n'avez pas committé
(`.claude/settings.local.json`). La référence sur les permissions énonce le principe sans
détour : *"The repository's own approvals don't count."* **Un dépôt n'a pas le droit de se
porter garant de lui-même.**

### Et le cas le plus récent est le plus parlant

Un mois plus tard, **v2.1.232** (13 août 2026) :

> *"Fixed nested git repositories inheriting trust from a parent directory; each repository
> now requires its own trust confirmation."*

Lisez ce que cela implique du comportement remplacé, c'est là qu'est la leçon durable : faire
confiance à un répertoire couvrait aussi tous les dépôts git imbriqués à l'intérieur. Un
sous-module, une dépendance vendorée, un sous-dépôt de monorepo, du code que vous n'aviez
jamais ouvert, couvert par une approbation donnée pour autre chose. C'est corrigé. Mais la
raison pour laquelle cela valait d'être corrigé est la raison de garder le réflexe : **une
décision de confiance a un rayon d'action, et il était plus large que la plupart des gens ne le
supposaient.**

Ce rayon est d'ailleurs toujours en cours de cartographie. La **v2.1.234** *"Fixed trust
prompts omitting the repository-wide scope warning when the directory was first seen before the
repository existed there"* : l'invite était bien affichée, la phrase qui vous disait jusqu'où
elle portait ne l'était pas. Et la **v2.1.265** *"Fixed Claude Code's own git status and diff
probes running clean filters configured by a nested repository inside the working tree"* : un
dépôt imbriqué pouvait configurer un filtre git, et le `git status` de routine de Claude Code
l'exécutait lui-même. Aucun des deux n'est une catastrophe. Ensemble, ils disent la même chose :
**l'imbrication est le cimetière des présupposés.**

La même semaine que la v2.1.232, une autre porte, **v2.1.228** (11 août 2026) :

> *"Hardened skills synced from claude.ai: they no longer shadow local commands or MCP
> prompts, their descriptions are sanitized and labeled, and on your machine their bodies
> don't run `!` commands or expand `@` files."*

Celle-là ne parle pas de dépôts du tout. Elle parle d'une configuration qui arrive par votre
*compte*, et c'est la raison d'être du point 7. Notez au passage ce qu'elle dit des collisions
de noms : elles sont réelles, et elles ne se résolvent pas toujours comme on l'imagine. La
**v2.1.233** a corrigé des alias de skills fournis avec le CLI, comme `/checkup` et `/review`,
qui répondaient `Unknown command` en mode `-p` quand un skill utilisateur ou projet les
masquait. Les sources se masquent l'une l'autre, dans les deux sens. L'étiquette
`claude.ai sync` existe pour être lue.

### La confiance décide maintenant de choses qu'on ne vous demande pas

C'est le changement qui affecte le plus ce réflexe, et il tombe pile sur le piège décrit plus
haut. **v2.1.238** (20 août 2026) :

> *"MCP `headersHelper` in a project `.mcp.json`, and inline MCP servers in project or
> `--add-dir` agent files, now require that folder's trust dialog to have been accepted
> (also under `claude -p`)."*

Rien n'a changé au dialogue lui-même : une exécution en `-p` continue de ne jamais l'afficher.
Ce qui a changé, c'est que son *absence de réponse* a désormais une conséquence. Dans un
dossier auquel vous n'avez jamais fait confiance, ces deux choses s'arrêtent au lieu de
démarrer, y compris dans le contexte scripté précis où l'on n'allait de toute façon jamais vous
demander votre avis. **Le verrou n'a plus besoin de vos yeux pour être un verrou.**

Deux lectures en découlent. La confortable : si une exécution en `-p` dans un dépôt inconnu
signale qu'un serveur MCP ne démarre pas, c'est la fonctionnalité, pas un obstacle à
contourner. L'autre : cela ne mord que là où la confiance est réellement absente. Faites
confiance une fois, pour « jeter un œil », et chaque exécution headless ultérieure dans ce
dossier hérite silencieusement de cette décision, sans rien pour vous le rappeler. **Un clic de
confiance est un état durable sur votre disque, pas la réponse ponctuelle à une question
ponctuelle.**

**La parade manuelle, gratuite, une minute.** Cet état est écrit dans `~/.claude.json` : le
correctif v2.1.259 sur les sessions concurrentes nomme le fichier quand il dit que la
*"workspace trust no longer resets"* à cet endroit. C'est du JSON en clair. Ouvrez-le et lisez
les entrées de projet : cette liste est le seul endroit où vos décisions de confiance passées
sont consignées, et c'est la page que personne ne pense à relire.

La même version a refermé le volet identifiants : un `headersHelper` venu d'un `.mcp.json` de
projet, d'un plugin ou d'un fichier d'agent *"runs without inherited credential env vars."* Un
assistant fourni par le dépôt ne démarre plus en possession de vos jetons.

Et l'énumération de la documentation elle-même, dialogue affiché en interactif, jamais en `-p`
ni dans le SDK, avait un troisième cas. La **v2.1.248** *"Fixed `claude agents` skipping the
workspace trust prompt when the `CI` environment variable is set."* Un `CI=true` qui traîne dans
un profil shell, une image de conteneur, un lanceur de tâches : n'importe lequel des trois
faisait se comporter une commande interactive comme une commande headless. « Suis-je
interactif ? » se répond par heuristique, et une heuristique a des bords.

### La surface d'entrée est plus large qu'elle n'en a l'air

La checklist vous dit de lire les `allowed-tools` d'un plugin. Nécessaire, pas suffisant : un
plugin déclare aussi des **chemins**, et un chemin est une affirmation sur l'endroit où vit un
fichier, que quelque chose doit vérifier. Entre le 28 août et le 9 septembre, quatre versions
ont réparé cette vérification.

- **v2.1.251** : *"Fixed plugin commands declared in a marketplace entry being able to point
  outside the plugin directory; such paths are now rejected with a path-traversal error."*
- **v2.1.257** : *"Fixed plugins being able to read files outside their own directory through
  a declared command, agent, skill, hooks or other component path that is a symlink; such
  paths are now refused with an error."*
- **v2.1.265** : *"Fixed a plugin path containing a backslash bypassing the symlink
  containment check on macOS and Linux."*
- **v2.1.267** : *"Fixed a case where a marketplace entry path containing a backslash could
  bypass the containment check for fetched marketplaces on macOS and Linux."*

Quatre entrées, une seule phrase : **ce qui se charge depuis un marketplace, un plugin ou un
skill s'exécute avec vos droits, et la barrière censée le tenir dans son propre dossier est du
code jeune.** Les deux dernières sont le même défaut trouvé deux fois : même antislash, deux
chemins de code différents, à un jour d'intervalle (les 8 et 9 septembre). C'est exactement
l'allure d'une frontière encore en train de durcir.

**La parade manuelle, gratuite, une minute.** Avant tout le reste dans un dépôt inconnu, ouvrez
chaque `plugin.json` et chaque entrée de marketplace, et lisez les chemins qu'ils déclarent.
Tout ce qui remonte (`../`), tout ce qui est un lien symbolique, tout ce qui s'écrit avec un
antislash sur une machine de type Unix : ce sont précisément les formes visées par les quatre
correctifs. `ls -lR .claude/` répond à la question des liens symboliques ;
`grep -rn '\.\.' .claude-plugin/` répond à l'autre.

### L'audit n'est pas un portique qu'on franchit une fois

La checklist se lit comme quelque chose qu'on fait à la porte. Deux versions ont rendu cette
lecture moins vraie. La **v2.1.246** a amélioré `/cd` de sorte que *"the new directory's project
settings, hooks, `.mcp.json` servers (behind the usual approval prompt), skills, and agents now
take effect right after the move instead of on `--resume`."* Et la **v2.1.257** *"Fixed settings
in a `.claude/` folder created after startup not being picked up until restart"* : un `.claude/`
qui apparaît pendant que vous travaillez, à la faveur d'un `git pull` ou d'un changement de
branche, s'applique immédiatement.

Les deux corrigent une gêne réelle. Les deux signifient que la version honnête du réflexe
devient : **rejouez la checklist quand la configuration change sous vous.** Après avoir tiré une
branche qui touche à `.claude/`, avant de faire `/cd` vers un endroit que vous n'avez pas lu, et
quand vous passez un répertoire à `--add-dir`, qui apporte ses propres skills, commandes et
agents exactement comme le fait un répertoire de travail.

Toutes ces versions pointent dans la même direction : le CLI rend la **source** et la **portée**
d'un élément de configuration visibles et opposables. C'est le travail que fait à la main la
checklist ci-dessus. Lisez toutefois cette portée honnêtement : rien de tout cela ne lit vos
hooks ni vos surcharges `env` à votre place. L'outillage vous a retiré des tâches de l'assiette,
il ne vous a pas retiré l'assiette.

## Les réflexes à abandonner

- Lire `.claude/` comme « juste de la configuration » plutôt que comme du code.
- Cliquer **trust** par réflexe pour se mettre au travail, en oubliant que le clic persiste.
  C'est la réponse dont héritera chaque exécution headless ultérieure dans ce dossier.
- **Supposer qu'un seul clic de confiance couvrait tout l'arbre.** Il ne couvrait pas les dépôts
  imbriqués avant la v2.1.232, il ne couvre toujours pas tout ce qui se trouve *à l'intérieur*
  du dossier, et aussi tard que la v2.1.265 un dépôt imbriqué pouvait encore agir par un filtre
  git.
- **Utiliser `claude -p` pour « jeter un coup d'œil rapide » à un dépôt inconnu.** Le dialogue
  de confiance n'apparaît jamais en `-p`, et la documentation décrit les serveurs `.mcp.json`
  d'un projet comme *"connected without asking, approved or not."* Depuis la **v2.1.238**, cette
  affirmation générale ne vaut plus pour l'ensemble : un `headersHelper` et les serveurs MCP
  déclarés en ligne sont conditionnés à la confiance enregistrée pour le dossier, y compris en
  `-p`. Mais le reste de la configuration se charge toujours. Passez `--bare`, qui *"skip
  auto-discovery of hooks, skills, custom commands, subagents, plugins, MCP servers, auto
  memory, and CLAUDE.md"* ; la
  documentation est franche sur sa raison d'être, puisque sans lui *"Claude Code runs the hooks
  in a project's `.claude/settings.json` even in a folder you've never trusted."*
- **Attraper un drapeau plus étroit sans vérifier qu'il fait ce que vous croyez.**
  `--setting-sources user` et `--settings '{"disableAllHooks": true}'` sont le bon instinct pour
  une exécution isolée, mais la **v2.1.246** *"Fixed the command sandbox's filesystem
  configuration not respecting `--setting-sources`"* : en dessous de cette version, le drapeau
  n'était pas honoré partout.
- **Désactiver les hooks dans ses propres réglages utilisateur et considérer l'affaire close.**
  D'après la documentation, cela ne suffit pas : les réglages de projet du dépôt priment sur les
  vôtres et peuvent remettre `disableAllHooks` à `false`. Passez-le pour l'exécution en cours.
- Faire tourner une vieille version du CLI « parce que ça marche ». Et sa cousine plus
  subtile : **se croire à jour parce qu'on suit un canal de publication.** Le 10 septembre 2026,
  `stable` est à la v2.1.236 et n'a pas bougé depuis le 19 août, tandis que `latest` est à la
  v2.1.267. Huit des quatorze versions citées dans cette page vivent dans cet écart.
- Copier-coller un `settings.json`, un agent ou un hook trouvé en ligne sans le lire.

## Rester à jour : la parade numéro un

Chacun des problèmes ci-dessus a été résolu par une mise à jour. Garder le CLI à jour n'est pas
de l'entretien courant, c'est votre défense principale. Vérifiez votre version avec
`claude doctor`, et vérifiez *quelle* version, pas simplement qu'une mise à jour a tourné :
comme le montre le tableau en haut de page, les deux canaux npm sont séparés par trois semaines
et trente et un numéros de version. Si vous suivez `stable` pour de bonnes raisons,
continuez, et sachez simplement ce que vous n'avez pas encore.

Pendant que vous y êtes, servez-vous des leviers que Claude Code vous donne pour *réduire* ce
qu'un environnement peut faire. `disallowed-tools` dans le frontmatter d'un skill (v2.1.152)
retire des outils précis, comme l'accès au shell, du pool de Claude tant que ce skill est actif.
Lisez-en la limite : d'après la documentation, *"the restriction clears when you send your next
message."* C'est un garde-fou valable pour une invocation, pas une politique. Pour ce que vous
ne voulez jamais voir un skill atteindre, la documentation est directe, *"To block tools across
all skills and prompts, add deny rules in your permission settings"*, ce qui est
[l'autre moitié de ce réflexe](harden-claude-code-permissions.fr.md). Le moindre privilège vaut
aussi pour les environnements IA.

Pour le travail précis dont parle cette page, regarder un dépôt qu'on n'a pas lu, la
**v2.1.248** a ajouté un instrument plus brutal qui mérite d'être connu :

> *"Added `--restricted` (or `CLAUDE_CODE_RESTRICTED=1`): removes the built-in tools that run
> commands or code and `WebFetch` (unless named in `--tools`), keeps file tools inside the
> working directory, refuses `bypassPermissions`, and ignores user, project and local
> settings files."*

Notez la dernière clause : il ignore purement et simplement les fichiers de réglages du projet,
c'est-à-dire exactement la catégorie de fichiers dont parle cette fiche. Il complète `--bare`
plutôt qu'il ne le remplace : `--bare` saute la découverte, `--restricted` retire les outils
capables d'agir sur ce qui a été découvert. Aucun des deux ne remplace la lecture des sept
points ci-dessus ; les deux vous donnent la marge de les lire.

---

<sub>Dernière revalidation le **2026-09-10** contre Claude Code **v2.1.267**. Cette fiche est
maintenue en parallèle de notre [veille hebdomadaire](../../watch/) : quand l'écosystème bouge,
la page est revérifiée. Les sources sont listées dans l'en-tête de la page.</sub>

<sub>Faire cela à la main sur de nombreux projets devient vite fastidieux.
<a href="https://agence-swel.fr/solutions/tarmhack/">Tarmhack'AI</a> analyse vos projets en
local et rassemble leur configuration Claude Code au même endroit : réglages, chaque hook avec
la commande exacte qu'il exécute, serveurs MCP, et les secrets en dur qu'il trouve, pour que la
revue ci-dessus prenne quelques secondes. Utilisez-le ou non, c'est le réflexe qui compte.</sub>
