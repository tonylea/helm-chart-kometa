# Changelog

All notable changes to this project will be documented in this file. See [commit-and-tag-version](https://github.com/absolute-version/commit-and-tag-version) for commit guidelines.

## 1.0.0 (2026-11-22)

### Features

* run Kometa on a schedule, as a non-root CronJob with a read-only config ([cecafb6](https://github.com/tonylea/helm-chart-kometa/commit/cecafb6fa12efd1d53d3da5775c4ccd5dd6930a2))

### Bug Fixes

* default to a daily run at 05:00, London time ([d3bbc36](https://github.com/tonylea/helm-chart-kometa/commit/d3bbc36de7730eadbadb710accf3d45914ae0303))
* retry a failed run with Kubernetes' default back-off ([747231c](https://github.com/tonylea/helm-chart-kometa/commit/747231c1eb11b60b6a69e951134fffdcbadaa617))

### Documentation

* describe the chart, its values and how to contribute ([68b2ee1](https://github.com/tonylea/helm-chart-kometa/commit/68b2ee1bf7300bc5d52367b274ffaffb1640c317))
