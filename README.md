Fixed by removing the `v0.0.0` part from all replace directives in all the `go.mod` files.

--------------

All services `go.mod` files point the `github.com/jybp/goworkconflict/dep` pkg to the same `dep` folder but `servicec` specifies a different relative path.

By default without a `go.work` go tools won't work when ran from the repo root, but will when ran from each module:

```
$ mv go.work go.work.tmp
$ go test ./...   
pattern ./...: directory prefix . does not contain main module or its selected dependencies

$ go test ./servicea
go: cannot find main module, but found .git/config in /goworkconflict
        to create a module there, run:go test ./servicea
        go mod init

$ cd ./servicea
$ go test
PASS
ok      github.com/jybp/goworkconflict/servicea 0.206s

$ cd ../
$ mv go.work.tmp go.work
```

Using `go test` with the current `go.work` (only `servicea` and `serviceb` specified) makes `servicec` test runs fail.

```
$ go test ./...
pattern ./...: directory prefix . does not contain modules listed in go.work or their selected dependencies

$ go test ./servicea ./serviceb ./dep
ok      github.com/jybp/goworkconflict/servicea 0.191s
ok      github.com/jybp/goworkconflict/serviceb 0.309s
ok      github.com/jybp/goworkconflict/dep      0.428s

$ cd subfolder/servicec
$ go test
current directory is contained in a module that is not one of the workspace modules listed in go.work. You can add the module to the workspace using:
        go work use .

$ cd ../../
$ go work sync
```

Adding `dep` in `go.work` has the same results:

```
$ go work use dep

$ go test ./...
pattern ./...: directory prefix . does not contain modules listed in go.work or their selected dependencies

$ go test ./servicea ./serviceb ./dep
ok      github.com/jybp/goworkconflict/servicea 0.190s
ok      github.com/jybp/goworkconflict/serviceb 0.126s
ok      github.com/jybp/goworkconflict/dep      0.232s

$ cd subfolder/servicec
$ go test
current directory is contained in a module that is not one of the workspace modules listed in go.work. You can add the module to the workspace using:
        go work use .

$ cd ../../
$ go work sync
```

Adding `subfolder/servicec` in `go.work` produces a new error:

```
$ go work use subfolder/servicec
$ go test ./...     

conflicting replacements found for github.com/jybp/goworkconflict/dep@v0.0.0 in workspace modules defined by /goworkconflict/serviceb/go.mod and /goworkconflict/subfolder/servicec/go.mod

$ go work sync

conflicting replacements found for github.com/jybp/goworkconflict/dep@v0.0.0 in workspace modules defined by /goworkconflict/serviceb/go.mod and /goworkconflict/subfolder/servicec/go.mod
```

------

https://github.com/golang/go/issues/51204 related?