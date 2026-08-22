# Security Policy — Docker Ubuntu 18.04 LTS (Bionic)

This repository publishes a **minimal, hardened Ubuntu 18.04 (Bionic) base image** for container use.

---

## Supported Versions

| Version | Status |
|---|---|
| `latest` | ✅ Supported |
| Latest monthly tag (`YYYY.MM`) | ✅ Supported |
| Older tags | ❌ Not supported |

Only the most recent published tags receive updates and security follow-up.

---

## Security Baseline

The image includes the following default protections:

- non-root runtime user: `appuser`
- locked `root` account
- restrictive `umask 027`
- reduced SUID/SGID exposure
- blocked automatic service start during package installation
- cleaned APT caches, temp files, and logs during build
- official Ubuntu OCI rootfs as the base source
- **no privileged process**: the image ends with `USER appuser`, so PID 1 itself runs unprivileged
- SBOM and SLSA provenance attestations published alongside every image
- Alpine builder pinned by digest; CI actions pinned by commit SHA

### `PUID` / `PGID` are fixed

`appuser` is created at `1000:1000` when the image is built. This image has no init system, so
nothing remaps it at container start: `-e PUID=…` has no effect. Rebuild with
`--build-arg PUID=… --build-arg PGID=…` if you need different IDs.

### Base image support status

Ubuntu 18.04 LTS left standard support on **31 May 2023**. Its public archive no longer receives
new security updates — those are published through Ubuntu Pro (ESM), which this image does not
subscribe to. Monthly rebuilds therefore refresh the image against a frozen archive, and known
CVEs may remain unfixed. Treat this image as suitable for legacy or reproducibility needs, not as
a maintained production base.

---

## Vulnerability Scanning

