---
title: "Durcissez vos propres permissions Claude Code : passez en refus par défaut"
description: "À mesure que Claude Code s'installe sur vos projets, ses permissions glissent silencieusement vers « autoriser ». Voici comment basculer en refus par défaut, écrire des règles dont vous comprenez vraiment la portée, et regarder l'approbation de commande pour ce qu'elle est : gratuite, manuelle, indépendante de tout éditeur."
last_revalidated: 2026-09-10
claude_code_ref: v2.1.267
sources:
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
  - https://code.claude.com/docs/en/permissions
  - https://code.claude.com/docs/en/permission-modes
  - https://code.claude.com/docs/en/settings
  - https://code.claude.com/docs/en/hooks
  - https://code.claude.com/docs/en/security
  - https://www.npmjs.com/package/@anthropic-ai/claude-code?activeTab=versions
lang: fr
---

# Durcissez vos propres permissions Claude Code : passez en refus par défaut

## D'abord : quelle version faites-vous tourner, réellement ?

Avant que le moindre conseil de cette page ait la moindre valeur, regardez ce que vous avez.
Au **2026-09-10**, les dist-tags npm de `@anthropic-ai/claude-code` donnent `stable` à
**v2.1.236** et `latest` à **v2.1.267**, et **`stable` n'a pas bougé depuis le 19 août.**
Vingt-deux versions séparent les deux canaux.

C'est un fait de gouvernance, pas une plainte. Presque tous les durcissements décrits ici (la
parenthèse qui faisait tomber silencieusement une règle de fichier, le lien symbolique échangé
après le contrôle, les règles `Read()` que Grep et Glob n'appliquaient pas) sont arrivés après
la v2.1.236. Si votre équipe épingle `stable`, ou si votre installation dort simplement là
depuis des semaines, **vous ne les avez pas.** Lancez `claude doctor`, lisez la version, et
décidez délibérément sur quel canal vous êtes. Un canal épinglé est un choix légitime ; un
canal subi n'en est pas un.

## Pourquoi le sujet tombe maintenant

Entre la v2.1.211 et la v2.1.226, Anthropic a fermé **plus d'une douzaine de façons distinctes,
pour une commande shell, de passer à travers le contrôle de permission**. Pas un bug : une
douzaine, sur quinze versions, presque toutes dans le même chemin de code. Un échantillon
représentatif, verbatim du changelog officiel :

- **Des commandes trop longues pour être jugées** (v2.1.214) : *"Fixed Bash permission checks
  misjudging very long commands — commands over 10,000 characters now always prompt instead of
  running automatically."*
- **Des redirections que l'analyseur lisait autrement que bash** (v2.1.214) : les contrôles se
  referment désormais en cas de doute sur les *"file-descriptor redirect forms that bash parses
  differently than the permission analyzer."*
- **Des conditionnelles zsh prises pour du texte inerte, deux fois.** La v2.1.214 a corrigé des
  contrôles qui traitaient *"zsh variable subscripts and modifiers in `[[ ]]` comparisons as
  inert text"* ; sept versions plus tard, la v2.1.221 fermait *"a Bash tool permission-check
  bypass where zsh could execute hidden commands in `[[ ]]` regex conditionals."*
