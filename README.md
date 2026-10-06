# Semaphore UI distfiles for the FreeBSD port

This repository provides the generated distfiles required by the FreeBSD port
[net-mgmt/semaphore](https://www.freshports.org/net-mgmt/semaphore/).

## Why these files are needed

Since Semaphore 2.10 the Vue web interface is embedded into the binary via
`go:embed`, but upstream publishes neither the built web assets nor a form of
the nested `pro` module (`replace github.com/semaphoreui/semaphore/pro => ./pro`)
that `GH_TUPLE` or the Go module proxy can consume.  Both artifacts therefore
have to be produced separately and are attached to the releases here:

| distfile | contents |
|---|---|
| `semaphore-webui-<VERSION>.tar.gz`     | the built web UI (`api/public`) |
| `semaphore-provendor-<VERSION>.tar.gz` | vendored `pro` module + matching `vendor/modules.txt` |

The port fetches them from the release tagged `semaphore-<VERSION>`, so the tag
name has to match `${PORTNAME}-${DISTVERSION}`.

## Producing the distfiles for a new release

Requires `node`, `npm` and a Go version matching `go.mod`.

```shell
V=2.19.7

git clone --depth 1 -b v${V} https://github.com/semaphoreui/semaphore.git
cd semaphore

# 1) web UI -> api/public
cd web
NODE_OPTIONS=--openssl-legacy-provider npm install
NODE_OPTIONS=--openssl-legacy-provider npm run build
cd ../api
tar czf ../../semaphore-webui-${V}.tar.gz public
cd ..

# 2) vendored pro module and modules.txt
go mod vendor
tar czf ../semaphore-provendor-${V}.tar.gz \
    vendor/modules.txt vendor/github.com/semaphoreui/semaphore/pro
cd ..

# 3) publish
gh release create semaphore-${V} \
    semaphore-webui-${V}.tar.gz semaphore-provendor-${V}.tar.gz \
    -R joneum/FreeBSD-Semaphore --title "semaphore ${V}"
```

Afterwards update `DISTVERSION` in the port, regenerate `GH_TUPLE`/`GL_TUPLE`
with `make gomod-vendor`, and run `make makesum`.

## Platform

- Built on FreeBSD 14 amd64
- Built on FreeBSD 15 amd64
- Built on FreeBSD 16 amd64

---

If you like my work, consider sponsoring me on [GitHub Sponsors](https://github.com/sponsors/joneum/).

## License

What is in this repository is BSD 2-Clause, see [LICENSE](LICENSE).
The distfiles attached to the releases are built from Semaphore UI
and stay under its licence.
