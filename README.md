# pdostal.cz

This is the initial commit of my new page / blog powered by [hugo].
Here is also one submodule pointing to modified [harbor] theme.

[hugo]: https://gohugo.io
[harbor]: https://themes.gohugo.io/harbor/

## Local development

```
$ podman run --rm -v "$PWD":/src -w /src docker.io/klakegg/hugo:ext build && zip -r /tmp/site.zip public
$ podman run --rm -v "$PWD":/src -p 0.0.0.0:1313:1313 -w /src docker.io/klakegg/hugo:ext serve
```

## Deployment

Every push to `master` triggers a GitHub Actions workflow that builds the site with Hugo and publishes a new GitHub release containing `site.tar.gz`. Each release is named with a timestamp (e.g. `30. 11. 2026 15:33:20`) and the release notes list all included commits with their short SHA and author.

Because GitHub always exposes the newest release under `/releases/latest/download/`, the server deployment script can stay static:

```sh
curl -fsSL https://github.com/pdostal/pdostal.cz/releases/latest/download/site.tar.gz \
  | tar -xz --strip-components=1 -C /var/www/pdostal.cz/public
```

