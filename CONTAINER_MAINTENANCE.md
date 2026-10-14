# Personal fork container maintenance

This fork publishes compatibility releases to `ghcr.io/adamcaoyu/subconverter-extended`.
The deployed release is `compat-v1.9.10-base64.1`. Keep a tested release tag pinned;
set `SUBCONVERTER_IMAGE` in the Compose environment when intentionally upgrading.

## Registry maintenance

The personal publishing workflow uses GitHub Actions build cache (`type=gha`).
It does not publish `ci-*` or `buildcache-*` registry tags. Daily registry pruning
is unnecessary for this deployment.

The inherited registry-cleanup script defaults to Aethersailor's Docker Hub and
GHCR packages. Its schedule is removed and its job is restricted to the upstream
repository. The workflow is also disabled in this personal repository's Actions
settings. Do not add credentials or run it against personal compatibility images:
its release-tag allowlist does not include `compat-v*`.

If obsolete personal releases eventually need removal, first inventory the
personal GHCR package and protect the deployed release plus rollback releases.
Treat that as a separate, explicitly scoped maintenance operation.

## Router Docker maintenance

Use the validated image, preserve the existing port and restart policy, and rotate
container logs with `json-file`, `max-size: 10m`, `max-file: 3`. The Compose example
already contains these logging options.

Persist `pref.toml` outside the container before recreating it. Mount just that
configuration file; mounting an entire old `/base` can hide the image's bundled
templates. Keep configuration backups private; they may contain credentials.

Do not schedule `docker system prune -a --volumes`. It can discard useful rollback
images and application data. Inspect `docker system df` before doing cleanup.
One active image with no unused images, volumes, or build cache needs no pruning.

For a container replacement, retain the original container as a stopped rollback
copy with restart disabled, verify the new converter, and leave OpenClash running.
Restarting this converter does not require restarting the proxy kernel.