Security scanning is automated with [Trivy](https://github.com/aquasecurity/trivy).

### Reports are published to

| Location | Format | Access |
|---|---|---|
| GitHub **Security → Code scanning** | SARIF | Repository security tab |
| GitHub Actions step summary | Markdown | Workflow run summary |
| GitHub Actions artifacts | JSON | Downloadable artifact |

The scan workflow runs:
- every week on Monday at **04:00 UTC**
- automatically after every build workflow that published an image
- manually through GitHub Actions if needed

Both architectures are scanned. Vulnerabilities with **no fix available** are reported rather than
filtered out: Bionic's archive is frozen, so most of its exposure has no fix, and hiding it would
make the image look clean. Expect the reports to be long — that is the accurate picture.

SARIF upload requires code scanning to be enabled under *Settings → Code security*. When it is
not, the scan still runs: the JSON report and the run summary remain available, and the workflow
records a warning instead of failing.

---

## Reporting a Vulnerability

If you discover a security issue in this repository, please **do not open a public issue**.

Use GitHub private vulnerability reporting instead:

1. Open the repository **[Security tab](https://github.com/Sam-Tech-Lab-Git/Docker-Ubuntu-18.04-Bionic-Test/security)**
2. Click **Report a vulnerability**
3. Provide:
   - a clear description
   - reproduction steps
   - impact and affected area

### Response targets

- acknowledgment within **5 business days**
- mitigation or remediation target within **30 days**, depending on severity

---

## Scope

### In scope
- Dockerfile misconfigurations
- privilege escalation risks
- exposed secrets or CI/CD injection issues
- vulnerable packages present in the published image

### Out of scope
- upstream Ubuntu issues with no available fix yet
- vulnerabilities in third-party images built on top of this image

---

## Version française

## Versions supportées

| Version | Statut |
|---|---|
| `latest` | ✅ Supportée |
| Dernier tag mensuel (`YYYY.MM`) | ✅ Supporté |
| Anciens tags | ❌ Non supportés |

Seules les versions les plus récentes publiées reçoivent un suivi de sécurité.

## Mesures de sécurité

L’image applique notamment :
- un utilisateur non-root par défaut : `appuser`
- le verrouillage du compte `root`
- un `umask 027`
- la réduction des bits SUID/SGID inutiles
- le blocage du démarrage automatique des services à l’installation
- le nettoyage des caches APT, fichiers temporaires et journaux
- **aucun processus privilégié** : l'image se termine par `USER appuser`, donc PID 1 lui-même
  s'exécute sans privilèges
- des attestations SBOM et de provenance SLSA publiées à côté de chaque image
- le builder Alpine figé par digest, les actions CI figées par SHA de commit

### `PUID` / `PGID` sont figés

`appuser` est créé en `1000:1000` à la construction. Cette image n'a pas de système d'init : rien
ne le remappe au démarrage du conteneur, et `-e PUID=…` n'a aucun effet. Reconstruisez avec
`--build-arg PUID=… --build-arg PGID=…` si vous avez besoin d'autres identifiants.

### État du support de l'image de base

Ubuntu 18.04 LTS est sorti du support standard le **31 mai 2023**. Son archive publique ne reçoit
plus de nouvelles mises à jour de sécurité : celles-ci passent par Ubuntu Pro (ESM), auquel cette
image n'est pas abonnée. Les reconstructions mensuelles rafraîchissent donc l'image à partir d'une
archive figée, et des CVE connues peuvent rester non corrigées. Considérez cette image comme
adaptée à des besoins de compatibilité ou de reproductibilité, non comme une base de production
maintenue.

## Analyse des vulnérabilités

L'analyse de sécurité est automatisée avec [Trivy](https://github.com/aquasecurity/trivy).

### Les rapports sont publiés dans

| Emplacement | Format | Accès |
|---|---|---|
| GitHub **Security → Code scanning** | SARIF | Onglet Security du dépôt |
| Résumé d'étape GitHub Actions | Markdown | Résumé du run de workflow |
| Artefacts GitHub Actions | JSON | Artefact téléchargeable |

Le workflow d'analyse s'exécute :
- chaque semaine, le lundi à **04h00 UTC**
- automatiquement après chaque workflow de build ayant publié une image
- manuellement via GitHub Actions si besoin

Les deux architectures sont analysées. Les vulnérabilités **sans correctif disponible** sont
remontées plutôt que filtrées : l'archive Bionic étant figée, l'essentiel de l'exposition n'a pas
de correctif, et la masquer donnerait l'image d'un système sain. Les rapports sont donc longs —
c'est le reflet fidèle de la situation.

L'envoi du SARIF suppose le code scanning activé dans *Settings → Code security*. Sinon l'analyse
tourne quand même : le rapport JSON et le résumé du run restent disponibles, et le workflow émet
un avertissement au lieu d'échouer.

## Signalement d’une vulnérabilité

Merci de **ne pas ouvrir d’issue publique**.

Utilisez le signalement privé GitHub :
1. ouvrir l’onglet **[Security](https://github.com/Sam-Tech-Lab-Git/Docker-Ubuntu-18.04-Bionic-Test/security)**
2. cliquer sur **Report a vulnerability**
3. décrire le problème, les étapes de reproduction et l’impact

Objectifs de réponse :
- accusé de réception sous **5 jours ouvrés**
- correction ou mitigation visée sous **30 jours** selon la sévérité

## Périmètre

### Dans le périmètre
- mauvaises configurations du Dockerfile
- risques d'élévation de privilèges
- secrets exposés ou problèmes d'injection CI/CD
- paquets vulnérables présents dans l'image publiée

### Hors périmètre
- problèmes Ubuntu amont sans correctif disponible pour le moment
- vulnérabilités dans des images tierces construites à partir de cette image

---

## License / Licence

This project is distributed under the **MIT** license. See [`LICENSE`](./LICENSE).

Ce projet est distribué sous la licence **MIT**. Voir [`LICENSE`](./LICENSE).

```text
Copyright (c) 2026 Sam Tech Lab
```
