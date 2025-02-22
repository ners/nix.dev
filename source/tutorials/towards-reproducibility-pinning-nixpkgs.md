(pinning-nixpkgs)=

# Towards reproducibility: pinning Nixpkgs

In various Nix examples, you'll often see references to [\<nixpkgs>](https://github.com/NixOS/nixpkgs), as follows.

```nix
{ pkgs ? import <nixpkgs> {}
}:

...
```

This is a **convenient** way to quickly demonstrate a Nix expression and get it working by importing Nix packages.

However, <ref-search-path>`the resulting Nix expression is not fully reproducible`.

## Pinning packages with URLs inside a Nix expression

To create **fully reproducible** Nix expressions, we can pin an exact version of Nixpkgs.

The simplest way to do this is to fetch the required Nixpkgs version as a tarball specified via the relevant Git commit hash:

```nix
{ pkgs ? import (fetchTarball "https://github.com/NixOS/nixpkgs/archive/3590f02e7d5760e52072c1a729ee2250b5560746.tar.gz") {}
}:

...
```

Picking the commit can be done via [status.nixos.org](https://status.nixos.org/),
which lists all the releases and the latest commit that has passed all tests.

When choosing a commit, it is recommended to follow either

- the **latest stable NixOS** release by using a specific version, such as `nixos-21.05`, **or**
- the latest **unstable release** via `nixos-unstable`.

## Dependency management with niv

If you'd like a bit more automation around bumping dependencies, including Nixpkgs,
[niv](https://github.com/nmattia/niv/) is made for exactly that. Niv itself is available
in `nixpkgs` so using it is simple:

```
$ nix-shell -p niv --run "niv init"
```

This command will generate `nix/sources.json` with information about how and where
dependencies are fetched. It will also create `nix/sources.nix`, which glues the sources together in Nix.

By default, `niv` will use the **latest stable** NixOS release. However, you should check to see which version is currently specified in [the niv repository](https://github.com/nmattia/niv) if you require a specific release, as it might lag behind.

You can see which version `niv` is tracking as follows:

```
$ niv show
```

And you can change the tracking branch to the one you want like this:

```
$ niv modify nixpkgs --branch nixos-21.05
```
You can use the generated `nix/sources.nix` with a top-level `default.nix`:

```nix
{ sources ? import ./nix/sources.nix
, pkgs ? import sources.nixpkgs {}
}:

...
```

And you can update all the dependencies by running:

```
$ nix-shell -p niv --run "niv update"
```

## Next steps

- For more examples and details of the different ways to pin `nixpkgs`, see {ref}`ref-pinning-nixpkgs`.
- To quickly set up a Nix project, read through
  [Getting started Nix template](https://github.com/nix-dot-dev/getting-started-nix-template).
