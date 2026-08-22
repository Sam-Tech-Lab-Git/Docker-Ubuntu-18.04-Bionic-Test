# Ubuntu 18.04 LTS (Bionic) — base image

Minimal, hardened, multi-architecture **Ubuntu 18.04 LTS (Bionic)** base image, built `FROM
scratch` from the official Ubuntu OCI rootfs. A complete Ubuntu userland, a non-root user, a
hardened baseline — then it gets out of your way.

**Full documentation, in English and French:**
<https://github.com/Sam-Tech-Lab-Git/Docker-Ubuntu-18.04-Bionic-Test>

*Version française plus bas.*

---

## ⚠️ Read this first — Ubuntu 18.04 is out of standard support

Ubuntu 18.04 LTS left standard support on **31 May 2023**. Its public archive no longer receives
new security updates: those are published through Ubuntu Pro (ESM), which this image does not
subscribe to.

- Monthly rebuilds refresh the image against what Ubuntu still serves for Bionic. They do **not**
  bring in fixes that exist only behind ESM, so a CVE fixed for 20.04 or 22.04 may stay open here
  indefinitely.
- The published vulnerability scans deliberately include findings with **no fix available** — on a
  frozen archive that is most of them. The reports are long on purpose.

Use this image where an 18.04 userland is a hard requirement: legacy binaries, an old toolchain,
reproducing a historical environment. **For anything new, start from a supported Ubuntu LTS.**

---

## Tags

| Tag | Contents |
|---|---|
| `latest` | Tracks the monthly rebuild — amd64 + arm64 |
| `YYYY.MM` (e.g. `2026.08`) | The build from that month — amd64 + arm64 |

Tags point at a multi-architecture manifest; Docker selects the right image for the host platform.

**Neither tag is immutable.** `YYYY.MM` names the month, not one specific build: any build during
that month republishes it. For a genuinely fixed image, pin by digest:

```bash
docker pull samtechlab/ubuntu-18.04-bionic-test@sha256:<digest>
```

Also published on GHCR as `ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test`.

---

## Quick start

```bash
# Shell as the unprivileged appuser
docker run -it --rm samtechlab/ubuntu-18.04-bionic-test:latest

# Check who you are
docker run --rm samtechlab/ubuntu-18.04-bionic-test:latest id
```

Build on top of it. The image ends with `USER appuser`, so switch to root to install, then switch
back:

```dockerfile
FROM samtechlab/ubuntu-18.04-bionic-test:latest

USER root
RUN apt-get update && \
    apt-get install -y --no-install-recommends your-package && \
    rm -rf /var/lib/apt/lists/*
USER appuser
```

---

## No init system — your CMD is PID 1

This image ships no init system: your `CMD` runs directly as PID 1. That is fine for a single
well-behaved process, but PID 1 has special duties on Linux — it must reap orphaned children and
handle signals itself. A process that does neither leaves zombies behind, or ignores `SIGTERM` so
`docker stop` waits out its timeout and kills it.

If your process is not designed for that role, let Docker supply a minimal init:

```bash
docker run --init samtechlab/ubuntu-18.04-bionic-test:latest your-command
```

```yaml
services:
  app:
    image: samtechlab/ubuntu-18.04-bionic-test:latest
    init: true
```

