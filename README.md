# Introduction to the OCaml ecosystem

__Tools__:
- [package manager](#Opam)
- [build system](#Dune)
- [text-editor features](#Merlin)
- [code auto-formatting](#OCamlformat)
- [documentation](#Odoc)

__Libraries__: (TODO)
- [printing](#Fmt)
- [logging](#Logs)
- [command-line interfaces](#Cmdliner)
- [pre-processing (PPX)](#ppx)
- [unit testing](#Alcotest)
- [fuzz testing](#Crowbar)
- [threading](#Lwt)

## Tooling

### [Opam][opam]

The standard OCaml package manager. Manages installation of:
 - OCaml compilers
 - OCaml-based CLI tools (including [Dune](#Dune) and [Merlin](#Merlin));
 - dependencies of OCaml libraries (when not managed by [Duniverse][duniverse]).

Each OCaml project specifies some metadata (dependencies, maintainer, license
etc.) in a top-level `.opam` file. In particular, the `.opam` file specifies the
version constraints for each of the project's dependencies. Packages are
typically installed from an OPAM _repository_ (defaults to
[ocaml/opam-repository][opam-repository]), which contains a large collection of
such `.opam` files. Releasing an OCaml package involves making a pull-request to
add new metadata to this repository.

#### Switches

`opam` maintains independent compiler+package environments known as _switches_
(similar to Python's 'virtual environments' and Haskell's 'sandboxes'). One
switch will be '_active_' at any given time, meaning that any OCaml builds will
use the corresponding compiler version and packages. The active switch also
determines which OCaml-based binaries are available in your shell environment.

A new switch can be installed with, e.g.

```shell
opam switch create <NAME> ocaml-base-compiler.4.08.1
```

And switched to with `opam switch set <NAME>`. When picking a compiler version,
it's usually recommended to use the latest version of `ocaml-base-compiler`. The
set of available compiler versions can be seen with `opam switch list-available`.

`opam` switches can be installed in specific directories, for instance in the
top-level of a project directory. Simply replace `<NAME>` in the installation
command above with the desired installation path. When OPAM is installed, it
will ask permission to add configuration to your `.<shell>rc` file that will
automatically select any local switches according to your current working
directory.

#### Pinning

Sometimes it is necessary for a project to depend on an unreleased dependency.
This can be achieved by '_pinning_' the dependency to a particular branch/commit
of a git repository. For instance, the following command pins the latest
`master` commit of the [mirage/index][index] library:

```
opam pin add index.dev git+https://github.com/mirage/index
```

This sets the version of Index to be `dev` and specifies the target from which
to install that version. After pinning a package, you must install it separately
with `opam install <NAME>`. `opam` requires you to explicitly re-install any
out-of-date pins with `opam upgrade <NAME>`.

It's possible to source a pin from your local file-system using e.g.:

```
opam pin add -k path <PATH/TO/SOURCE/CODE>
```

This is particularly useful when working on several libraries concurrently,
where one depends on another.

### [Dune][dune]

The (almost-)standard OCaml build system, installed via `opam`. Each project
contains a small `dune-project` file at the top-level. Build artefacts
(libraries, executables, tests etc.) are specified using `dune` files in the
same directory as the artefact.

### [Merlin][merlin]

Provides IDE features for OCaml, in your chosen text-editor. Most
importantly, allows you to see inferred OCaml types and any type errors
directly in your editor. Guides for configuring Merlin exist for [most common
editors][merlin-setup].

### [Odoc][odoc]

Inline OCaml code documentation, similar to [Javadoc][javadoc] and
[Haddock][haddock]. Reads 'documentation' comments of the form `(** ... *)` and
outputs an HTML tree that matches the project structure. Typically, each `.mli`
file will contain many documentation comments explaining the behaviour of each
field exposed in the signatures.

Full syntax is specified [here][odoc-syntax].

The documentation of an installed OCaml package can be browsed using
[odig][odig].

#### Integration with Dune

Documentation for a project can be generated via the Dune `doc` target:

```shell
dune build @doc
```

The documentation is generated in the Dune `_build` directory, and can be
browsed with e.g. 
```shell
xdg-open _build/default/_doc/_html/index.html
```

### [OCamlformat][ocamlformat]

An OCaml code auto-formatter. Highly-configurable, but with a recommended
'conventional' profile that uses sensible defaults. To use OCamlformat in a
project, the project must contain a top-level `.ocamlformat` file such as:

```
version = 0.11.0
profile = conventional
```

Dune supports OCamlformat directly. If the `dune-project` file of a given
project contains the line `(using fmt 1.2)`, the code can be formatted using:

```shell
dune build @fmt
```

Running this command will update a _copy_ of the project's source tree (stored
in the `_build` directory) and show any diffs from the real copy. If you are
happy with the suggested changes, run `dune promote` to update the real source code
accordingly.

It is recommended to configure your text editor to automatically run OCamlformat
over an OCaml file on each save. Instructions for doing this can be found
[here][ocamlformat-editor].

[dune]: https://github.com/ocaml/dune
[duniverse]: https://github.com/avsm/duniverse
[opam]: http://opam.ocaml.org
[opam-repository]: https://github.com/ocaml/opam-repository
[index]: https://github.com/mirage/index
[merlin]: https://github.com/ocaml/merlin
[merlin-setup]: https://github.com/ocaml/merlin#editor-setup
[odoc]: https://github.com/ocaml/odoc
[odoc-syntax]: http://caml.inria.fr/pub/docs/manual-ocaml/ocamldoc.html#sec357
[javadoc]: https://en.wikipedia.org/wiki/Javadoc
[haddock]: https://github.com/haskell/haddock
[odig]: https://github.com/b0-system/odig
[ocp-browser]: https://opam.ocaml.org/packages/ocp-browser
[ocamlformat]: https://github.com/ocaml-ppx/ocamlformat
[ocamlformat-editor]: https://github.com/ocaml-ppx/ocamlformat#editor-setup

## Libraries

All of the libraries in common use are available via OPAM.

- [fmt][fmt]; standard pretty-printing library that extends with standard OCaml
  [Format][format] library with combinators for user-defined pretty-printers.
  
- [logs][logs]; our standard OCaml logging framework.

- [alcotest][alcotest]; our standard unit-testing framework in OCaml.

- [cmdliner][cmdliner]; provides tools for creating a consistent CLI (argument
  parsing, manpage generation etc.).
  
- [lwt][lwt]; provides 'lightweight threads' for use in concurrency. Many Mirage
  libraries rely on Lwt to provide non-blocking IO operations. Tutorial
  [here][lwt-tutorial].

[fmt]: https://github.com/dbuenzli/fmt
[format]: http://caml.inria.fr/pub/docs/manual-ocaml/libref/Format.html
[logs]: https://github.com/dbuenzli/logs
[cmdliner]: https://github.com/dbuenzli/cmdliner
[lwt]: https://ocsigen.org/lwt/4.3.1/manual/manual
[lwt-tutorial]: https://ocsigen.org/tuto/6.4/manual/lwt
[alcotest]: https://github.com/mirage/alcotest