- **Des commandes qui se cachent du dialogue que vous êtes en train de lire** (v2.1.223) :
  correction des invites de permission *"so commands padded with tabs or invisible Unicode can no
  longer hide part of the command from the approval dialog"*, livrée avec *"a Bash permission
  bypass where a crafted command could hide parts of itself from permission checks."* Des
  correctifs voisins sont arrivés en v2.1.211 (caractères de forçage bidirectionnel, largeur nulle
  et guillemets sosies dans les aperçus relayés) et en v2.1.216 (Unicode invisible dans la
  validation PowerShell, et frontières de mots non ASCII dans l'analyse Bash).
- **Des commandes qui ont l'air en lecture seule et ne le sont pas** (v2.1.214) : les invocations
  de `help` et `man` qui *"could run unsafe options, command substitutions, or backslash paths"*
  ont cessé de s'auto-approuver. Idem pour les commandes `docker` portant un drapeau de
  redirection de démon (`--url`, `--connection`, `--identity`), et pour `file` employé avec
  `-m`/`--magic-file` ou `-f`/`--files-from`.
- **Des bugs de portée dans les règles elles-mêmes** (v2.1.214) : *"Fixed single-segment `dir/**`
  allow rules like `Edit(src/**)` auto-approving writes to nested `dir/` directories anywhere in
  the tree instead of only `<cwd>/dir`."* On y revient plus bas : c'est le point le plus
  actionnable de cette page.

### Et ça n'en est pas resté là : v2.1.227 → v2.1.267

Six semaines plus tard, le même chemin de code produit toujours la même forme de constat. Les
plus récents sont même plus instructifs, parce que plusieurs d'entre eux cassaient **les règles
elles-mêmes** plutôt que l'analyseur de commandes :

- **Une règle peut être jetée en silence à cause d'un caractère dans son chemin** (v2.1.260) :
  *"Fixed `Edit`/`Write`/`Read` permission rules whose path contains parentheses being dropped as
  invalid or ignored by the Bash sandbox, which left "read-only" folders writable."* Relisez la
  fin. La règle était dans le fichier. Le dossier était inscriptible.
- **Une seule mauvaise règle pouvait casser toutes les éditions** (v2.1.260) : *"Fixed one file
  permission rule with an uncompilable pattern (e.g. an unclosed `[`) making every file edit fail
  with `Invalid regular expression`; such a deny rule now guards the literal path it spells."*
- **Une règle qui n'attrapait rien était acceptée en silence** (v2.1.260) : *"Changed permission
  rules with text after the closing parenthesis (e.g. `Bash(ls) x`), which never matched anything,
  to be reported as invalid settings instead of being silently ignored."*
- **Le fichier pouvait changer entre le contrôle et l'écriture** (v2.1.251) : *"Fixed file tools
  (Read, Write, Edit) following a symlink swapped inside the working directory after the permission
  check, which could read or write outside the approved location."*
- **Deux outils n'appliquaient pas du tout vos règles de refus** (v2.1.251) : *"Fixed Grep and Glob
  not applying `Read(...)` deny rules to files reached through a symlinked search path."*
- **L'arithmétique comme contournement** (v2.1.251) : *"Fixed Bash permission checks auto-approving
  commands that assign an arithmetic expression to an integer shell variable (e.g. `OPTIND=1/0`,
  `RANDOM=2+2`); these now prompt for approval."*
- **zsh, une cinquième fois** (v2.1.260) : *"Fixed Bash permission checks auto-approving zsh
  commands that hide a command substitution in a REPORTTIME, REPORTMEMORY or DIRSTACKSIZE
  assignment; these now prompt for approval."*
- **Une règle `ask` qui ne demandait pas** (v2.1.257) : *"Fixed a `permissions.ask` rule being
  skipped in auto mode when the matching command ran inside a compound command or subshell, letting
  it run without the confirmation prompt."*

### La direction tient depuis juin

Ce n'est pas une panique soudaine. Le même thème est livré régulièrement.

Début juin (v2.1.160 → v2.1.168), le refus par défaut est devenu bon marché : motifs glob
acceptés en position de nom d'outil dans les règles de refus (v2.1.166) ; demande de confirmation
avant qu'`acceptEdits` n'écrive une configuration d'outil de build capable d'exécuter du code
(`.npmrc`, `.yarnrc*`, `bunfig.toml`, `.bazelrc`, `.pre-commit-config.yaml`, `.devcontainer/`),
plus les fichiers de démarrage du shell et `~/.config/git/` (v2.1.160) ; et les réglages managés
`requiredMinimumVersion` / `requiredMaximumVersion`, qui font refuser à Claude Code de démarrer
hors de la plage autorisée (v2.1.163).

Fin juin (v2.1.178 → v2.1.195), le moindre privilège est descendu d'un étage :

- **Les règles sont devenues paramétriques** (v2.1.178) : *"Added `Tool(param:value)` syntax for
  permission rules to match a tool's input parameters (with `*` wildcard), e.g. `Agent(model:opus)`
  to block Opus subagents."* La v2.1.186 a fermé un trou où les règles de refus `Agent(type)` et
  les restrictions de types autorisés `Agent(x,y)` étaient *"not being enforced for named subagent
  spawns."*
- **`sandbox.credentials`** (v2.1.187) empêche les commandes exécutées en bac à sable de lire vos
  fichiers d'identifiants et vos variables d'environnement secrètes.
- **Les refus sont devenus lisibles** (v2.1.193) : les motifs de refus du mode auto apparaissent
  dans le transcript, dans la notification de refus, et sous les refus récents de `/permissions`.
- **Un filet pour les commandes destructrices** (v2.1.183) : le mode auto bloque `git reset --hard`,
  `git clean -fd` et `terraform`/`pulumi`/`cdk destroy` quand vous n'avez pas demandé à jeter du
  travail.

Le fil s'est prolongé tout l'été. Celui qu'il faut retenir : la v2.1.235 a corrigé les dialogues de
permission, dont *"display text and "don't ask again" options now always match what a grant would
cover, and "don't ask again" is withheld when contents cannot be fully displayed."* Jusque-là, un
clic sur « ne plus demander » pouvait accorder davantage que ce que le dialogue vous avait montré.

Deux ans de notes de version qui pointent dans la même direction, ce n'est pas du bruit. C'est une
position de conception. Adoptez-la.

## L'approbation de commande est un filet, pas un mur

Voici la lecture honnête de ces listes. Décider si une commande shell est sûre suppose d'analyser
du shell, et analyser du shell *exactement comme le shell le fera* est un problème réellement
difficile. L'analyseur de permissions est une seconde implémentation qui court après bash, zsh et
PowerShell. Presque tous les trous ci-dessus sont le même trou : l'analyseur a lu la commande
autrement que le shell ne l'aurait fait. Les constats les plus récents élargissent le propos d'un
cran : l'analyseur peut aussi lire **votre fichier de règles** autrement que vous ne l'avez lu
(les parenthèses, v2.1.260), et le système de fichiers peut changer sous une décision déjà prise
(le lien symbolique échangé, v2.1.251).

Anthropic ferme ces trous vite, et les ferme de la bonne manière : la v2.1.214 fait échouer en
mode fermé les formes de redirection ambiguës, la v2.1.246 exige désormais *"always require
approval for malformed commands with a dangling `&&` or `||` operator"*, et la v2.1.267 corrige les
réglages managés `allowedHttpHookUrls`, `httpHookAllowedEnvVars` et `allowedChannelPlugins` *"to
admit nothing, not everything, when unreadable."* Se refermer sur ce qu'on n'arrive pas à analyser
est la bonne valeur par défaut, et elle est appliquée avec constance. Mais le point structurel
reste, et Anthropic l'écrit noir sur blanc dans sa propre documentation des hooks, à propos du
filtre `if` qui conditionne un hook à la forme d'une commande :

> *"The filter also fails open, running your hook regardless of pattern, when the Bash command
> can't be parsed. Because the `if` filter is best-effort, use the permission system rather
> than a hook to enforce a hard allow or deny."*

Deux conséquences en découlent, et les deux méritent une action aujourd'hui.

**Une liste d'autorisation étroite vaut mieux qu'une liste d'interdiction astucieuse.** Une liste
d'interdiction doit anticiper toutes les orthographes de toutes les commandes dangereuses : chaque
astuce de quoting, chaque dialecte de shell, chaque imbrication de `$()`. Une liste d'autorisation
n'a qu'à énumérer la poignée de choses que vous faites vraiment. Quand l'analyseur est le maillon
faible, c'est le côté où vous rangez « inconnu » qui décide du résultat. Rangez-le du côté du refus.

**Le dialogue d'approbation est une interface, et une interface, ça se trompe.** Trois versions
distinctes ont corrigé des caractères qui changeaient ce que l'humain voyait par rapport à ce qui
allait s'exécuter. Donc quand une commande est longue, dense ou bizarrement formatée, l'invite est
le pire moment possible pour trancher, et vous avez maintenant un appui de moins : la v2.1.257
*"Removed the Ctrl+E command explanation on Bash and PowerShell permission prompts."* Si vous
aviez pris l'habitude d'appuyer sur `Ctrl+E` pour obtenir une lecture de risque Faible / Moyen /
Élevé sur une commande que vous ne compreniez pas, cette habitude n'a plus rien sous le doigt. Ce
qui ne fait qu'aiguiser le vrai conseil : ne comptez pas sur votre jugement à chaud pour les
catégories dont vous savez déjà que vous ne les approuverez jamais. Écrivez-les une fois, comme
règles.

## La portée d'une règle peut bouger sous vos pieds, dans les deux sens

C'est la leçon la plus récente de cette page, et celle qui a le plus de chances de mordre un
lecteur soigneux, parce qu'elle punit exactement celui qui a écrit de bonnes règles et est passé
à autre chose.

En **v2.1.259**, le traitement des arguments Bash a été resserré : les règles de refus `Read()`
se sont mises à couvrir les *"files given as option values (`--ignore-revs-file=.env`, `-f.env`,
`@file`), `git diff`/`git grep` file operands, or `cd DIR && cat FILE` compounds."* Strictement
plus protecteur. Puis, une version plus tard, **la v2.1.260 est revenue dessus** :

