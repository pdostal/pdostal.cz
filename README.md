# pdostal.cz

This is the initial commit of my new page / blog powered by [hugo].
Here is also one submodule pointing to modified [harbor] theme.

[hugo]: https://gohugo.io
[harbor]: https://themes.gohugo.io/harbor/

## tmp

```
$ podman run --rm -v "$PWD":/src -w /src docker.io/klakegg/hugo:ext build && zip -r /tmp/site.zip public
$ podman run --rm -v "$PWD":/src -p 0.0.0.0:1313:1313 -w /src docker.io/klakegg/hugo:ext serve
```

