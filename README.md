# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) presets for projects built on the
[LucidModules Docker images](https://hub.docker.com/u/lucidmodules). Public, so every Renovate runner can read
it without a token. It holds rules only: no secrets, no runner.

## Presets

### `lucidmodules-images`

Keeps pinned `lucidmodules/*` image tags up to date:

- finds `lucidmodules/<image>:<tag>` in `.gitlab-ci.yml`, compose files and Dockerfiles, and Dockerfile `ARG`s
  marked with a comment on the line above:
  ```dockerfile
  # renovate: image=lucidmodules/docker-laravel-php
  ARG DOCKER_LARAVEL_PHP_VERSION=8.5.13-frankenphp1.13.6
  ```
- compares `docker-laravel-php` tags (`8.5.13-frankenphp1.13.6`) by PHP version, then FrankenPHP version;
- puts **patch** updates in one MR per project that merges itself when the MR pipeline passes;
- puts **minor/major** updates (PHP 8.5 → 8.6, Composer 2.10 → 2.11) in a separate MR that waits for review;
- proposes only tags already published, with no cooldown, because these images *are* the security fix.

Use it from a project's `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["https://raw.githubusercontent.com/LucidModules/renovate-config/master/lucidmodules-images.json"]
}
```

The raw URL avoids GitHub API rate limits on shared CI IPs.

## Runners

Renovate itself runs elsewhere: one runner per AWS account, each with its own least-privilege CI runner
and GitLab token. This repo holds only the rules they share.

## Changing a preset

A change here affects every project on every runner at its next run. Validate before merging:

```sh
docker run --rm -v "$PWD":/w -w /w --entrypoint renovate-config-validator renovate/renovate:<version> lucidmodules-images.json
```
