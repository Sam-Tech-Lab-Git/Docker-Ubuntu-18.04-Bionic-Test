# Docker Ubuntu 18.04 LTS (Bionic) - Sam Tech Lab

<table align="center">
  <tr>
    <td align="center" width="50%">
      <a href="https://github.com/Sam-Tech-Lab-Git" target="_blank">
        <img src="https://raw.githubusercontent.com/Sam-Dz-Devops/Images/main/Sam-Tech-Site-Web.png"
             alt="Sam Tech Lab Logo" width="300"/>
      </a>
    </td>
    <td align="center" width="50%">
      <a href="https://hub.docker.com/r/samtechlab/ubuntu-18.04-bionic-test" target="_blank">
        <img src="https://upload.wikimedia.org/wikipedia/commons/7/76/Ubuntu-logo-2022.svg?sanitize=true"
             alt="Ubuntu Logo" width="180"/>
      </a>
    </td>
  </tr>
</table>

<p align="center">
  <a href="https://hub.docker.com/r/samtechlab/ubuntu-18.04-bionic-test" target="_blank">
    <img src="https://img.shields.io/docker/pulls/samtechlab/ubuntu-18.04-bionic-test.svg?style=for-the-badge&logo=docker&logoColor=white" alt="Docker Pulls"/>
  </a>
  <a href="https://hub.docker.com/r/samtechlab/ubuntu-18.04-bionic-test" target="_blank">
    <img src="https://img.shields.io/docker/stars/samtechlab/ubuntu-18.04-bionic-test.svg?style=for-the-badge&logo=docker&logoColor=white" alt="Docker Stars"/>
  </a>
  <a href="https://github.com/Sam-Tech-Lab-Git" target="_blank">
    <img src="https://img.shields.io/static/v1?label=SamTechLab&message=GitHub&color=94398d&labelColor=555555&style=for-the-badge&logo=github&logoColor=white" alt="GitHub"/>
  </a>
  <a href="https://github.com/Sam-Tech-Lab-Git/Docker-Ubuntu-18.04-Bionic-Test/blob/main/LICENSE" target="_blank">
    <img src="https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge" alt="License: MIT"/>
  </a>
  <a href="https://github.com/Sam-Tech-Lab-Git/Docker-Ubuntu-18.04-Bionic-Test/actions/workflows/build-multi-arch.yml" target="_blank">
      <img src="https://github.com/Sam-Tech-Lab-Git/Docker-Ubuntu-18.04-Bionic-Test/actions/workflows/build-multi-arch.yml/badge.svg" alt="Build multi-arch"/>
  </a>
  <a href="https://github.com/Sam-Tech-Lab-Git/Docker-Ubuntu-18.04-Bionic-Test/actions/workflows/vuln-scan.yml" target="_blank">
      <img src="https://github.com/Sam-Tech-Lab-Git/Docker-Ubuntu-18.04-Bionic-Test/actions/workflows/vuln-scan.yml/badge.svg" alt="Vulnerability Scan"/>
  </a>
</p>

<p align="center">
  <b>English</b> · <a href="#documentation-française">Version française ↓</a>
</p>

---

## Quickstart

```bash
# Run a shell (as the unprivileged appuser)
docker run -it --rm ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest

# Or from Docker Hub
docker run -it --rm samtechlab/ubuntu-18.04-bionic-test:latest
```

Build on top of it:

```dockerfile
FROM ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest

# The image runs as appuser; switch to root to install, then switch back.
USER root
RUN apt-get update && \
    apt-get install -y --no-install-recommends your-package && \
    rm -rf /var/lib/apt/lists/*
USER appuser
```

