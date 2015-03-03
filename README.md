Cabal + Package = Cabbage
===

The `cabbage` script bridges the gap between Haskell's
[cabal](https://www.haskell.org/cabal/users-guide/) tooling and
[nix](https://github.com/NixOS/nixpkgs).

Usage
---

In a directory with a `.cabal` file, run `cabbage` to build Nix
expressions for every dependency pinned to a specific set of versions
identified by the cabal solver. Next, run `nix-shell --command 'sh
$setup'` to ensure all dependencies are available in the Nix store and
are linked into the cabal sandbox in the current development directory
(this sandbox will be created for you if one doesn't exist). From this
point on, you can `cabal configure`, etc. as you normally would with
cabal sandbox development.

To compile a library from [hackage](http://hackage.haskell.org) and
cache it in the Nix store, run `cabbage pandoc` (to install, e.g., the
`pandoc` package). This does not link the named package in your Nix
environment, but makes the package available for re-use in subsequent
builds. If the named package includes executables that you *do* want
to link into your environment, follow the instructions in the last
line of output from the cabbage command (i.e. `cabbage pandoc`).

To set flags for any package involved in a build, create a file
`cabbage.config` in your project's root directory. The file format
looks like this,

```
flags:
  PackageName: foo -bar
```

This will set flag `foo` to `True` and flag `bar` to `False` for the
package `PackageName`.

*NOTE*: The `cabbage` tool will not overwrite a `shell.nix` if one
exists, so, if you want `cabbage` to generate a fresh default
`shell.nix`, delete the old one first.

See the
[overview](https://github.com/acowley/cabbage/blob/master/Nix-notes.org#overview)
section of the source code for more information.

Background
---

The Nix packages collection, `nixpkgs`, is geared towards supporting
NixOS. There is, therefore, an emphasis on maintaining a single set of
chosen versions of packages that NixOS users can expect will work
together. In the Haskell development world, on the other hand, the
`cabal` build system goes to great lengths to support the
specification of dependencies with specific versions or version
ranges.

A limitation of the support for multiple versions of Haskell packages
has been [GHC](https://www.haskell.org/ghc/)'s use of package
databases populated by packages identified by names and version
numbers. An example problem is that a record of which versions of its
dependencies a package was built against is **not** reflected in the
package's name or version number. This leads to situations where
upgrading packages for one project *breaks packages used for an
entirely separate project*. Unacceptable!

More recently, `cabal` has gained support for *sandboxes* that permit
the user to maintain distinct package databases for different
projects. These work very well, but have the drawback that they end up
requiring the user to recompile common packages again and again
because each package must be freshly compiled for each sandbox it is
to be used in, no matter if the results of those compilations are all
identical.

Cabbage uses the Nix tooling to identify packages not only by their
name, version, and source code, but also by the versions of
dependencies they are built against. All of the distinct ways of
producing a compiled library are kept in the Nix store, but truly
redundant recompilations are avoided.

The Name
---

I wish I could take credit for the name `cabbage`, but it is entirely
due to Shae Erisson (shapr).
