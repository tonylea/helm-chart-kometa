# Kometa Helm Chart

![Downloads](https://img.shields.io/github/downloads/tonylea/helm-chart-kometa/total)

Deploys [Kometa](https://github.com/Kometa-Team/Kometa) from its official image, [`kometateam/kometa`](https://hub.docker.com/r/kometateam/kometa).

Kometa runs as a Kubernetes CronJob: each run is one `--run` that exits when it's done, and a run never starts while the last one is still going.
A failed run isn't retried, since a retry repeats the whole run against Plex; its pod is kept for the logs.
Kometa's config directory, `/config`, is a persistent volume holding its SQLite cache, logs, reports and poster backups.

## Values

| Value | Default |
| --- | --- |
| `image.tag` | the chart's `appVersion` |
| `user` | `1000` |
| `group` | `1000` |
| `schedule` | `@daily` |
| `timeZone` | `Etc/UTC` |
| `config` | none; required |
| `files` | none |
| `secretEnv` | none |
| `resources` | none |
| `persistence.existingClaim` | none; the chart creates a claim |
| `persistence.storageClassName` | the cluster's default class |
| `persistence.size` | `10Gi` |
| `assets.existingClaim` | none; Kometa's asset directory stays on the config volume |
| `affinity` | none |

`config` is the content of [`config.yml`](https://kometa.wiki/en/latest/config/overview/).
`files` maps a file name to the content of a collection, overlay or playlist file, mounted at `/config/<name>`, so `config.yml` refers to it as `config/<name>`.
`secretEnv` names an existing Secret whose keys become env vars: a key `KOMETA_<NAME>` is read in `config.yml` as `<<name>>`, and the name can't contain underscores.
`timeZone` is used for the schedule and given to Kometa as `TZ`.

## Fixed settings

- Kometa runs with `--read-only-config`, since `config.yml` and the extra files are mounted read-only: they can only be changed through `config` and `files`.
- Kometa runs as `user` and `group`, never root, with a read-only root filesystem and no capabilities.
  The image has no `USER` of its own, so it would otherwise run as root.
  `/tmp` is an emptyDir.

## Requirements

- Plex, reachable from the namespace, and a Secret holding its token and any API keys `config.yml` reads.
- A storage class for the claim the chart creates, the cluster's default unless named.
  The cache is a SQLite database, so `/config` needs block storage, not a network share such as NFS, whose locking SQLite can't rely on.
  Without the cache, every run applies every overlay again.
- For `assets.existingClaim`, the claim and its directory: Kometa doesn't create its asset directory.

## Usage

```sh
helm repo add kometa https://tonylea.github.io/helm-chart-kometa
helm install kometa kometa/kometa --values my-values.yaml
```

```yaml
schedule: "0 5 * * *"
timeZone: Europe/London
secretEnv: kometa
config: |
  libraries:
    Movies:
      collection_files:
        - default: franchise
  plex:
    url: http://plex:32400
    token: <<plextoken>>
  tmdb:
    apikey: <<tmdbkey>>
```

With a Secret `kometa` holding the keys `KOMETA_PLEXTOKEN` and `KOMETA_TMDBKEY`.

## Development

Pull requests run:

- `helm lint --strict`, and the rendered chart checked by kubeconform against Kubernetes 1.37.
- helm-unittest's tests in `tests/`.
- A fresh install with chart-testing in a kind cluster, against the claim in `ci/kind`.
  kind binds a claim only once a pod uses it, and the CronJob starts none at install, so the claim is made outside the release.

Commits follow Conventional Commits.
Releases are versioned and tagged automatically from `main`, and published to the Helm repository on this repo's GitHub Pages.