**Contents:** [Overview](#overview) · [Image reference](#image-reference) ·
[Getting started](#getting-started) · [Complete example NGINX](#complete-example-nginx) ·
[Security model](#security-model) · [Troubleshooting](#troubleshooting) ·
[Maintenance](#maintenance)

---

## Overview

A **minimal, hardened, multi-architecture Ubuntu 18.04 LTS (Bionic) base image**, built
`FROM scratch` from the **official Ubuntu OCI rootfs** — for authenticity, small size and
reproducibility.

It is designed as a **foundation for your own images**: a complete Ubuntu system (without a kernel — containers
share the host's), a non-root user,
and a hardened system baseline — then it gets out of your way.

> **Supported architectures:** `linux/amd64`, `linux/arm64`
> **Automatic monthly rebuilds** pick up whatever Ubuntu still publishes for Bionic — read
> [Base image support status](#base-image-support-status) before deploying this in production.

### Key features

- ✅ **Built `FROM scratch`** from the official Ubuntu OCI rootfs — no third-party base layer
- ✅ **Multi-arch** published as a single manifest (`amd64`, `arm64`)
- ✅ **Runs as a non-root user** (`appuser`) end to end — there is no privileged process
- ✅ **System hardening** — `root` account locked, SUID/SGID bits stripped, world-writable bits
  removed, `umask 027`
- ✅ **Supply-chain integrity** — Alpine builder pinned by digest, CI actions pinned by commit SHA
- ✅ **APT & dpkg optimisation** — no recommended/suggested packages, no translations, clean cache
- ✅ **Service managers neutralised** (`systemd`, `upstart`) so packages do not try to start daemons
- ✅ **Locale and timezone configured** (`en_US.UTF-8`, `UTC`)
- ✅ **SBOM and SLSA provenance** attached to every published image
- ✅ **Continuously verified** — hadolint, 9 container integration tests run on **both
  architectures**, weekly Trivy scans

---

## Image reference

### Registries and tags

| Registry | Image | Architectures |
|---|---|---|
| Docker Hub | `samtechlab/ubuntu-18.04-bionic-test:latest` | amd64 + arm64 |
| Docker Hub | `samtechlab/ubuntu-18.04-bionic-test:YYYY.MM` | amd64 + arm64 |
| GHCR | `ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest` | amd64 + arm64 |
| GHCR | `ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:YYYY.MM` | amd64 + arm64 |

Tags point at a multi-architecture manifest — Docker automatically selects the right image for
the host platform. `latest` tracks the monthly rebuild.

**Neither tag is immutable.** A `YYYY.MM` tag names the month a build ran, not one specific build:
any build during that month republishes it — the monthly schedule, a manual dispatch, or a push to
`main` that touches the Dockerfile. On 2 August 2026, six builds all wrote `2026.08`.

For a genuinely fixed image, **pin by digest** — see
[Verifying what you are running](#verifying-what-you-are-running).

### Included packages

| Category | Packages |
|---|---|
| Shell & base | `bash`, `cron`, `curl`, `gnupg`, `jq`, `netcat-openbsd`, `tzdata` |
| System support | `apt-utils`, `ca-certificates`, `locales` |

### Environment variables

| Variable | Default | Description |
|---|---|---|
| `HOME` | `/config` | Home directory of `appuser` |
| `TZ` | `UTC` | Timezone |
| `LANG` | `en_US.UTF-8` | Locale (also `LANGUAGE`, `LC_ALL`) |
| `TERM` | `xterm` | Terminal type |
| `DEBIAN_FRONTEND` | `noninteractive` | Suppresses interactive APT prompts |
| `PATH` | `/usr/local/sbin:/usr/local/bin:…` | Standard system path |

> **`PUID` and `PGID` are build arguments, not runtime settings.** They are baked in when
> `appuser` is created (`1000:1000`), and nothing applies them at container start — this image has
> no init system. Passing `-e PUID=1001` to `docker run` has **no effect**. To use different IDs:
>
> ```bash
> docker buildx build -f Dockerfile-multi-arch \
>   --build-arg PUID=1001 --build-arg PGID=1001 -t my-bionic .
> ```
>
> …or align permissions on the host side instead (see [Troubleshooting](#troubleshooting)).

### Filesystem layout

| Path | Purpose |
|---|---|
| `/config` | Home of `appuser`, mode `750`, owned by `appuser` — mount your persistent data here |

### Default behaviour

| Setting | Value |
|---|---|
| User | `appuser` (UID `1000`, GID `1000`) |
| Shell | `SHELL ["/bin/bash", "-c"]` |
| Command | `CMD ["/bin/bash"]` |
| Entrypoint | none |

---

## Getting started

### Run a container

```bash
docker run -it --rm ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest
```

You get a `bash` shell as `appuser`. There is no root process in the container.

### Build your own image

The image ends with `USER appuser`, so **any derived image must switch back to `root` to install
packages**, then drop privileges again:

```dockerfile
FROM ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest

USER root
RUN apt-get update && \
    apt-get install -y --no-install-recommends nginx && \
    rm -rf /var/lib/apt/lists/*
USER appuser

CMD ["nginx", "-g", "daemon off;"]
```

> **Do not install packages at container start** (e.g. an `apt-get install` in `command:`).
> The container runs unprivileged, so APT fails with `Permission denied` on
> `/var/lib/apt/lists`. Install at build time, as above.

### Running processes and signals

The image has no init system: your `CMD` runs directly as PID 1. That is fine for a single
well-behaved process, but PID 1 has special duties in Linux — it must reap orphaned child
processes and handle signals itself. A process that does neither leaves zombies behind, or ignores
`SIGTERM` so `docker stop` has to wait for its timeout and kill it.

If your process is not designed for that role, let Docker supply a minimal init:

```bash
docker run --init ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest your-command
```

```yaml
services:
  app:
    image: ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest
    init: true
```

If you need to supervise **several** processes in one container, a plain `CMD` is not enough —
use a real init system, or split them into separate containers.

---

## Complete example NGINX

A complete, working derived image. Since the container runs unprivileged, NGINX cannot bind a
port below 1024 and cannot write to its usual runtime locations — hence the adjustments below.

`Dockerfile`
```dockerfile
FROM ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest

USER root
RUN apt-get update && \
    apt-get install -y --no-install-recommends nginx && \
    rm -rf /var/lib/apt/lists/* && \
    # Listen on an unprivileged port (IPv4 and IPv6 lines both end in
    # "80 default_server;", so only that suffix is replaced).
    sed -i 's/80 default_server;/8080 default_server;/g' /etc/nginx/sites-enabled/default && \
    # The "user" directive only applies when the master runs as root.
    sed -i '/^user /d' /etc/nginx/nginx.conf && \
    # /run is not writable by appuser.
    sed -i 's#pid /run/nginx.pid;#pid /tmp/nginx.pid;#' /etc/nginx/nginx.conf && \
    # Send logs to the container's stdout/stderr.
    ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log && \
    # Cache and temp directories must be writable by appuser.
    chown -R appuser:appuser /var/lib/nginx
USER appuser

EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]
```

`docker-compose.yml`
```yaml
services:
  web:
    build: .
    container_name: nginx-web
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - ./html:/var/www/html
    environment:
      TZ: "Europe/Paris"
```

```bash
mkdir -p html && echo '<h1>It works</h1>' > html/index.html
docker compose up -d --build
curl http://localhost:8080
```

> A bind mount **replaces** the directory's contents in the image. Mounting an empty `./html`
> over `/var/www/html` removes NGINX's default page, and NGINX answers `403 Forbidden` because it
> has no `index.html` to serve. Put a file there first, as above.

---

## Security model

The container has **no privileged process**: the image ends with `USER appuser`, so everything —
including PID 1 — runs unprivileged.

| Control | Implementation |
|---|---|
| Container user | `appuser` (UID `1000`), set with `USER` — no root process |
| `root` account | Password locked (`passwd -l`), `/root` mode `700` |
| Login shell for `appuser` | `/usr/sbin/nologin` |
| SUID/SGID binaries | Stripped image-wide at build time |
| World-writable files | Write bit removed image-wide at build time |
| Default umask | `027` |
| `/config` | Mode `750`, owned by `appuser` |
| Service managers | `policy-rc.d` and `initctl` neutralised |
| Supply chain | Alpine builder pinned by digest; CI actions pinned by commit SHA |

Recommended runtime hardening for your deployments:

```yaml
services:
  app:
    image: ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    read_only: true
    tmpfs:
      - /tmp
```

### Base image support status

Ubuntu 18.04 LTS left standard support on **31 May 2023**. Its public archive no longer receives
new security updates: those are published through Ubuntu Pro (ESM), which this image does not
subscribe to. Two consequences worth stating plainly:

- The monthly rebuild refreshes the image against what Ubuntu still serves for Bionic. It does
  **not** bring in fixes that only exist behind ESM, so a CVE fixed for 20.04 or 22.04 may stay
  open here indefinitely.
- Trivy itself flags this on every scan (*"This OS version is no longer supported by the
  distribution"*): Ubuntu no longer issues advisories for Bionic, so Trivy's data for it stops
  where those advisories stopped. The scans deliberately report vulnerabilities that have **no fix
  available** — on a frozen archive that is most of them.

Use this image where an 18.04 userland is a hard requirement — legacy binaries, an old toolchain,
reproducing a historical environment. For anything new, start from a supported Ubuntu LTS. If you
must stay on 18.04 in production, add an Ubuntu Pro subscription inside your derived image.

### Verifying what you are running

Every image carries OCI provenance labels — the exact commit it was built from, and when:

```bash
docker inspect --format '{{json .Config.Labels}}' \
  ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest | jq
```

Pin by digest to guarantee byte-for-byte reproducibility:

```bash
docker pull ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test@sha256:<digest>
```

Vulnerability reporting is covered in [`SECURITY.md`](./SECURITY.md).

---

## Troubleshooting

### Getting a shell inside a running container

```bash
docker exec -it <container> bash          # as appuser
docker exec -it -u 0 <container> bash     # as root, for debugging
docker logs <container>
```

### Common problems

**`E: Could not open lock file /var/lib/apt/lists/lock (13: Permission denied)`**
APT is running unprivileged. Install packages at build time in your `Dockerfile` — and remember to
`USER root` before the `RUN`, then `USER appuser` after.

**`bind() to 0.0.0.0:80 failed (13: Permission denied)`**
Unprivileged processes cannot bind ports below 1024. Use a port ≥ 1024 inside the container and
remap it on the host (`-p 80:8080`).

**Setting `-e PUID=…` changes nothing**
Expected: `appuser` has a fixed UID of `1000` baked in at build time. Either rebuild with the IDs
you need, or adjust ownership on the host: `sudo chown -R 1000:1000 ./data`.

**Files created in a bind mount are owned by UID 1000 on the host**
That is `appuser`. Either run your host tooling under a matching UID, or `chown` the directory to
your own user after the fact.

**`Permission denied` writing to a mounted volume**
The host directory is not writable by UID `1000`. Fix it on the host:
`sudo chown -R 1000:1000 ./data`.

**`docker stop` takes ~10 seconds**
Your `CMD` is running as PID 1 and ignoring `SIGTERM`. Run it with `--init` (or `init: true` in
Compose) so a proper init forwards signals for it.

**Zombie processes accumulate**
Same cause — PID 1 is not reaping children. Use `--init`.

---

## Maintenance

- **Images are rebuilt monthly** (1st of the month, 03:00 UTC) against the current Bionic
  archive, and can be triggered manually from the Actions tab. See
  [Base image support status](#base-image-support-status) for what that does and does not cover.
- **Vulnerabilities are scanned weekly** (Mondays, 04:00 UTC) and after every build that published
  an image, with Trivy, on both architectures. Findings with no fix available are included. Full
  JSON reports are kept as build artifacts for 90 days, and every run writes a summary table to
  its workflow page. Results also go to the **Security → Code scanning** tab, which requires code
  scanning to be enabled in *Settings → Code security*; when it is not, the scan still runs and
  says so in its summary.
- **Every pull request runs hadolint and the integration tests** on amd64 and arm64. Publishing
  waits on both, and never happens from a pull request.
- **The Docker Hub description** is a separate file, `README-dockerhub.md`, synchronised by its
  own workflow when that file changes.

Source: [`Dockerfile-multi-arch`](./Dockerfile-multi-arch).
Contributions are welcome: see [`CONTRIBUTING.md`](./CONTRIBUTING.md) and the
[`CODE_OF_CONDUCT.md`](./CODE_OF_CONDUCT.md).

---
---

# Documentation française

<p align="center">
  <a href="#docker-ubuntu-1804-lts-bionic---sam-tech-lab">English version ↑</a> · <b>Français</b>
</p>

## Démarrage rapide

```bash
# Lancer un shell (en tant qu'appuser, non privilégié)
docker run -it --rm ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest

# Ou depuis Docker Hub
docker run -it --rm samtechlab/ubuntu-18.04-bionic-test:latest
```

Construire par-dessus :

```dockerfile
FROM ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest

# L'image tourne en appuser : repasser en root pour installer, puis redescendre.
USER root
RUN apt-get update && \
    apt-get install -y --no-install-recommends votre-paquet && \
    rm -rf /var/lib/apt/lists/*
USER appuser
```

**Sommaire :** [Présentation](#présentation) · [Référence de l'image](#référence-de-limage) ·
[Prise en main](#prise-en-main) · [Exemple complet NGINX](#exemple-complet-nginx) ·
[Modèle de sécurité](#modèle-de-sécurité) · [Dépannage](#dépannage) ·
[Maintenance](#maintenance-1)

---

## Présentation

Une **image de base Ubuntu 18.04 LTS (Bionic) minimale, durcie et multi-architecture**, construite
`FROM scratch` à partir du **rootfs OCI officiel d'Ubuntu** — pour l'authenticité, la légèreté et
la reproductibilité.

Elle est conçue comme une **fondation pour vos propres images** : un système Ubuntu complet (sans noyau — les conteneurs
partagent celui de l'hôte), un
utilisateur non-root, un socle système durci — puis elle vous laisse travailler.

> **Architectures supportées :** `linux/amd64`, `linux/arm64`
> **Reconstructions mensuelles automatiques** intégrant ce qu'Ubuntu publie encore pour Bionic —
> lisez [État du support de l'image de base](#état-du-support-de-limage-de-base) avant tout
> déploiement en production.

### Points forts

- ✅ **Construite `FROM scratch`** depuis le rootfs OCI officiel Ubuntu — aucune couche de base tierce
- ✅ **Multi-arch** publiée sous un manifeste unique (`amd64`, `arm64`)
- ✅ **Exécution en utilisateur non-root** (`appuser`) de bout en bout — aucun processus privilégié
- ✅ **Durcissement système** — compte `root` verrouillé, bits SUID/SGID supprimés, bits
  world-writable retirés, `umask 027`
- ✅ **Intégrité de la chaîne d'approvisionnement** — builder Alpine figé par digest, actions CI
  figées par SHA de commit
- ✅ **Optimisation APT & dpkg** — aucun paquet recommandé/suggéré, aucune traduction, cache propre
- ✅ **Gestionnaires de services neutralisés** (`systemd`, `upstart`) pour que les paquets
  n'essaient pas de démarrer de daemons
- ✅ **Locale et fuseau horaire configurés** (`en_US.UTF-8`, `UTC`)
- ✅ **SBOM et provenance SLSA** joints à chaque image publiée
- ✅ **Vérifiée en continu** — hadolint, 9 tests d'intégration sur conteneur joués sur **les deux
  architectures**, scans Trivy hebdomadaires

---

## Référence de l'image

### Registres et tags

| Registre | Image | Architectures |
|---|---|---|
| Docker Hub | `samtechlab/ubuntu-18.04-bionic-test:latest` | amd64 + arm64 |
| Docker Hub | `samtechlab/ubuntu-18.04-bionic-test:YYYY.MM` | amd64 + arm64 |
| GHCR | `ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest` | amd64 + arm64 |
| GHCR | `ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:YYYY.MM` | amd64 + arm64 |

Les tags pointent vers un manifeste multi-architecture : Docker sélectionne automatiquement
l'image correspondant à la plateforme hôte. `latest` suit la reconstruction mensuelle.

**Aucun de ces tags n'est immuable.** Un tag `YYYY.MM` désigne le mois où un build a eu lieu, pas
un build en particulier : tout build de ce mois-là le republie. Le 2 août 2026, six builds ont
tous écrit `2026.08`.

Pour une image réellement figée, **épinglez par digest** — voir
[Vérifier ce que vous exécutez](#vérifier-ce-que-vous-exécutez).

### Paquets inclus

| Catégorie | Paquets |
|---|---|
| Shell & base | `bash`, `cron`, `curl`, `gnupg`, `jq`, `netcat-openbsd`, `tzdata` |
| Outils système | `apt-utils`, `ca-certificates`, `locales` |

### Variables d'environnement

| Variable | Défaut | Description |
|---|---|---|
| `HOME` | `/config` | Répertoire personnel de `appuser` |
| `TZ` | `UTC` | Fuseau horaire |
| `LANG` | `en_US.UTF-8` | Locale (également `LANGUAGE`, `LC_ALL`) |
| `TERM` | `xterm` | Type de terminal |
| `DEBIAN_FRONTEND` | `noninteractive` | Supprime les invites APT interactives |
| `PATH` | `/usr/local/sbin:/usr/local/bin:…` | Chemin système standard |

> **`PUID` et `PGID` sont des arguments de build, pas des réglages d'exécution.** Ils sont figés
> à la création de `appuser` (`1000:1000`), et rien ne les applique au démarrage du conteneur :
> cette image n'a pas de système d'init. Passer `-e PUID=1001` à `docker run` **n'a aucun effet**.
> Pour d'autres identifiants :
>
> ```bash
> docker buildx build -f Dockerfile-multi-arch \
>   --build-arg PUID=1001 --build-arg PGID=1001 -t mon-bionic .
> ```
>
> …ou alignez les permissions côté hôte (voir [Dépannage](#dépannage)).

### Arborescence

| Chemin | Rôle |
|---|---|
| `/config` | Home de `appuser`, mode `750`, appartenant à `appuser` — montez vos données persistantes ici |

### Comportement par défaut

| Réglage | Valeur |
|---|---|
| Utilisateur | `appuser` (UID `1000`, GID `1000`) |
| Shell | `SHELL ["/bin/bash", "-c"]` |
| Commande | `CMD ["/bin/bash"]` |
| Entrypoint | aucun |

---

## Prise en main

### Lancer un conteneur

```bash
docker run -it --rm ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest
```

Vous obtenez un shell `bash` en tant que `appuser`. Aucun processus root ne tourne dans le
conteneur.

### Construire votre propre image

L'image se termine par `USER appuser` : **toute image dérivée doit donc repasser en `root` pour
installer des paquets**, puis redescendre :

```dockerfile
FROM ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest

USER root
RUN apt-get update && \
    apt-get install -y --no-install-recommends nginx && \
    rm -rf /var/lib/apt/lists/*
USER appuser

CMD ["nginx", "-g", "daemon off;"]
```

> **N'installez pas de paquets au démarrage du conteneur** (par ex. un `apt-get install` dans
> `command:`). Le conteneur tourne sans privilèges, APT échouera donc avec `Permission denied`
> sur `/var/lib/apt/lists`. Installez au build, comme ci-dessus.

### Processus et signaux

L'image ne contient pas de système d'init : votre `CMD` s'exécute directement en PID 1. C'est
suffisant pour un unique processus bien conçu, mais le PID 1 a des responsabilités particulières
sous Linux — il doit récupérer les processus fils orphelins et gérer lui-même les signaux. Un
processus qui ne fait ni l'un ni l'autre laisse des zombies, ou ignore `SIGTERM`, obligeant
`docker stop` à attendre son délai d'expiration avant de le tuer.

Si votre processus n'est pas prévu pour ce rôle, laissez Docker fournir un init minimal :

```bash
docker run --init ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest votre-commande
```

```yaml
services:
  app:
    image: ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest
    init: true
```

Si vous devez superviser **plusieurs** processus dans un même conteneur, un simple `CMD` ne suffit
pas : utilisez un véritable système d'init, ou séparez-les en plusieurs conteneurs.

---

## Exemple complet NGINX

Une image dérivée complète et fonctionnelle. Le conteneur tournant sans privilèges, NGINX ne peut
ni se lier à un port inférieur à 1024, ni écrire dans ses emplacements d'exécution habituels —
d'où les ajustements ci-dessous.

`Dockerfile`
```dockerfile
FROM ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest

USER root
RUN apt-get update && \
    apt-get install -y --no-install-recommends nginx && \
    rm -rf /var/lib/apt/lists/* && \
    # Écouter sur un port non privilégié (les lignes IPv4 et IPv6 se
    # terminent toutes deux par "80 default_server;", seul ce suffixe
    # est remplacé).
    sed -i 's/80 default_server;/8080 default_server;/g' /etc/nginx/sites-enabled/default && \
    # La directive "user" ne s'applique que si le master tourne en root.
    sed -i '/^user /d' /etc/nginx/nginx.conf && \
    # /run n'est pas accessible en écriture par appuser.
    sed -i 's#pid /run/nginx.pid;#pid /tmp/nginx.pid;#' /etc/nginx/nginx.conf && \
    # Rediriger les logs vers stdout/stderr du conteneur.
    ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log && \
    # Les répertoires de cache et temporaires doivent être accessibles
    # en écriture par appuser.
    chown -R appuser:appuser /var/lib/nginx
USER appuser

EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]
```

`docker-compose.yml`
```yaml
services:
  web:
    build: .
    container_name: nginx-web
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - ./html:/var/www/html
    environment:
      TZ: "Europe/Paris"
```

```bash
mkdir -p html && echo '<h1>Ça marche</h1>' > html/index.html
docker compose up -d --build
curl http://localhost:8080
```

> Un montage de volume **remplace** le contenu du répertoire dans l'image. Monter un `./html`
> vide sur `/var/www/html` supprime la page par défaut de NGINX, et NGINX répond
> `403 Forbidden` faute d'`index.html` à servir. Créez d'abord un fichier, comme ci-dessus.

---

## Modèle de sécurité

Le conteneur n'a **aucun processus privilégié** : l'image se termine par `USER appuser`, donc tout
— y compris le PID 1 — s'exécute sans privilèges.

| Contrôle | Mise en œuvre |
|---|---|
| Utilisateur du conteneur | `appuser` (UID `1000`), défini via `USER` — aucun processus root |
| Compte `root` | Mot de passe verrouillé (`passwd -l`), `/root` en mode `700` |
| Shell de connexion de `appuser` | `/usr/sbin/nologin` |
| Binaires SUID/SGID | Supprimés sur toute l'image au build |
| Fichiers world-writable | Bit d'écriture retiré sur toute l'image au build |
| Umask par défaut | `027` |
| `/config` | Mode `750`, appartenant à `appuser` |
| Gestionnaires de services | `policy-rc.d` et `initctl` neutralisés |
| Chaîne d'approvisionnement | Builder Alpine figé par digest ; actions CI figées par SHA de commit |

Durcissement recommandé à l'exécution pour vos déploiements :

```yaml
services:
  app:
    image: ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    read_only: true
    tmpfs:
      - /tmp
```

### État du support de l'image de base

Ubuntu 18.04 LTS est sorti du support standard le **31 mai 2023**. Son archive publique ne reçoit
plus de nouvelles mises à jour de sécurité : celles-ci passent par Ubuntu Pro (ESM), auquel cette
image n'est pas abonnée. Deux conséquences à énoncer clairement :

- La reconstruction mensuelle rafraîchit l'image à partir de ce qu'Ubuntu sert encore pour Bionic.
  Elle n'apporte **pas** les correctifs qui n'existent que derrière l'ESM : une CVE corrigée pour
  20.04 ou 22.04 peut rester ouverte ici indéfiniment.
- Trivy le signale à chaque analyse (*« This OS version is no longer supported by the
  distribution »*) : Ubuntu ne publie plus d'avis de sécurité pour Bionic, et les données de Trivy
  s'arrêtent donc là où ces avis se sont arrêtés. Les analyses remontent volontairement les
  vulnérabilités **sans correctif disponible** — sur une archive figée, c'est la majorité d'entre
  elles.

Utilisez cette image là où un userland 18.04 est une contrainte dure — binaires hérités, ancienne
chaîne de compilation, reproduction d'un environnement historique. Pour tout nouveau projet,
partez d'une LTS Ubuntu encore supportée. Si vous devez rester sur 18.04 en production, ajoutez un
abonnement Ubuntu Pro dans votre image dérivée.

### Vérifier ce que vous exécutez

Chaque image porte des labels de provenance OCI — le commit exact dont elle est issue, et sa date
de construction :

```bash
docker inspect --format '{{json .Config.Labels}}' \
  ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test:latest | jq
```

Figez par digest pour garantir une reproductibilité au bit près :

```bash
docker pull ghcr.io/sam-tech-lab-git/ubuntu-18.04-bionic-test@sha256:<digest>
```

Le signalement de vulnérabilités est décrit dans [`SECURITY.md`](./SECURITY.md).

---

## Dépannage

### Obtenir un shell dans un conteneur en cours d'exécution

```bash
docker exec -it <conteneur> bash          # en tant qu'appuser
docker exec -it -u 0 <conteneur> bash     # en tant que root, pour déboguer
docker logs <conteneur>
```

### Problèmes courants

**`E: Could not open lock file /var/lib/apt/lists/lock (13: Permission denied)`**
APT s'exécute sans privilèges. Installez les paquets au build dans votre `Dockerfile` — et
pensez à `USER root` avant le `RUN`, puis `USER appuser` après.

**`bind() to 0.0.0.0:80 failed (13: Permission denied)`**
Un processus non privilégié ne peut pas se lier aux ports inférieurs à 1024. Utilisez un port
≥ 1024 dans le conteneur et remappez-le côté hôte (`-p 80:8080`).

**Passer `-e PUID=…` ne change rien**
C'est attendu : `appuser` a un UID fixe de `1000`, figé au build. Reconstruisez l'image avec les
identifiants voulus, ou ajustez la propriété côté hôte : `sudo chown -R 1000:1000 ./data`.

**Les fichiers créés dans un volume appartiennent à l'UID 1000 sur l'hôte**
C'est `appuser`. Faites tourner vos outils hôte sous un UID correspondant, ou faites un `chown`
du répertoire vers votre propre utilisateur ensuite.

**`Permission denied` en écriture sur un volume monté**
Le répertoire hôte n'est pas accessible en écriture par l'UID `1000`. Corrigez côté hôte :
`sudo chown -R 1000:1000 ./data`.

**`docker stop` met une dizaine de secondes**
Votre `CMD` tourne en PID 1 et ignore `SIGTERM`. Lancez-le avec `--init` (ou `init: true` en
Compose) pour qu'un véritable init relaie les signaux à sa place.

**Des processus zombies s'accumulent**
Même cause : le PID 1 ne récupère pas ses fils. Utilisez `--init`.

---

## Maintenance

- **Les images sont reconstruites chaque mois** (le 1er, à 03h00 UTC) à partir de l'archive
  Bionic courante, et peuvent être déclenchées manuellement depuis l'onglet Actions. Voir
  [État du support de l'image de base](#état-du-support-de-limage-de-base) pour ce que cela
  couvre — et ne couvre pas.
- **Les vulnérabilités sont scannées chaque semaine** (lundi, 04h00 UTC) et après chaque build
  ayant publié une image, avec Trivy, sur les deux architectures. Les vulnérabilités sans
  correctif disponible sont incluses. Les rapports JSON complets sont conservés 90 jours en
  artefacts de build, et chaque run écrit un tableau de synthèse sur sa page de workflow. Les
  résultats vont aussi dans l'onglet **Security → Code scanning**, ce qui suppose le code scanning
  activé dans *Settings → Code security* ; sinon l'analyse tourne quand même et le signale.
- **Chaque pull request exécute hadolint et les tests d'intégration** sur amd64 et arm64. La
  publication attend les deux, et n'a jamais lieu depuis une pull request.
- **La description Docker Hub** est un fichier distinct, `README-dockerhub.md`, synchronisé par
  son propre workflow quand ce fichier change.

Source : [`Dockerfile-multi-arch`](./Dockerfile-multi-arch).
Les contributions sont bienvenues : voir [`CONTRIBUTING.md`](./CONTRIBUTING.md) et le
[`CODE_OF_CONDUCT.md`](./CODE_OF_CONDUCT.md).

---

## License / Licence

This project is distributed under the **MIT** license — see the [LICENSE](./LICENSE) file for more details.

Ce projet est distribué sous la licence **MIT** — consultez le fichier [LICENSE](./LICENSE) pour plus de détails.

---

## Copyright / Droit d'auteur

```text
Copyright (c) 2025 Sam Tech Lab
```
