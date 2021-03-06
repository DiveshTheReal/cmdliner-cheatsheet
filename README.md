Cmdliner cheatsheet [![CircleCI badge](https://circleci.com/gh/mjambon/cmdliner-cheatsheet.svg?style=svg)](https://app.circleci.com/pipelines/github/mjambon/cmdliner-cheatsheet)
==

[Cmdliner](https://erratique.ch/software/cmdliner) is a
feature-complete library for parsing the command line from an OCaml
program. The output of `--help` is similar to a man page, see for
example `opam --help`.
Cmdliner is to be used instead of the `Arg` module from the standard library.

This cheatsheet is a compact reference for common patterns.

[A sample program](src/Main.ml) is also provided for exploration
purposes or as a template for new projects. Clone this git repository
and test it as follows (requires `dune` and of course `cmdliner`):
```
$ make
$ ./bin/cmdliner-cheatsheet --help
$ ./bin/cmdliner-cheatsheet
```

Anonymous argument at specific position
--

```
$ foo 42 -j 8 data.csv        -> Some "data.csv"
      ^^      ^^^^^^^^
      0       1
```

```ocaml
let input_file_term =
  let info =
    Arg.info []  (* list must be empty for anonymous arguments *)
      ~doc:"Example of an anonymous argument at a fixed position"
  in
  Arg.value (Arg.pos 1 (Arg.some Arg.file) None info)
```

Any number of anonymous arguments
--

```
$ foo                         -> []
$ foo a.csv b.csv c.csv       -> ["a.csv"; "b.csv"; "c.csv"]
```

```ocaml
Arg.value (Arg.pos_all Arg.file [] info)
```

Any number of anonymous arguments, defaulting to non-empty list
--

```
$ foo                         -> ["."]
$ foo a.csv b.csv c.csv       -> ["a.csv"; "b.csv"; "c.csv"]
```

```ocaml
let default = ["."] in
Arg.value (Arg.pos_all Arg.file default info)
```

Simple flag
--

```
$ foo             -> false
$ foo --no-exe    -> true
```

```ocaml
let no_exe_term =
  let info =
    Arg.info ["no-exe"]
      ~doc:"Example of a flag, which sets a boolean to true."
  in
  Arg.value (Arg.flag info)
```

Optional argument specification
--

```
$ foo                  -> 1
$ foo -j 8             -> 8
$ foo --num-cores 8    -> 8
```

```ocaml
let num_cores_term =
  let default = 1 in
  let info =
    Arg.info ["j"; "num-cores"]  (* '-j' and '--num-cores' will be synonyms *)
      ~docv:"NUM"
      ~doc:"Example of an optional argument with a default.
            The value of \\$\\(docv\\) is $(docv)."
  in
  Arg.value (Arg.opt Arg.int default info)
```

Optional value without a default
--

```
$ foo                 -> None
$ foo --bar thing     -> Some "thing"
```

```ocaml
Arg.value (Arg.opt (Arg.some Arg.string) None info)
```

Optional value with a default
--

```
$ foo             -> 1
$ foo --bar 8     -> 8
```

```ocaml
let default = 1 in
Arg.value (Arg.opt Arg.int default info)
```

Predefined argument converters
--

```
Raw argument        OCaml value        Converter

abc                 "abc"              Arg.string
true                true               Arg.bool
false               false              Arg.bool
x                   'x'                Arg.char
%                   '%'                Arg.char
123                 123                Arg.int
123                 123.               Arg.float
123.4               123.4              Arg.float
-1.2e6              -1.2e6             Arg.float
123                 123l               Arg.int32
123                 123L               Arg.int64
foo                 Foo                Arg.enum ["foo", Foo; "bar", Bar]
data.csv            "data.csv"         Arg.file (* path must exist *)
data/               "data/"            Arg.file (* path must exist *)
data/               "data/"            Arg.dir (* must be a folder *)
data.csv            "data.csv"         Arg.non_dir_file (* file must exist *)
foo.sock            "foo.sock"         Arg.non_dir_file (* file must exist *)
ab,c,d              ["ab"; "c"; "d"]   Arg.list Arg.string
17,42               [|17; 42|]         Arg.array Arg.int
ab:c:d              ["ab"; "c"; "d"]   Arg.list ~sep:':' Arg.string
```

Conclusion
--

Cmdliner offers various advanced and useful features that are not
covered here. Please consult the [official
documentation](https://erratique.ch/software/cmdliner/doc/Cmdliner.html).
