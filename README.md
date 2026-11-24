# cybersec skills pour Claude Code

Un kit pour que Claude Code écrive, vérifie et mette à jour du code sans ouvrir de faille.

## Pourquoi ce kit existe

Un avis de sécurité publié sur une version de Next.js a bloqué le build d'une vraie application : le build refusait de se terminer, donc aucun déploiement ne pouvait partir. La version était visée par sept avis publiés, tous corrigés dans une version plus récente.

Mettre à jour vite, sans vérifier, peut casser l'app. Ne rien faire laisse une faille ouverte. Ce kit donne à Claude une méthode pour faire les deux dans le bon ordre, les outils de scan pour voir les problèmes avant qu'ils partent en production, et la vérification formelle (pstack) pour prouver qu'un parcours ne peut pas casser :

- **`app-audit-upgrade`** : auditer les dépendances et les avis de sécurité, puis mettre à jour dans un worktree Git séparé, avec sauvegarde, vérification des doublons, installation depuis le lockfile, typecheck, lint, tests, build, et un nouvel audit à la fin.
- **`cybersec-checklist`** : la procédure pour tout changement qui touche l'authentification, les paiements, le chiffrement, les entrées utilisateur, les secrets, les dépendances, les routes publiques ou les données personnelles. Avec une checklist à cocher avant chaque merge.

## Ce que le kit installe

| Quoi | Source | Rôle |
|---|---|---|
| `app-audit-upgrade`, `cybersec-checklist` | ce dépôt | nos skills, copiés dans `~/.claude/skills/` |
| plugin `security-guidance` | Anthropic | alertes de sécurité pendant que Claude modifie le code |
| plugin `cloudflare` | Cloudflare | Workers, WAF, Zero Trust, Turnstile |
| plugin `aikido` | Aikido | scan du code, des dépendances et des secrets |
| plugin `sonatype-guide` | Sonatype | risque d'un paquet avant de l'ajouter |
| plugin `ai-plugins` | Endor Labs | tri des vulnérabilités, mises à jour, réponse aux malwares |
| plugin `nightvision` | NightVision | découverte d'API et scans de sécurité de l'app |
| plugin `pstack` | [cursor/plugins](https://github.com/cursor/plugins/tree/main/pstack) (Lauren Tan) | vérification formelle TLA+, statecharts, vérification de l'app comme un utilisateur, TDD |

Les plugins sont installés depuis la marketplace officielle d'Anthropic et celle de Cursor pour pstack, à jour, jamais copiés ici. Certains demandent un compte chez leur éditeur (Cloudflare, Aikido, Sonatype, Endor Labs, NightVision) : Claude Code te demandera de te connecter la première fois que tu t'en sers.

## Installation

En une ligne :

```bash
curl -fsSL https://raw.githubusercontent.com/EagleHeaven/cybersec-skills/main/install.sh | bash
```

Ou en clonant le dépôt :

```bash
git clone https://github.com/EagleHeaven/cybersec-skills.git
cd cybersec-skills
./install.sh
```

Options :

```bash
./install.sh --list                 # tout ce que le kit installe
./install.sh --core                 # seulement nos skills, sans les plugins
./install.sh cybersec-checklist     # seulement les skills nommés de ce dépôt
./install.sh --help
```

Avec la ligne `curl`, les options passent après `bash -s --`, par exemple `... | bash -s -- --core`.

Nos skills vont dans `~/.claude/skills/`. Si une version existe déjà, elle est déplacée dans `~/.claude/skills-backups/`, rien n'est écrasé. Les plugins demandent la commande `claude`. Pour couper les couleurs : `NO_COLOR=1`.

Ensuite, ouvre une nouvelle session Claude Code.

Pour le design, voir le kit séparé : [design-skills](https://github.com/EagleHeaven/design-skills).

---

Publié par RedEagle01.