> *"Reverted the 2.1.259 change applying `Read()` deny rules to Bash arguments; it denied `npm
> run build` under a `Read(./**/build/**)` rule in every mode and made `cd … && grep` prompt
> even in auto mode."*

Notez ce que dit le motif de la révocation : une règle de refus parfaitement ordinaire,
`Read(./**/build/**)`, s'est mise à bloquer `npm run build`. Le resserrage était juste dans
l'intention et faux dans son rayon d'effet, et il est arrivé puis reparti en deux versions.

Ce n'est pas la première fois sur cette surface. La v2.1.232 annonçait *"Bash input redirections
(`< file`) are now permission-checked like their argument spellings on all platforms"* ; la
v2.1.233 a *"Reverted the 2.1.232 Bash permission changes for Cygwin-style symlinks on Windows
and for input redirections (`< file`); a narrower version will return in a later release"* ; et en
v2.1.257 la version plus étroite est effectivement revenue : *"Fixed Bash `Read()`/`Edit()` deny
rules not applying to `< file` redirects and reader commands like `tac` and `egrep`; a deny rule on
any argument or redirect target now refuses the command."* Sortie, puis revenue, en vingt-cinq
versions.

Trois conséquences pratiques :

1. **Une règle qui marchait le mois dernier peut être plus large ou plus étroite aujourd'hui**, et
   aucune des deux directions n'est annoncée avant d'atterrir. Plus large, votre build casse ; plus
   étroite, votre plancher de refus a discrètement un trou.
2. **Une règle protectrice demande donc un test de vivacité périodique**, pas une écriture unique.
   Une fois par trimestre, ou après chaque mise à jour que vous remarquez, provoquez-la et
   vérifiez qu'elle se déclenche toujours.
3. **Lisez les notes de version pour les révocations, pas seulement pour les correctifs.** Une
   ligne de revert est le seul endroit où le retrait d'une restriction soit jamais consigné.

## Le problème, en clair

Quand vous menez un seul projet, vous lisez chaque demande de permission. Quand vous en menez dix,
vous arrêtez. Vous cliquez **autoriser**, vous passez en `acceptEdits` « pour ne plus être
interrompu », et sur quelques mois votre posture dérive : trop de choses sont permises par défaut,
et vous ne voyez plus ce qui est réellement *refusé*. C'est ça, le mode de défaillance. Pas une
brèche spectaculaire : une accumulation lente et invisible de confiance que vous n'avez jamais
réexaminée.

Le correctif est une posture, pas un outil : **partez du refus, et accordez le minimum dont
chaque projet a besoin.** C'est le même réflexe de moindre privilège que vous appliqueriez à un
jeton de CI ou à un compte de service. L'outillage IA l'a mérité le jour où sa configuration a pu
exécuter du code sur votre machine.

## Basculer en refus par défaut

C'est une modification gratuite et manuelle de votre propre `settings.json`. Rien à installer.

1. **Auditez ce que vous autorisez aujourd'hui, et sachez où ça vit.** Lisez `permissions.allow`
   dans vos fichiers de niveau utilisateur (`~/.claude/settings.json`) et de niveau projet
   (`.claude/settings.json`) comme si quelqu'un d'autre les avait écrits. Pour chaque entrée : est-ce
   que *ce* projet en a réellement besoin ? La plupart n'ont pas besoin de la plupart. Notez que
   depuis la **v2.1.211**, une approbation « Oui, ne plus demander » est enregistrée dans
   `.claude/settings.local.json` **à la racine du dépôt git**, résolue à travers les worktrees
   jusqu'au checkout principal : elle s'applique donc désormais aux sous-répertoires et aux
   worktrees, là où elle était auparavant confinée au répertoire de démarrage. Si vous accumuliez
   des approbations depuis vos worktrees, elles ont déménagé. Allez lire ce fichier.

2. **N'essayez pas de « tout refuser, puis ré-autoriser ».** C'est le geste intuitif et il ne
   fonctionne pas. `permissions.deny` accepte bien `"*"` en position de nom d'outil, mais la
   référence des permissions est explicite : *"a tool matched by a bare-name glob deny rule is
   removed from Claude's context, the same as a bare tool name."* Et les règles sont *"evaluated in
   order: deny, then ask, then allow. The first match in that order determines the outcome, and
   rule specificity doesn't change the order"*, donc *"a deny rule can't carry allowlist
   exceptions."* `"deny": ["*"]` ne vous donne pas un plancher sur lequel construire : il retire
   chaque outil sans chemin de retour. Si vous voulez une vraie posture de liste d'autorisation,
   utilisez le mode fait pour ça : `permissions.defaultMode: "dontAsk"` *"auto-denies every tool
   call that would otherwise prompt you,"* ne laissant que vos règles `permissions.allow`, les
   commandes Bash en lecture seule intégrées, et les appels approuvés par un hook PreToolUse. Il
   est conçu pour la CI et les environnements verrouillés, et c'est un moyen rapide et honnête de
   découvrir à quel point votre vraie liste d'autorisation est courte.

3. **Posez ensuite un plancher de refus sur ce qui ne doit jamais arriver.** Les règles de refus
   ciblées sont le contrôle le plus durable dont vous disposiez, parce qu'elles sont la seule chose
   qui survive à tous les modes : selon la référence des modes, les règles de refus et les règles
   `ask` explicites *"apply in every mode, including `bypassPermissions`,"* tandis que *"allow rules
   have no effect in `bypassPermissions`."* Si vous n'écrivez qu'un seul contrôle cette année,
   écrivez celui-là. Une règle `deny` ou `ask` qui nomme un outil inexistant produit un
   avertissement au démarrage, donc cette faute de frappe-là remonte au lieu de ne rien faire en
   silence, et depuis la v2.1.260 c'est aussi le cas d'une règle portant du texte parasite après sa
   parenthèse fermante, qui était jusque-là acceptée puis ignorée. L'avertissement ne couvre pas
   tout pour autant : voyez plus bas les deux caractères qui peuvent annuler une règle sans un mot.

4. **Sachez jusqu'où va une règle de refus, et que cette limite bouge.** Les règles de refus Read
   et Edit s'appliquent aux outils de fichier intégrés de Claude et aux commandes de fichier que
   Claude Code reconnaît dans Bash. Cet ensemble reconnu est plus large qu'avant : la v2.1.257 l'a
   étendu aux redirections `< file` et à des lecteurs comme `tac` et `egrep`, et la v2.1.251 a fait
   honorer les règles de refus `Read(...)` par Grep et Glob à travers un chemin de recherche
   symlinké. Il est aussi plus étroit que ce que la v2.1.259 en avait brièvement fait : voyez la
   section ci-dessus. Ce qui n'a pas changé, c'est la frontière externe : d'après la documentation,
   ces règles *"don't apply to arbitrary subprocesses that read or write files indirectly, like a
   Python or Node script that opens files itself."* Pour un contrôle qui tienne face à n'importe
   quel processus, c'est le bac à sable qui est fait pour ça. Savoir où s'arrête un garde-fou fait
   partie de son usage.

5. **Envisagez de bloquer les lectures hors de vos répertoires de travail.** La v2.1.257 a ajouté
   *"a one-time prompt in auto mode before the first file read outside the working directories,
   with the option to block such reads (`permissions.blockReadsOutsideWorkingDirectories`)."* C'est
   le réglage neuf le plus rentable de cette page : il convertit « Claude peut lire tout mon
   répertoire personnel sauf si une règle de refus dit le contraire » en « Claude lit le projet, et
   demande une fois avant le reste ». Activez-le, puis attendez-vous à accorder deux ou trois
   exceptions. La v2.1.260 l'a corrigé *"on macOS hiding the user's git config from sandboxed git
   and hiding a worktree-isolated sub-agent's own checkout"*, alors assurez-vous d'être sur une
   version postérieure avant de le juger.

6. **Ne faites pas tourner `acceptEdits` partout.** C'est commode et ça se retourne contre vous.
   Depuis la v2.1.160, le mode demande confirmation avant d'écrire les fichiers de configuration qui
   accordent l'exécution de code, mais la bonne habitude durable est de le cantonner au travail
   jetable ou en bac à sable, pas d'en faire votre défaut sur toute la machine. Point voisin, utile
   quand vous héritez du dépôt de quelqu'un d'autre : depuis la v2.1.257, un `defaultMode:
   "bypassPermissions"` posé *"in `.claude/settings.json` or `.claude/settings.local.json`"* est
   *"ignored, like `"auto"`; set it in user or managed settings, or pass `--permission-mode`."* Un
   fichier de projet versionné ne peut plus mettre votre session en mode bypass.

7. **Rendez vos règles `WebFetch` explicites.** Depuis la v2.1.162, une règle
   `WebFetch(domain:...)` explicite en deny, ask ou allow l'emporte sur l'auto-autorisation des
   hôtes pré-approuvés. Notez que le joker est délibérément conservateur :
   `WebFetch(domain:example.*)` attrape `example.org` mais *pas* `example.evil.com`, car hors d'un
   `*.` initial le joker ne franchit pas un point. C'est un choix de conception qui empêche un
   joker final d'attraper un domaine qu'un attaquant pourrait déposer.

8. **Ciblez vos règles de refus sur des paramètres, pas seulement sur des outils.** Depuis la
   v2.1.178, vous pouvez viser le paramètre d'entrée de premier niveau d'un outil avec
   `Tool(param:value)` : `Agent(model:opus)`, `Agent(isolation:worktree)`,
   `Bash(run_in_background:true)`, avec `*` comme joker. **Cela ne fonctionne que pour les règles
   deny et ask.** La documentation est explicite : *"an allow rule for one parameter value wouldn't
   establish that the call is safe overall, so allow rules continue to use each tool's own specifier
   syntax."* Une réserve : si vous avez écrit un refus `Agent(type)` ou une règle de types autorisés
   `Agent(x,y)` avant la v2.1.186, revérifiez-la, ces règles n'étaient pas appliquées aux lancements
   de sous-agents nommés avant ce correctif.

9. **Pour les exécutions sans surveillance, utilisez les drapeaux faits pour ça plutôt que
   d'assouplir vos réglages.** Deux sont arrivés cet été, et ce sont la bonne réponse à « l'agent
   s'arrête tout le temps en CI » :

   - **`--restricted`** (v2.1.248, ou `CLAUDE_CODE_RESTRICTED=1`) *"removes the built-in tools that
     run commands or code and `WebFetch` (unless named in `--tools`), keeps file tools inside the
     working directory, refuses `bypassPermissions`, and ignores user, project and local settings
     files."* Lisez la dernière clause à la lettre : les réglages mis de côté sont ceux de niveau
     **utilisateur, projet et local**. **Les réglages managés ne figurent pas dans cette liste et
     continuent de s'appliquer**, ce qui est bien ce qu'on veut, puisque c'est là que vit le
     plancher d'une organisation. `--restricted` ne veut pas dire « aucune configuration n'est
     lue » : il veut dire « vos trois niveaux de configuration à vous sont mis de côté ».
   - **`--permission-prompts none`** (v2.1.259) *"for unattended headless hosts: anything that would
     prompt is denied automatically while the active permission mode (including auto mode) keeps
     deciding."* Le refus par défaut sous forme de drapeau de lancement, pour la seule situation où
     personne n'est là pour répondre.

10. **En équipe, épinglez une fenêtre de versions.** `requiredMinimumVersion` /
    `requiredMaximumVersion` (v2.1.163) dans les réglages managés, pour que personne ne fasse
    tourner en silence une version antérieure à un correctif. Vu combien de correctifs de cet été
    portaient sur le contrôle de permission, et vu l'écart de 22 versions entre `stable` et `latest`
    relevé en tête de page, c'est le réglage qui transforme « on est à jour » d'un espoir en un fait.

11. **En équipe, faites échouer bruyamment une politique illisible.** La v2.1.259 a corrigé
    *"managed settings silently going unenforced when the managed-settings file, a drop-in, the MDM
    plist, or the HKLM value cannot be parsed: Claude Code now refuses to start and names the
    source."* Il n'y a rien à configurer : c'est un comportement dont vous héritez en étant à jour.
    Mais il vaut la peine de connaître le mode de défaillance qu'il remplace : une faute de frappe
    dans la politique de votre organisation valait *aucune* politique, en silence, sur chaque poste.
    Si votre parc est épinglé sous la v2.1.259, validez ce fichier à la main.

12. **En équipe, verrouillez les modèles qui peuvent tourner.** Posez une liste `availableModels`
    dans les réglages managés et mettez `enforceAvailableModels: true` (v2.1.175) : la liste *"also
    constrains the Default model,"* et *"user or project settings can no longer widen a managed
    `availableModels` list."* La v2.1.172 l'a rendue applicable partout : surcharges de modèle des
    sous-agents, sélecteur de dispatch, modèle conseiller. Ça compte parce qu'un sous-agent peut
    lancer ses propres sous-agents (profondeur 3 par défaut depuis la v2.1.219) ; sans application
    de la règle jusqu'à ces surcharges, un agent imbriqué pourrait choisir un modèle que vous
    n'avez jamais approuvé.

13. **Activez `sandbox.credentials`.** Si vous exécutez des commandes en bac à sable, posez-le
    (v2.1.187) pour qu'elles ne puissent pas lire vos fichiers d'identifiants ni vos variables
    d'environnement secrètes. C'est gratuit, et ça ferme un chemin d'exfiltration évident.

## Sachez ce que vos motifs attrapent réellement

C'est la partie que la plupart des gens ratent, et la v2.1.214 explique pourquoi elle vaut dix
minutes de votre temps. **Le même motif n'a pas la même portée partout.** Les règles Read et Edit
utilisent la syntaxe [gitignore](https://git-scm.com/docs/gitignore), et pour un motif relatif à
segment de répertoire unique, la profondeur d'appariement dépend du type de règle.

Sur un projet qui a un `src/` à la racine et une copie imbriquée dans `vendor/pkg/src/` :

| Règle | Attrape `src/app.ts` | Attrape `vendor/pkg/src/lib.js` |
| :--- | :--- | :--- |
| `Edit(src/**)` en règle **allow** | Oui | **Non** |
| `Edit(src/**)` en règle **deny ou ask** | Oui | **Oui** |
| `Edit(/src/**)` quel que soit le type | Oui | Non |
| `Edit(**/src/**)` quel que soit le type | Oui | Oui |

Cette asymétrie est délibérée et c'est le bon arbitrage : **les règles d'autorisation ont été
resserrées, les règles de refus sont restées larges.** Une règle d'autorisation qui attrape plus
que prévu est un trou ; une règle de refus qui attrape plus que prévu est simplement conservatrice.
Sûr par construction. Mais cela veut dire que vous ne pouvez pas raisonner sur `src/**` sans savoir
dans quelle liste il se trouve.

Et il existe une troisième portée pour la même chaîne. Dans une **condition `if:` de hook**, la
v2.1.214 a changé les motifs à segment unique pour qu'ils n'attrapent que `<cwd>/dir` ; pour
attraper à n'importe quelle profondeur, il faut écrire `**/dir/**`. Le changelog énonce la coupure
directement :

> *"Changed single-segment `dir/**` hook `if:` conditions to match only `<cwd>/dir`; write
> `**/dir/**` for any-depth matching. `deny`/`ask` permission rules keep their any-depth
> match."*

Donc `Edit(src/**)` veut dire trois choses différentes selon qu'il s'agit d'une règle
d'autorisation, d'une règle de refus ou d'une condition de hook. Si vous avez copié un motif d'un
endroit à l'autre, et tout le monde le fait, allez le relire maintenant.

Quatre autres règles d'ancrage à intégrer, toutes tirées de la référence des permissions :

- **Une seule barre oblique initiale n'est pas un chemin absolu.** `/path` s'ancre sur la *source
  du réglage* : `<racine du projet>/path` dans les réglages de projet, `~/.claude/path` dans les
  réglages utilisateur. Pour un vrai chemin absolu, écrivez `//path` ; pour un chemin relatif au
  répertoire personnel, `~/path`. Le piège fait le plus mal dans les réglages utilisateur : une
  règle de refus `Read(/secrets/**)` posée là bloque `~/.claude/secrets/**`, **pas** un répertoire
  `secrets` dans votre projet.
- **Un nom de fichier nu attrape à n'importe quelle profondeur.** `Read(.env)` et `Read(**/.env)`
  sont équivalents.
- **Refusez les écritures de fichier avec `Edit(...)`, pas avec `Write(...)`.** Une règle de refus
  `Write(docs/**)` ne fait pas ce qu'elle a l'air de faire, et depuis la v2.1.210 Claude Code vous
  le dit au démarrage : cette version a *"Added a startup warning for `Write(path)`,
  `NotebookEdit(path)`, and `Glob(path)` permission rules — use `Edit(path)` or `Read(path)`
  instead."* Les règles `Edit` couvrent tous les outils d'édition de fichier.
- **Sous Windows, les chemins sont normalisés en forme POSIX** avant appariement : `C:\Users\alice`
  devient `/c/Users/alice`. Utilisez `//c/**/.env` pour un seul lecteur, `//**/.env` pour tous.

### Deux caractères qui peuvent annuler une règle purement et simplement

Deux choses valent une vérification à l'œil, tout de suite, dans chaque fichier de réglages qui
vous appartient.

**Une parenthèse n'importe où dans le chemin.** Jusqu'à la v2.1.260, une règle `Edit`/`Write`/`Read`
dont le chemin contenait `(` ou `)` pouvait être *"dropped as invalid or ignored by the Bash
sandbox, which left "read-only" folders writable."* Sous Windows, ce n'est pas un cas exotique :
`C:\Program Files (x86)\...` et `.../Mon Projet (ancien)/...` sont des chemins ordinaires. La même
version a amélioré le diagnostic *"for rules such as `Edit(C:\dir\(name)\**)`, where `\(` is read as
an escaped parenthesis rather than a path separator, to suggest an unambiguous spelling."* Si une de
vos règles protectrices nomme un chemin comportant une parenthèse, considérez-la comme non prouvée
tant que vous ne l'avez pas vue se déclencher sur une version v2.1.260 ou postérieure.

**Un crochet non fermé dans une seule règle.** Avant la v2.1.260, une règle unique portant un motif
non compilable faisait *"every file edit fail with `Invalid regular expression`."* Une faute de
frappe, toutes les éditions à terre. Le comportement se dégrade désormais proprement, *"such a deny
rule now guards the literal path it spells"*, mais l'ancien comportement reste un symptôme utile à
reconnaître : si les éditions se mettent à échouer globalement juste après que vous avez touché aux
permissions, cherchez un crochet, pas un bug.

### Un joker placé avant la sous-commande attrape plus que vous ne croyez

La v2.1.246 *"Added a startup warning for Bash allow rules with a wildcard before the subcommand
(e.g. `Bash(git * main)`), since they also match options inserted before the subcommand."*

Le défaut, ici, n'est pas qu'une telle règle ne fasse rien : c'est qu'elle en fait **plus** que ce
que son auteur voulait. Tout ce qui tient entre `git` et `main`, y compris des options que vous
n'avez jamais eu l'intention de bénir, satisfait le motif. C'est une règle d'autorisation avec un
trou au milieu que personne n'a examiné, et elle se lit parfaitement naturellement. Gardez vos
jokers après la sous-commande, là où la forme de la commande est déjà fixée.

## Anti-patterns à abandonner

- **Une autorisation par défaut que vous n'avez jamais auditée.** Si vous ne savez pas dire ce que
  votre configuration refuse, elle ne refuse rien.
- **`acceptEdits` comme défaut global.** Une commodité qui écrit à votre place des fichiers
  accordant l'exécution de code n'est pas une commodité.
- **Faire confiance à une règle que vous n'avez jamais vue se déclencher.** Une règle peut être
  syntaxiquement valide et sémantiquement inerte, et les preuves continuent d'arriver. La v2.1.224
  a corrigé des entrées de refus du bac à sable *"written with a trailing slash (e.g. `denyRead:
  "~/.aws/"`) being silently bypassable on Linux and macOS."* La v2.1.257 a corrigé des hôtes réseau
  du bac à sable *"written with a trailing dot (`example.com.`)"*, où une entrée `deniedDomains`
  *"didn't block the host inside the sandbox."* La v2.1.260 a corrigé le cas de la parenthèse
  ci-dessus. Une barre finale, un point final, une parenthèse. La leçon se généralise : après avoir
  écrit une règle protectrice, provoquez-la une fois et vérifiez qu'elle se déclenche, puis
  provoquez-la de nouveau après une mise à jour.
- **Supposer que la portée d'une règle est stable.** Elle ne l'est pas, dans les deux sens : voyez
  l'aller-retour v2.1.259 / v2.1.260 plus haut. Retestez vos règles protectrices à intervalle
  régulier, et lisez les lignes de revert des notes de version aussi attentivement que les lignes
  de correctif.
- **Essayer de cadrer Bash par paramètre.** `Bash(command:rm *)` a l'air raisonnable et se fait
  ignorer avec un avertissement au démarrage, parce qu'une commande composée le contournerait.
  Écrivez `Bash(rm *)`. Même chose pour le champ de contenu principal de Read, Edit, Write, Grep,
  Glob, NotebookEdit et WebFetch.
- **S'appuyer sur une condition `if:` de hook comme mécanisme d'application.** C'est un filtre au
  mieux, qui s'ouvre en grand sur les commandes qu'il n'arrive pas à analyser. La recommandation
  d'Anthropic elle-même est de passer par le système de permissions pour une autorisation ou un
  refus ferme.
- **Faire tourner une vieille version « parce que ça marche ».** Plus d'une douzaine de correctifs
  de contrôle de permission sont sortis en quinze versions cet été, et les six semaines suivantes en
  ont produit une nouvelle série. Ils ne vous protègent que si vous les avez. Vérifiez avec
  `claude doctor`, et rappelez-vous que `stable` et `latest` sont à 22 versions d'écart au moment
  de cette revalidation ; une fenêtre de versions (v2.1.163) en fait un réglage d'équipe plutôt
  qu'un espoir.
- **Croire qu'une liste de modèles managée peut être élargie en local.** Depuis la v2.1.175, les
  réglages de projet ou d'utilisateur ne peuvent plus élargir une liste `availableModels` managée.
  Si vous la posez comme plancher, traitez-la comme un plancher.
- **Ne jamais lire ce qui a été refusé.** Depuis la v2.1.193, les refus apparaissent dans le
  transcript, dans une notification, et sous les refus récents de `/permissions` ; et depuis la
  v2.1.234, vous pouvez ouvrir `/permissions` pendant que Claude travaille, les changements de
  règles s'appliquant au reste du tour en cours. Un plancher de refus que vous ne regardez jamais
  est une supposition ; la liste des refus récents vous dit s'il attrape ce que vous vouliez, ou
  s'il bloque du vrai travail et vous entraîne à l'élargir.
- **Attendre de l'invite qu'elle vous explique une commande.** `Ctrl+E` a disparu en v2.1.257. Si
  vous ne savez pas lire une commande à l'invite, la réponse est de la refuser et d'aller la lire
  dans un éditeur, jamais de l'approuver pour voir.
- **Copier un `settings.json` trouvé en ligne sans le lire.** Même réflexe que traiter [la
  configuration d'un dépôt cloné comme du code non fiable](untrusted-repo-config.fr.md), sauf
  qu'ici, c'est *votre* posture que vous confiez à un inconnu.

## Un dernier point, pour les configurations multi-sessions

Si vous faites tourner plusieurs sessions Claude qui se parlent, notez la v2.1.166 : les messages
relayés par `SendMessage` depuis une autre session **ne portent plus l'autorité de l'utilisateur**,
le destinataire refuse les demandes de permission relayées. Il n'y a rien à configurer, c'est
simplement vrai maintenant. La surface a continué de se durcir en grandissant : la v2.1.222 fait
passer `SendMessage` par le classifieur de permissions avant expédition, la v2.1.224 a ajouté les
réglages `crossSessionInbound` et `dialogExpiry` pour que les messages envoyés à une session dont
les permissions sont contournées soient retenus en attente de votre approbation, et la v2.1.225 a
ajouté une invite de confiance d'espace de travail à `claude agents` pour les répertoires non
fiables, à l'image de ce que `claude` faisait déjà.

Deux suites si vous posez `crossSessionInbound`. La v2.1.248 a corrigé *"an invalid
`crossSessionInbound` value being silently ignored: it now warns and holds cross-session messages
(user settings) or refuses them (managed settings) until fixed"*, encore un cas de règle inerte qui
ne le disait pas. Et la v2.1.234 a corrigé *"session-scoped permission answers (including denies)
being dropped when answering background subagent tool permission prompts"*, si bien qu'un refus
donné à un agent d'arrière-plan tient désormais réellement pour la session.

Votre plancher de refus ne peut pas être levé par un message venu d'une session sœur. C'est le
modèle qui travaille *avec* le refus par défaut, pas autour.

---

<sub>Dernière revalidation le **2026-09-10** contre Claude Code **v2.1.267**. Cette fiche est tenue
à jour en parallèle de notre [veille hebdomadaire](../../watch/) : quand l'écosystème bouge, la
page est revérifiée. Les sources sont listées dans l'en-tête de page.</sub>

<sub>Auditer tout cela à la main sur de nombreux projets devient vite fastidieux.
<a href="https://agence-swel.fr/solutions/tarmhack/">Tarmhack'AI</a> lit la configuration Claude
Code de vos projets en local et met à plat, au même endroit, ce que chacun autorise et refuse :
repérer le projet resté en autorisation par défaut prend quelques secondes au lieu de dix
ouvertures de fichier. Utilisez-le ou non, c'est la posture qui compte.</sub>