To supervise **several** processes in one container, a plain `CMD` is not enough: use a real init
system, or split them into separate containers. A variant of this image with
[s6-overlay](https://github.com/Sam-Tech-Lab-Git/Docker-Ubuntu-18.04-Bionic-S6-Overlay-Test)
covers that case.

---

## Key features

- Built `FROM scratch` from the official Ubuntu OCI rootfs — no third-party base layer
- Published as a single multi-arch manifest (`amd64`, `arm64`)
- **Runs as a non-root user (`appuser`) end to end** — there is no privileged process, PID 1 included
- **Hardening** — `root` locked, SUID/SGID stripped, world-writable bits removed, `umask 027`
- **Service managers neutralised** (`policy-rc.d`, `initctl`) so packages do not try to start daemons
- **Supply-chain integrity** — Alpine builder pinned by digest, CI actions pinned by commit SHA,
  SBOM and SLSA provenance attached to every image
- APT & dpkg optimisation — no recommended/suggested packages, no translations, clean cache
- Locale and timezone configured (`en_US.UTF-8`, `UTC`)
- Continuously verified — hadolint, 9 container integration tests on both architectures, weekly
  Trivy scans

---

## Environment variables

| Variable | Default | Description |
|---|---|---|
| `HOME` | `/config` | Home directory of `appuser` |
| `TZ` | `UTC` | Timezone |
| `LANG` | `en_US.UTF-8` | Locale (also `LANGUAGE`, `LC_ALL`) |
| `TERM` | `xterm` | Terminal type |
| `DEBIAN_FRONTEND` | `noninteractive` | Suppresses interactive APT prompts |
| `PATH` | `/usr/local/sbin:/usr/local/bin:…` | Standard system path |

**`PUID` and `PGID` are build-time arguments, not runtime settings.** `appuser` is created at
`1000:1000` when the image is built, and nothing applies them at container start — this image has
no init system. Passing `-e PUID=1001` to `docker run` has **no effect**. To use different IDs,
rebuild with `--build-arg PUID=1001 --build-arg PGID=1001`, or align permissions host-side.

---

## Filesystem and defaults

| Path | Purpose |
|---|---|
| `/config` | Home of `appuser`, mode `750`, owned by `appuser` — mount your persistent data here |

| Setting | Value |
|---|---|
| User | `appuser` (UID `1000`, GID `1000`) |
| Command | `CMD ["/bin/bash"]` |
| Entrypoint | none |

---

## Security model

The container has **no privileged process**: the image ends with `USER appuser`, so everything —
including PID 1 — runs unprivileged.

| Control | Implementation |
|---|---|
| Container user | `appuser` (UID `1000`), set with `USER` — no root process |
| `root` account | Password locked, `/root` mode `700` |
| Login shell for `appuser` | `/usr/sbin/nologin` |
| SUID/SGID binaries | Stripped image-wide at build time |
| World-writable files | Write bit removed image-wide at build time |
| Default umask | `027` |
| `/config` | Mode `750`, owned by `appuser` |
| Service managers | `policy-rc.d` and `initctl` neutralised |
| Supply chain | Alpine builder pinned by digest; CI actions pinned by commit SHA; SBOM + provenance published |

Recommended runtime hardening:

```yaml
services:
  app:
    image: samtechlab/ubuntu-18.04-bionic-test:latest
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    read_only: true
    tmpfs:
      - /tmp
```

Vulnerability reporting:
[SECURITY.md](https://github.com/Sam-Tech-Lab-Git/Docker-Ubuntu-18.04-Bionic-Test/blob/main/SECURITY.md)

---

## Troubleshooting

**`E: Could not open lock file /var/lib/apt/lists/lock (13: Permission denied)`**
The image runs as `appuser`. Switch to `USER root` in your Dockerfile to install packages, then
back to `USER appuser`.

**`bind() to 0.0.0.0:80 failed (13: Permission denied)`**
Unprivileged processes cannot bind ports below 1024. Use a port ≥ 1024 inside the container and
remap it on the host (`-p 80:8080`).

**`docker stop` takes ~10 seconds**
Your process is not handling `SIGTERM`, or is not reaping children as PID 1. Run it with
`--init`, or handle signals in the process itself.

**Files created in a mounted volume have the wrong owner**
`appuser` is fixed at `1000:1000`. Either `chown` the host directory to `1000:1000`, or rebuild
with `--build-arg PUID=… --build-arg PGID=…`.

**Zombie processes accumulate**
Your PID 1 is not reaping orphans. Use `--init`.

More entries in the
[full documentation](https://github.com/Sam-Tech-Lab-Git/Docker-Ubuntu-18.04-Bionic-Test#troubleshooting).

---

## License

MIT — see
[LICENSE](https://github.com/Sam-Tech-Lab-Git/Docker-Ubuntu-18.04-Bionic-Test/blob/main/LICENSE).
Copyright (c) 2026 Sam Tech Lab.

---
---

# Ubuntu 18.04 LTS (Bionic) — image de base

Image de base **Ubuntu 18.04 LTS (Bionic)** minimale, durcie et multi-architecture, construite
`FROM scratch` à partir du rootfs OCI officiel d'Ubuntu. Un userland Ubuntu complet, un
utilisateur non-root, un socle durci — puis elle vous laisse travailler.

**Documentation complète, en anglais et en français :**
<https://github.com/Sam-Tech-Lab-Git/Docker-Ubuntu-18.04-Bionic-Test>

---

## ⚠️ À lire d'abord — Ubuntu 18.04 est sorti du support standard

Ubuntu 18.04 LTS est sorti du support standard le **31 mai 2023**. Son archive publique ne reçoit
plus de nouvelles mises à jour de sécurité : celles-ci passent par Ubuntu Pro (ESM), auquel cette
image n'est pas abonnée.

- Les reconstructions mensuelles rafraîchissent l'image à partir de ce qu'Ubuntu sert encore pour
  Bionic. Elles n'apportent **pas** les correctifs qui n'existent que derrière l'ESM : une CVE
  corrigée pour 20.04 ou 22.04 peut rester ouverte ici indéfiniment.
- Les analyses de vulnérabilités publiées incluent volontairement les vulnérabilités **sans
  correctif disponible** — sur une archive figée, c'est la majorité d'entre elles.

Utilisez cette image là où un userland 18.04 est une contrainte dure : binaires hérités, ancienne
chaîne de compilation, reproduction d'un environnement historique. **Pour tout nouveau projet,
partez d'une LTS Ubuntu encore supportée.**

---

## Tags

| Tag | Contenu |
|---|---|
| `latest` | Suit la reconstruction mensuelle — amd64 + arm64 |
| `YYYY.MM` (par ex. `2026.08`) | Le build de ce mois-là — amd64 + arm64 |

Les tags pointent vers un manifeste multi-architecture : Docker sélectionne l'image correspondant
à la plateforme hôte.

**Aucun de ces tags n'est immuable.** `YYYY.MM` désigne le mois, pas un build en particulier :
tout build de ce mois-là le republie. Pour une image réellement figée, épinglez par digest :

```bash
docker pull samtechlab/ubuntu-18.04-bionic-test@sha256:<digest>
```

Également publiée sur GHCR : `ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test`.

---

## Démarrage rapide

```bash
# Shell en tant qu'appuser, non privilégié
docker run -it --rm samtechlab/ubuntu-18.04-bionic-test:latest

# Vérifier l'utilisateur effectif
docker run --rm samtechlab/ubuntu-18.04-bionic-test:latest id
```

Construire par-dessus. L'image se termine par `USER appuser` : repassez en root pour installer,
puis redescendez :

```dockerfile
FROM samtechlab/ubuntu-18.04-bionic-test:latest

USER root
RUN apt-get update && \
    apt-get install -y --no-install-recommends votre-paquet && \
    rm -rf /var/lib/apt/lists/*
USER appuser
```

---

## Pas de système d'init — votre CMD est PID 1

Cette image ne fournit aucun système d'init : votre `CMD` s'exécute directement en tant que
PID 1. C'est adapté à un processus unique et bien élevé, mais PID 1 a des devoirs particuliers
sous Linux — il doit récupérer les processus orphelins et gérer lui-même les signaux. Un processus
qui ne fait ni l'un ni l'autre laisse des zombies, ou ignore `SIGTERM` et oblige `docker stop` à
attendre son délai avant de le tuer.

Si votre processus n'est pas conçu pour ce rôle, laissez Docker fournir un init minimal :

```bash
docker run --init samtechlab/ubuntu-18.04-bionic-test:latest votre-commande
```

```yaml
services:
  app:
    image: samtechlab/ubuntu-18.04-bionic-test:latest
    init: true
```

Pour superviser **plusieurs** processus dans un même conteneur, un simple `CMD` ne suffit pas :
utilisez un vrai système d'init, ou séparez-les en conteneurs distincts. Une variante de cette
image avec
[s6-overlay](https://github.com/Sam-Tech-Lab-Git/Docker-Ubuntu-18.04-Bionic-S6-Overlay-Test)
couvre ce cas.

---

## Points forts

- Construite `FROM scratch` depuis le rootfs OCI officiel Ubuntu — aucune couche de base tierce
- Publiée comme un manifeste multi-architecture unique (`amd64`, `arm64`)
- **Tourne en utilisateur non-root (`appuser`) de bout en bout** — aucun processus privilégié,
  PID 1 compris
- **Durcissement** — `root` verrouillé, bits SUID/SGID supprimés, bits world-writable retirés,
  `umask 027`
- **Gestionnaires de services neutralisés** (`policy-rc.d`, `initctl`) : les paquets n'essaient
  pas de démarrer de daemons
- **Intégrité de la chaîne d'approvisionnement** — builder Alpine figé par digest, actions CI
  figées par SHA de commit, SBOM et provenance SLSA joints à chaque image
- Optimisation APT & dpkg — aucun paquet recommandé/suggéré, aucune traduction, cache propre
- Locale et fuseau horaire configurés (`en_US.UTF-8`, `UTC`)
- Vérifiée en continu — hadolint, 9 tests d'intégration sur les deux architectures, scans Trivy
  hebdomadaires

---

## Variables d'environnement

| Variable | Défaut | Description |
|---|---|---|
| `HOME` | `/config` | Répertoire personnel de `appuser` |
| `TZ` | `UTC` | Fuseau horaire |
| `LANG` | `en_US.UTF-8` | Locale (également `LANGUAGE`, `LC_ALL`) |
| `TERM` | `xterm` | Type de terminal |
| `DEBIAN_FRONTEND` | `noninteractive` | Supprime les invites APT interactives |
| `PATH` | `/usr/local/sbin:/usr/local/bin:…` | Chemin système standard |

**`PUID` et `PGID` sont des arguments de build, pas des réglages d'exécution.** `appuser` est créé
en `1000:1000` à la construction de l'image, et rien ne les applique au démarrage du conteneur —
cette image n'a pas de système d'init. Passer `-e PUID=1001` à `docker run` n'a **aucun effet**.
Pour d'autres identifiants, reconstruisez avec `--build-arg PUID=1001 --build-arg PGID=1001`, ou
alignez les permissions côté hôte.

---

## Arborescence et valeurs par défaut

| Chemin | Rôle |
|---|---|
| `/config` | Home de `appuser`, mode `750`, lui appartenant — montez vos données persistantes ici |

| Réglage | Valeur |
|---|---|
| Utilisateur | `appuser` (UID `1000`, GID `1000`) |
| Commande | `CMD ["/bin/bash"]` |
| Entrypoint | aucun |

---

## Modèle de sécurité

Le conteneur n'a **aucun processus privilégié** : l'image se termine par `USER appuser`, donc tout
— y compris PID 1 — s'exécute sans privilèges.

| Contrôle | Mise en œuvre |
|---|---|
| Utilisateur du conteneur | `appuser` (UID `1000`), défini par `USER` — aucun processus root |
| Compte `root` | Mot de passe verrouillé, `/root` en mode `700` |
| Shell de connexion d'`appuser` | `/usr/sbin/nologin` |
| Binaires SUID/SGID | Supprimés sur toute l'image au build |
| Fichiers world-writable | Bit d'écriture retiré sur toute l'image au build |
| Umask par défaut | `027` |
| `/config` | Mode `750`, appartenant à `appuser` |
| Gestionnaires de services | `policy-rc.d` et `initctl` neutralisés |
| Chaîne d'approvisionnement | Builder Alpine figé par digest ; actions CI figées par SHA ; SBOM et provenance publiés |

Durcissement recommandé à l'exécution :

```yaml
services:
  app:
    image: samtechlab/ubuntu-18.04-bionic-test:latest
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    read_only: true
    tmpfs:
      - /tmp
```

Signalement de vulnérabilité :
[SECURITY.md](https://github.com/Sam-Tech-Lab-Git/Docker-Ubuntu-18.04-Bionic-Test/blob/main/SECURITY.md)

---

## Dépannage

**`E: Could not open lock file /var/lib/apt/lists/lock (13: Permission denied)`**
L'image tourne en `appuser`. Passez en `USER root` dans votre Dockerfile pour installer des
paquets, puis revenez à `USER appuser`.

**`bind() to 0.0.0.0:80 failed (13: Permission denied)`**
Un processus non privilégié ne peut pas écouter sous le port 1024. Utilisez un port ≥ 1024 dans
le conteneur et remappez-le côté hôte (`-p 80:8080`).

**`docker stop` traîne une dizaine de secondes**
Votre processus ne gère pas `SIGTERM`, ou ne récupère pas ses enfants en tant que PID 1. Lancez-le
avec `--init`, ou gérez les signaux dans le processus lui-même.

**Les fichiers créés dans un volume monté ont le mauvais propriétaire**
`appuser` est figé à `1000:1000`. Soit vous faites un `chown` du répertoire hôte vers `1000:1000`,
soit vous reconstruisez avec `--build-arg PUID=… --build-arg PGID=…`.

**Des processus zombies s'accumulent**
Votre PID 1 ne récupère pas les orphelins. Utilisez `--init`.

D'autres entrées dans la
[documentation complète](https://github.com/Sam-Tech-Lab-Git/Docker-Ubuntu-18.04-Bionic-Test#dépannage).

---

## Licence

MIT — voir
[LICENSE](https://github.com/Sam-Tech-Lab-Git/Docker-Ubuntu-18.04-Bionic-Test/blob/main/LICENSE).
Copyright (c) 2026 Sam Tech Lab.
