# ci-workflows

Shared GitHub Actions for the Pubky organisation. Changing anything here changes the
builds of every consuming repository, so read the policy below before opening a PR.

## Actions

| Action | Purpose |
|---|---|
| `docker/build_and_push` | Build a multi-arch image and push it to GCP Artifact Registry or Docker Hub |
| `docker/registry_login` | Log in to either registry, chosen from the registry hostname |
| `docker/check_if_image_exists` | Check whether a tag already exists for a given architecture |
| `docker/get_head_commit_hash` | **Deprecated.** `build_and_push` resolves the hash itself |

```yaml
- uses: pubky/ci-workflows/.github/actions/docker/registry_login@main
  with:
    registry: europe-west6-docker.pkg.dev/infra-464608/synonym-private-repo
    gcp_service_account_key: ${{ secrets.GCR_JSON_KEY }}

- uses: pubky/ci-workflows/.github/actions/docker/build_and_push@main
  with:
    registry: europe-west6-docker.pkg.dev/infra-464608/synonym-private-repo
    image: my-service
    context: .
```

## Policy

**Third-party actions are pinned to a commit SHA**, with the version in a trailing comment:

```yaml
uses: docker/login-action@c94ce9fb468520275223c153574b00df6fe4bcc9 # v3
```

A tag can be moved by its owner; a SHA cannot. The comment is not decoration — Dependabot
reads it to work out which version a pin corresponds to, and raises bump PRs weekly with a
7-day cooldown so a freshly published release is never proposed the moment it lands.

**No action here may reference another action in this repository.** Doing so reintroduces a
floating `@main` that a consumer cannot pin past.

**`zizmor` runs on every pull request and blocks the merge.** Results are uploaded to the
repository's Security tab as SARIF. To reproduce locally:

```
docker run --rm -e GH_TOKEN="$(gh auth token)" -v "$PWD:/src:ro" -w /src \
  ghcr.io/zizmorcore/zizmor:1.30.0 .
```

## Versioning

Releases are tagged from `v1.0.0`. Consumers currently reference `@main`; the tags exist so
that a later migration to pinned references has real version history to pin against.
