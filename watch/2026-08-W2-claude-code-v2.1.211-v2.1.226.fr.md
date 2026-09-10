---
title: "Claude Code - Ce qui change cette semaine (v2.1.211 → v2.1.226)"
date: 2026-08-10
period: "Semaine W2 · v2.1.211 → v2.1.226"
sources:
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
lang: fr
---

# Claude Code - Ce qui change cette semaine

> Période : semaine W2 (août 2026) · Versions v2.1.211 → v2.1.226 (16 versions)
> Sources : [CHANGELOG.md GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [code.claude.com](https://code.claude.com/docs/en/changelog)
> Généré le 2026-08-10

> **Statut de stabilité - confirmé ce run.** Les dist-tags npm sont cohérents avec le changelog officiel : `stable 2.1.220` / `latest 2.1.226`. Autrement dit, tout ce qui est ≤ v2.1.220 est déjà sur le canal stable ; ce qui est ≥ v2.1.221 est encore en préparation et peut bouger. Vérifiez `claude --version` et votre canal avant de déployer en production.

---

## Ce qui compte vraiment

### Claude Opus 5 devient le modèle Opus par défaut - v2.1.219

La fenêtre de contexte passe à un million de tokens, et le fast mode change de titulaire.

> *"Added Claude Opus 5 (`claude-opus-5`), now the default Opus model — 1M context, fast mode at $10/$50 per Mtok"*
>
> *"Removed Opus 4.7 from fast mode; `/fast` now applies to Opus 5 and Opus 4.8"*

**Ce que ça change concrètement** : vous pouvez charger beaucoup plus de code, de logs ou de documentation dans une seule session avant que le modèle ne perde le fil. Sur un gros dépôt, c'est la différence entre découper son problème et le donner d'un bloc. Si vous épingliez un ancien modèle Opus, regardez votre picker `/model` : la ligne Opus est désormais fusionnée et affichée « Opus (1M context) ». Si vous laissez le défaut, il n'y a rien à faire.

---

### Le mode plan ne lance plus de commandes qui modifient vos fichiers - v2.1.212

Le mode plan est censé réfléchir, pas agir. Ce n'était pas tout à fait vrai.

> *"Fixed plan mode auto-running file-modifying Bash commands (e.g. `touch`, `rm`) without a permission prompt or SDK `canUseTool` callback"*

**Ce que ça change concrètement** : un `rm` ou un `touch` pouvait passer sans prompt d'approbation ni callback `canUseTool`. C'est fermé. Si vous utilisez le mode plan pour cadrer avant d'exécuter, vous pouvez de nouveau vous y fier sur ce point précis. Le correctif s'applique tout seul à la mise à jour, il n'y a rien à configurer.

---

### Un motif `dir/**` n'a plus la même portée selon où vous l'écrivez - v2.1.214

C'est le changement le plus discret de la semaine, et celui qui a le plus de chances de vous piéger.

> *"Fixed single-segment `dir/**` allow rules like `Edit(src/**)` auto-approving writes to nested `dir/` directories anywhere in the tree instead of only `<cwd>/dir`"*
>
> *"Changed single-segment `dir/**` hook `if:` conditions to match only `<cwd>/dir`; write `**/dir/**` for any-depth matching. `deny`/`ask` permission rules keep their any-depth match."*

**Ce que ça change concrètement** : deux choses, et il faut les tenir ensemble. D'abord, une règle comme `Edit(src/**)` approuvait par erreur les écritures dans *n'importe quel* dossier `src/` de l'arborescence, pas seulement celui de votre projet - c'est corrigé, et c'est une bonne nouvelle. Ensuite, et c'est le piège : dans une **condition `if:` de hook**, un `dir/**` mono-segment ne vise plus que `<cwd>/dir`. Pour retrouver un match à n'importe quelle profondeur, il faut écrire `**/dir/**`. Mais les règles de permission `deny` et `ask`, elles, **conservent** le match tous niveaux. Le même motif, écrit à deux endroits, ne couvre donc plus le même périmètre. Si vous avez des hooks qui filtrent par chemin, relisez-les : ils peuvent s'être silencieusement rétrécis à votre répertoire courant.

---

### `claude agents` demande votre confiance sur un dossier inconnu - v2.1.225

Les deux points d'entrée se comportent enfin pareil.

> *"Added a workspace trust prompt to `claude agents` for untrusted directories, matching the behavior of `claude`"*

**Ce que ça change concrètement** : lancer un agent en arrière-plan dans un répertoire jamais approuvé revenait à exécuter du code sans le garde-fou que `claude` impose déjà. C'est aligné. La première fois que vous lancez `claude agents` dans un nouveau dossier, attendez-vous à un prompt de confiance ; une fois approuvé, il ne revient plus pour ce dossier. Sur le fond, la direction est la bonne : un même geste risqué déclenche la même vérification, quel que soit le chemin emprunté pour y arriver.

---

## Améliorations notables

**Masquage de secrets renforcé dans le sandbox (v2.1.224, v2.1.221)** - de nouvelles options pour cacher des credentials aux commandes sandboxées : extraction ciblée par regex (`extract`, `onExtractNoMatch`), masquage conscient des JWT (`decode: "jwt"` + `maskClaims`), re-signature AWS SigV4 (`awsPairs`/`sigv4`), et un `mode: "mask"` qui laisse la commande lire une copie sentinelle pendant que le proxy substitue la vraie valeur en sortie. Si vous faites tourner des commandes qui ont besoin d'un token sans jamais devoir l'exposer en clair, c'est précisément l'outil.

**Une source de plugins `archive`, avec pinning SHA-256 (v2.1.224)** - on peut désormais installer un plugin depuis un zip servi en HTTPS, sans git ni npm, avec un pinning SHA-256 optionnel. Pratique en environnement fermé. Le mot important est *optionnel* : sans pinning, vous installez ce que le serveur vous donne le jour où vous l'appelez. Si vous utilisez cette source, épinglez le hash.

**Un plafond sur les recherches web et les sous-agents (v2.1.212)** - limite par session sur les appels WebSearch (200 par défaut) et sur les sous-agents lancés (200 par défaut, `/clear` remet le compteur à zéro), ajustable par variable d'environnement. L'objectif est de couper les boucles qui partent en vrille et brûlent du budget sans rien produire. À noter : le plafond de 200 sous-agents par session a été retiré en v2.1.224.

---

## Corrections importantes

**Plusieurs contournements du contrôle de permission fermés (v2.1.214, v2.1.216, v2.1.223)** - les commandes de plus de 10 000 caractères promptent désormais toujours au lieu de s'exécuter automatiquement ; les redirections de descripteurs de fichiers que l'analyseur lisait mal échouent maintenant côté sûr ; les subscripts zsh dans les `[[ ]]` ne sont plus traités comme du texte inerte ; et une commande maquillée par des tabulations ou des caractères Unicode invisibles ne peut plus cacher une partie d'elle-même au dialogue d'approbation.

> *"Fixed Bash permission checks misjudging very long commands — commands over 10,000 characters now always prompt instead of running automatically"*
>
> *"Fixed permission prompts so commands padded with tabs or invisible Unicode can no longer hide part of the command from the approval dialog"*

**Ce que ça change concrètement** : quatre contournements distincts du même garde-fou, fermés en seize versions. Ce n'est pas un accident de parcours : décider d'autoriser une commande en analysant du shell est un problème structurellement difficile. Si vous vous appuyez sur l'approbation Bash comme filet de sécurité, prenez ces correctifs - mais gardez en tête que c'est un filet, pas un mur.

---

## En bref (mineur / cosmétique)

- v2.1.211 : `--forward-subagent-text` (et `CLAUDE_CODE_FORWARD_SUBAGENT_TEXT`) inclut le texte et le raisonnement des sous-agents dans la sortie stream-json ; en v2.1.219 le forwarding s'étend aux sous-agents imbriqués.
- v2.1.211 : les règles « always allow » se sauvegardent à la racine du dépôt, pour persister d'un worktree à l'autre.
- v2.1.212 : `/fork` copie la conversation dans une session d'arrière-plan ; le paramètre `mode` du Task tool est déprécié - les sous-agents héritent du mode de permission du parent.
- v2.1.214 : nouveau timestamp `modified` dans le frontmatter des fichiers mémoire ; outil `EndConversation` ajouté ; nouveaux attributs OpenTelemetry (`message.uuid`, `client_request_id`, `tool_source`).
- v2.1.216, v2.1.222 : les écritures de workflow et de tâches planifiées ne suivent plus un symlink `.claude` ; l'isolation worktree s'applique aux edits et au Bash de toute session.
- v2.1.218 : `/code-review` tourne comme sous-agent d'arrière-plan et n'encombre plus la conversation ; Claude n'auto-lance plus `/verify`, `/code-review` ni `/deep-research`.
- v2.1.223 : `/review` devient un alias de `/code-review` ; `/teleport` fait son apparition.
- v2.1.224 : `claude self-hosted-runner` (plans Team et Enterprise) ; réglages `crossSessionInbound` et `dialogExpiry`.
- v2.1.220, v2.1.226 : « Bug fixes and reliability improvements ».

---

**Pour aller plus loin** - Cette semaine est dense en durcissements du contrôle de permission : quatre contournements fermés (v2.1.214, v2.1.216, v2.1.223) et une règle `Edit(src/**)` qui cesse d'auto-approuver tout l'arbre. La leçon durable n'est pas « il y avait des trous », c'est qu'un filtre d'approbation qui analyse du shell est fragile par nature - et qu'une allowlist étroite vaut mieux qu'une denylist. Le raisonnement complet, avec une checklist gratuite : notre guide [Harden Claude Code permissions](../resources/guides/harden-claude-code-permissions.md).

---

*Veille générée à partir du changelog officiel Anthropic.*
*Les interprétations et cas d'usage sont explicitement indiqués comme tels.*
