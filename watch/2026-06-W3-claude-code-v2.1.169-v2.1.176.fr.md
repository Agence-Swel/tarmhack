---
title: "Claude Code - Ce qui change cette semaine (v2.1.169 → v2.1.176)"
date: 2026-06-15
period: "Semaine W3 · v2.1.169 → v2.1.176"
sources:
  - https://code.claude.com/docs/en/changelog
  - https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
lang: fr
---

# Claude Code - Ce qui change cette semaine

> Période : semaine W3 (3ᵉ semaine de juin) · 8 au 12 juin 2026 · Versions v2.1.169 → v2.1.176 (8 releases)
> Sources : [CHANGELOG.md GitHub](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md) · [code.claude.com](https://code.claude.com/docs/en/changelog)
> Généré le 2026-06-15

---

## Ce qui compte vraiment

### Un nouveau modèle, Claude Fable 5 - v2.1.170

Claude Code annonce l'arrivée d'un nouveau modèle, Claude Fable 5, accessible à partir de la v2.1.170. Le changelog ne tourne pas autour du pot.

> *"Introducing Claude Fable 5: a Mythos-class model […]. Fable's capabilities exceed those of any model we've ever made generally available."*

**Ce que ça change concrètement** : pour y accéder, mettez à jour vers la 2.1.170 ou plus récent. Détail utile - Fable 5 embarque 1M de contexte par défaut, et le CLI sait désormais le gérer proprement : le suffixe `[1m]` dans le nom du modèle est retiré automatiquement (v2.1.173). Rien à bricoler côté nom de modèle.

---

### Le verrouillage des modèles autorisés devient contraignant - v2.1.175

Jusqu'ici, le réglage `availableModels` listait les modèles permis. Le nouveau réglage managé `enforceAvailableModels` le rend contraignant.

> *"the `availableModels` allowlist also constrains the Default model […], and user or project settings can no longer widen a managed `availableModels` list"*

**Ce que ça change concrètement** : si vous administrez Claude Code pour une équipe, c'est le levier qui garantit qu'un modèle non approuvé ne tournera nulle part - la restriction s'applique partout, y compris aux overrides de modèle des sous-agents, au picker de dispatch d'agents et au modèle de l'advisor (v2.1.172). Si vous êtes développeur dans une équipe encadrée, attendez-vous à voir votre liste de modèles verrouillée par le haut.

---

### Les sous-agents peuvent maintenant s'imbriquer - v2.1.172

Un sous-agent peut désormais lancer ses propres sous-agents, sur cinq niveaux de profondeur.

> *"Sub-agents can now spawn their own sub-agents (up to 5 levels deep)"*

**Ce que ça change concrètement** : c'est puissant pour décomposer une tâche complexe. Le revers : suivre ce qui tourne réellement devient plus dur - la même version corrige d'ailleurs un sous-agent imbriqué resté bloqué en « actif » après l'arrêt de son enfant. Notre conseil : utilisez la cascade, mais gardez un œil sur la profondeur.

---

### Un mode sûr pour déboguer - v2.1.169

Le nouveau drapeau `--safe-mode` (et la variable `CLAUDE_CODE_SAFE_MODE`) démarre Claude Code avec toutes les personnalisations désactivées.

> *"with all customizations (CLAUDE.md, plugins, skills, hooks, MCP servers) disabled"*

**Ce que ça change concrètement** : quand quelque chose se comporte mal et que vous ne savez pas si c'est un hook, un plugin ou un serveur MCP, c'est le réflexe à avoir - lancez en mode sûr et isolez le problème en une commande, au lieu de désactiver vos extensions une par une.

---

## Améliorations notables

**`/cd` sans casser le cache (v2.1.169)** - la commande `/cd` déplace une session vers un nouveau répertoire de travail sans casser le cache de prompt. Pratique pour changer de dossier sans repartir de zéro ni perdre le bénéfice du cache.

**Supervision des sessions plus complète (v2.1.169)** - `claude agents --json` gagne un drapeau `--all` pour inclure les sessions terminées, plus des champs `id` et `state` ; il n'oublie plus les sessions bloquées ou tout juste lancées. Si vous scriptez la supervision de vos sessions en arrière-plan, vous avez enfin une vue complète.

**Région Bedrock lue depuis `~/.aws` (v2.1.172)** - côté Amazon Bedrock, le CLI lit maintenant la région depuis vos fichiers `~/.aws` quand `AWS_REGION` n'est pas défini, comme le fait le SDK AWS, et `/status` indique d'où vient la région. Moins de configuration redondante.

**Mémoire d'équipe retrouvée en session distante (v2.1.172)** - le rappel mémoire retrouve désormais les « team memory stores » montés via `CLAUDE_MEMORY_STORES`, y compris en session distante.

---

## Corrections importantes

**Repli propre sans Opus 4.8 (v2.1.176)** - sur les organisations sans Opus 4.8, le classificateur de mode auto retombe proprement sur le meilleur modèle Opus disponible au lieu d'échouer. Corrigé.

**Bannière de crédits Fable 5 (v2.1.174)** - la bannière « Fable 5 consomme des crédits » ne s'affiche plus à tort pour les comptes entreprise en facturation à l'usage. Corrigé.

**Règles de permission à jokers (v2.1.172)** - `WebFetch(domain:*.example.com)` matche bien les sous-domaines, et `Read(secrets-*/config.json)` (joker au milieu du motif) n'est plus rejeté au démarrage. Si vous aviez des règles silencieusement ignorées, vérifiez-les. Corrigé.

**Transcripts retrouvés (v2.1.170)** - les sessions lancées depuis le terminal intégré de VS Code (ou tout shell héritant des variables d'environnement de Claude Code) sauvegardent à nouveau leur transcript et réapparaissent dans `--resume`. Corrigé.

**Réactivité en longue conversation (v2.1.172)** - suppression de normalisations de messages redondantes en longues conversations et moins de re-renders quand des sous-agents tournent en parallèle. L'app répond mieux quand l'historique s'allonge.

---

## En bref (mineur / cosmétique)

- v2.1.174 : `wheelScrollAccelerationEnabled` désactive l'accélération de la molette en plein écran.
- v2.1.176 : `footerLinksRegexes` - badges de liens dans le pied de page, déclenchés par expression régulière.
- v2.1.169 : `disableBundledSkills` (et `CLAUDE_CODE_DISABLE_BUNDLED_SKILLS`) masque les skills, workflows et slash commands intégrés.
- v2.1.176 : les titres de session sont générés dans la langue de votre conversation ; épinglez une langue via le réglage `language`.
- v2.1.174 : le picker `/model` affiche plus clairement la famille vers laquelle « Default » résout (Opus, Sonnet selon le plan).
- v2.1.172 : chargement des outils Claude in Chrome en un seul appel groupé au lieu d'un par outil ; métrique OTEL `claude_code.lines_of_code.count` enrichie d'un attribut `model`.
- v2.1.172 / v2.1.175 : correctifs Bedrock GovCloud (`us-gov-*`) et de caching des credentials.
- v2.1.169 / v2.1.172 / v2.1.174 / v2.1.176 : nombreux correctifs Remote Control et sessions en arrière-plan (respawn, reconnexion, daemon Windows).

---

**Pour aller plus loin** - Cette semaine ajoute un levier de gouvernance managée : `enforceAvailableModels` (v2.1.175) verrouille la liste des modèles autorisés pour toute une équipe, même via un sous-agent. Comment en faire une posture durable de moindre privilège (deny-by-default, écritures sensibles, fenêtre de versions, allowlist de modèles) : notre guide [Harden your own Claude Code permissions: go deny-by-default](../resources/guides/harden-claude-code-permissions.md).

---

*Veille générée à partir du changelog officiel Anthropic.*
*Les interprétations et cas d'usage sont explicitement indiqués comme tels.*
