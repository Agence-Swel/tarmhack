---
title: "Claude Code — Ce qui change cette semaine (v2.1.227 → v2.1.233)"
date: 2026-08-17
period: "Semaine W3 · v2.1.227 → v2.1.233"
sources:
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
  - https://registry.npmjs.org/-/package/@anthropic-ai/claude-code/dist-tags
lang: fr
---

# Claude Code — Ce qui change cette semaine

> Période : semaine W3 (août 2026) · Versions v2.1.227 → v2.1.233 (6 versions publiées — la v2.1.230 n'existe pas au changelog)
> Sources : [CHANGELOG.md GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [code.claude.com](https://code.claude.com/docs/en/changelog)
> Généré le 2026-08-17

> **Statut de stabilité — cette semaine, tout est encore en préparation.** Les dist-tags npm donnent `stable 2.1.224` / `latest 2.1.233` : les six versions décrites ci-dessous sont toutes au-dessus du canal stable. Si vous suivez stable, vous n'avez encore rien de tout ça — c'est exactement le rôle du décalage entre les deux canaux. Le changement de valeur par défaut sur le fork de sous-agents, en particulier, gagne à mûrir une semaine de plus. Vérifiez `claude --version` et votre canal avant de déployer.

---

## Ce qui compte vraiment

### Vos sous-agents se forkent, et ils travaillent en arrière-plan — v2.1.232 (13 août 2026)

Deux bascules dans la même phrase, et elles changent le comportement par défaut.

> *"Subagent forking is now on by default: a `subagent_type: \"fork\"` subagent inherits the full conversation and prompt cache, and non-teammate agent spawns in interactive sessions now run in the background by default"*

**Ce que ça change concrètement** : un sous-agent forké hérite désormais de *toute* la conversation et du cache de prompt, au lieu de repartir d'une page blanche. C'est du concret sur la latence et sur la facture — le contexte déjà payé n'est pas repayé. La seconde bascule est plus discrète : quand vous lancez un agent depuis une session interactive, il part en arrière-plan sans que vous l'ayez demandé. Votre session principale ne vous attend plus. C'est confortable à une condition, savoir ce qui tourne : attendez-vous à voir plus de sessions dans vos listings sans avoir rien changé à vos habitudes. Prenez le réflexe `/tasks`.

---

### La confiance ne s'hérite plus d'un dossier parent — v2.1.232 (13 août 2026)

Approuver un dossier n'ouvre plus tout ce qu'il contient.

> *"Fixed nested git repositories inheriting trust from a parent directory; each repository now requires its own trust confirmation"*

**Ce que ça change concrètement** : si vous travaillez avec des dépôts imbriqués — monorepo à sous-modules, worktrees, dépendances vendorées — vous allez voir des demandes de confiance que vous ne voyiez pas. Un peu de friction contre une vraie garantie. Le point à surveiller n'est pas l'usage interactif, où vous répondez simplement à l'invite : c'est **l'automatisation**. Un pipeline ou un script qui lance Claude Code sans humain devant **n'a aucun moyen de répondre** à une demande de confirmation, et ce qu'il obtiendra ressemblera à une panne silencieuse. Testez vos automatisations avant que cette version atteigne le canal stable, en vérifiant sur quel dépôt exactement elles s'exécutent.

---

### Tapez `@` pour parler à une autre session — v2.1.232 (13 août 2026)

Vos sessions cessent d'être des silos.

> *"Type `@` in the prompt to mention another Claude session by name; Claude then uses `SendMessage` to reach that session directly"*
>
> *"Interactive sessions on one machine now keep unique names: starting or renaming a session to a name another live session already uses gives it a `name-word-word` variant and tells you"*

**Ce que ça change concrètement** : on passe de « j'ai plusieurs sessions ouvertes » à « mes sessions se parlent ». Vous pouvez faire préparer un contexte par l'une pendant qu'une autre écrit du code, et les faire converger sans passer par le presse-papier. L'unicité des noms n'est pas un détail cosmétique : sans elle, une messagerie par nom est ingérable — et le fait que les deux soient livrées ensemble indique une fonctionnalité pensée pour durer. Nommez vos sessions explicitement dès le départ (`revue`, `tests`, `doc`) plutôt que de subir les variantes automatiques, et réglez tout de suite dans `/config` la ligne « Messages from your other sessions » : elle accepte accepter, suspendre ou refuser. Si vous n'en voulez pas, refusez — ne subissez pas.

---

### Les outils de todo disparaissent sur les modèles récents — v2.1.233 (14 août 2026)

C'est un retrait, pas un bug.

> *"Todo/task-tracking tools (TaskCreate/Get/Update/List, TodoWrite) are no longer available on Opus 4.8, Sonnet 5, Fable 5, Mythos 5, and newer models; set `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` to bring them back"*

**Ce que ça change concrètement** : si vous avez des fichiers d'instructions, des compétences ou des agents qui disent explicitement au modèle de tenir une liste de tâches avec `TodoWrite`, ils réclament désormais un outil qui n'existe plus. Le symptôme sera discret — le modèle fait autre chose, personne ne remarque. Allez relire ces fichiers maintenant, pas quand ce sera le comportement par défaut partout. La variable `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` les rétablit si vous en dépendez vraiment, mais traitez-la comme un sursis : quand une plateforme retire un outil de son offre par défaut, l'échappatoire est rarement éternelle.

---

## Améliorations notables

**Neuf familles de jetons GitLab sont désormais masquées (v2.1.232)** — la rédaction de secrets couvre `glrt-`, `gloas-`, `glptt-`, `glagent-`, `glimt-`, `glsoat-`, `glcbt-`, `glft-`, `glffct-`, avec rédaction complète des jetons routables `glpat-`/`gldt-`. Le magasin de configuration du CLI `glab` reçoit les mêmes protections de sandbox et de chemin de credentials que `gh`. Si vous êtes sur GitLab, c'est la ligne la plus utile de la semaine.

**Les marketplaces de plugins parlent GitLab (v2.1.232, v2.1.233)** — les URLs `gitlab.com` nues, sous-groupes imbriqués compris, se clonent comme les URLs `github.com` ; le flag `--worktree` et la vue `claude agents` acceptent les URLs de merge request, affichées `!N` ; et l'astuce d'installation de l'app GitHub disparaît des dépôts dont l'origine est sur gitlab.com ou bitbucket.org. Trois lignes, une même direction : l'outil cesse d'être GitHub-centré.

**Les compétences synchronisées depuis claude.ai sont bridées (v2.1.228)** — elles ne peuvent plus masquer une commande locale ou un prompt MCP du même nom, leurs descriptions sont assainies et étiquetées, et leurs corps n'exécutent plus `!` ni n'expansent `@`.

> *"Hardened skills synced from claude.ai: they no longer shadow local commands or MCP prompts, their descriptions are sanitized and labeled, and on your machine their bodies don't run `!` commands or expand `@` files"*

**Deux variables d'environnement nouvelles (v2.1.233)** — `CLAUDE_CODE_TOOL_MEMORY_LIMIT` active sous Linux, en opt-in, un cgroup mémoire sur les commandes de l'outil Bash *« so a runaway build can't stall the session »*. `CLAUDE_CODE_WEBFETCH_CACHE_TTL_MS` règle le TTL du cache d'URL de WebFetch (défaut inchangé, 15 minutes).

---

## Corrections importantes

**Une classe de contournement de chemin Windows est fermée (v2.1.233)** — c'est la correction la plus sérieuse du lot.

> *"Fixed Windows paths spelled with the NT `\??\` device prefix bypassing UNC path validation, closing an NTLM credential-leak vector"*

**Ce que ça change concrètement** : un chemin orthographié avec le préfixe device NT passait à travers la validation UNC, ouvrant un vecteur de fuite d'identifiants NTLM. Si vous êtes sous Windows, cette version vaut le passage dès qu'elle atteint le canal stable.

**Deux contournements de permission fermés, PowerShell et Git Bash (v2.1.232)** — d'un côté des paramètres d'écriture de variables qui pouvaient écraser silencieusement `$PSDefaultParameterValues` et rediriger l'accès fichier des commandes suivantes ; de l'autre, des symlinks de style Cygwin que la validation de chemin voyait comme des fichiers ordinaires. Les écritures qui passent par eux exigent maintenant une approbation.

**Un aller-retour sur les permissions Bash sous Windows (v2.1.232 puis v2.1.233)** — la v2.1.232 avait généralisé le contrôle des redirections d'entrée (`< file`) et resserré le traitement des symlinks Cygwin. La v2.1.233 **révoque** ces deux changements — *« a narrower version will return in a later release »* — et corrige au passage une régression où le mode auto s'arrêtait pour approbation manuelle sur des commandes `cd <dir> && <command> > file` parfaitement ordinaires. Si vous aviez trouvé le mode auto insupportable en v2.1.232, c'était ça.

**Le nettoyage de session ne vide plus le dossier `memory/` d'un projet (v2.1.228)** — correction sèche, conséquence sérieuse : la mémoire d'un projet pouvait être supprimée par un cycle de ménage automatique.

**Trois crashs et trois irritants MCP (v2.1.229, v2.1.231, v2.1.233)** — côté crashs : un appel d'outil avec une valeur `glob`, `file_path` ou `command` non textuelle, un `RangeError` sur terminal très étroit (qui pouvait aussi tuer `claude --continue` au démarrage), et sous Windows un chemin étendu ou UNC référencé dans un message. Côté MCP : le timeout de connexion de 30 secondes s'écoulait intégralement quand un serveur répondait mal à la sonde de protocole, l'OAuth échouait sur les serveurs à autorisation stricte (`127.0.0.1` remplace `localhost`) et sur ceux à client pré-enregistré, et les connexions v2 ne rouvrent plus indéfiniment le flux `subscriptions/listen`.

---

## En bref (mineur / cosmétique)

- v2.1.233 : une ligne `[claude-code:unrecognized_model]` est écrite sur stderr en mode print quand une requête part pour un identifiant de modèle inconnu ; `modelOverrides` la fait taire.
- v2.1.233 : les hooks `Notification` ne se déclenchaient pas sur les invites de permission sous Claude Desktop et VS Code — corrigé. `claude plugin validate` inspecte désormais un répertoire `.claude/skills` nu et signale les `SKILL.md` au frontmatter illisible.
- v2.1.232 : `additionalMarketplaces` et `allowedMarketplaces` deviennent des alias acceptés de `extraKnownMarketplaces` et `strictKnownMarketplaces`. `sandbox.ripgrep` n'est plus honoré depuis les settings de projet — user, managed et `--settings` uniquement.
- v2.1.232 : le répertoire de socket de messagerie inter-sessions sur `/tmp` partagé refuse un symlink pré-planté ou le répertoire d'un autre utilisateur. Les sessions Cowork n'inlinent plus les imports `@` externes des fichiers de mémoire utilisateur.
- v2.1.229 : `/commit-push-pr` n'auto-approuve plus les commandes git/gh portant `--force`, `--amend`, `--no-verify`. Le sandbox refuse en fail-closed les orthographes ambiguës de domaines réseau, signalées par `/doctor`. Un marketplace de plugins peut venir d'une **commande locale**, re-résolue à chaque session (`mode: "link"` l'utilise sur place).
- v2.1.228 : l'outil Write peut écraser un fichier non lu dans la session sur les modèles récents, comme le fait déjà Edit ; les modèles plus anciens exigent toujours la lecture préalable.
- v2.1.227 à v2.1.232 : confort d'interface — menu des commandes slash plus lisible, groupes de sessions dans la barre latérale VS Code, panneau `/btw` redimensionnable, et Remote Control qui reconnecte pendant ~30 minutes après une coupure réseau.

---

**Pour aller plus loin** — La semaine est dense en frontières de confiance : la confiance cesse de s'hériter d'un dossier parent, deux contournements de permission sont fermés sous Windows, un troisième est révoqué le lendemain de sa sortie, et `/commit-push-pr` cesse d'auto-approuver `--force`. La leçon durable n'est pas « il y avait des trous » — c'est qu'un filtre d'approbation qui analyse du shell reste fragile par construction, et qu'une allowlist étroite vaut mieux qu'une denylist. Le raisonnement complet, avec une checklist gratuite : notre guide [Harden Claude Code permissions](../resources/guides/harden-claude-code-permissions.md).

---

*Veille générée à partir du changelog officiel Anthropic.*
*Les interprétations et cas d'usage sont explicitement indiqués comme tels.*
