# governa-buildtool

Build pipeline orchestration for Go repos: validates `programVersion` constants in `cmd/*/main.go`, runs `go vet` and `go test`, builds and installs each `cmd/<target>` binary, and suggests the next patch tag from `git tag --list`. Exposes a `PostInstallHook` extension point on `Config` so callers can run custom validation between install and next-tag-suggest without forking the orchestration.

## Why

CLIs in the governa family share a common build flow: every `cmd/<x>/main.go` declares a `programVersion`, the build vets and tests, builds each target, installs it, and prints a suggested next tag. Before extraction, every governa-family repo carried a copy of this orchestration logic and silently drifted; the library exists to be the single source of truth, semver-versioned, picked up via `go get -u`.

The package is leaf-clean of governa governance — it knows nothing about AC files, CHANGELOG row shapes, or governance docs. It takes a `Config` and a pair of `io.Writer`s, and orchestrates the build. The `PostInstallHook` field lets callers (e.g. governa's own `cmd/build`) inject post-install steps without baking those steps into the library. Usable in any Go repo that wants the same `programVersion`-aware build flow.

## Install

    go get github.com/queone/governa-buildtool

## Usage

```go
import (
    "os"
    "github.com/queone/governa-buildtool"
)

func main() {
    cfg, helpRequested, err := buildtool.ParseArgs(os.Args[1:])
    if err != nil {
        fmt.Fprintln(os.Stderr, err)
        os.Exit(2)
    }
    if helpRequested {
        fmt.Print(buildtool.Usage())
        return
    }
    // Optional: register a hook that runs between install and next-tag-suggest.
    // cfg.PostInstallHook = func(out, errOut io.Writer) error { ... }
    if err := buildtool.Run(cfg, os.Stdout, os.Stderr); err != nil {
        fmt.Fprintln(os.Stderr, err)
        os.Exit(1)
    }
}
```

## Versioning

This library follows the policy in [governa/docs/library-policy.md](https://github.com/queone/governa/blob/main/docs/library-policy.md). See `CHANGELOG.md` for version history and deprecations.
