All services go.mod files point the `goworkconflict/dep` pkg to the same `dep` folder but `servicec` specifies a different relative paths.

By default without a `go.work` go tools won't work when ran from the repo root, but will when ran from each module:

```
$ mv go.work go.work.tmp
$ go test ./...   
pattern ./...: directory prefix . does not contain main module or its selected dependencies

$ go test ./servicea
go: cannot find main module, but found .git/config in /Users/jean-baptistepinalie/Documents/code/goworkconflict
        to create a module there, run:
        go mod init

$ cd ./servicea
$ go test
PASS
ok      servicea        0.102s

$ cd ../servicea
$ mv go.work.tmp go.work
```

Using `go test` with the current `go.work` (only `servicea` and `serviceb` specified) makes `servicec` test runs fail.

```
$ go test ./...
pattern ./...: directory prefix . does not contain modules listed in go.work or their selected dependencies

$ go test ./servicea ./serviceb ./dep
ok      servicea        0.138s
ok      serviceb        0.098s
ok      goworkconflict/dep      0.178s

$ cd subfolder/servicec
$ go test
directory . is contained in a module that is not one of the workspace modules listed in go.work. You can add the module to the workspace using go work use .

$ cd ..
$ go work sync
```

Adding `dep` in `go.work` makes every test fail

```
$ go work use dep

$ go test ./...
go: dep@ used for two different module paths (dep and goworkconflict/dep)

$ go test ./servicea ./serviceb ./dep
go: dep@ used for two different module paths (dep and goworkconflict/dep)

$ go work sync
go: dep@ used for two different module paths (dep and goworkconflict/dep)
go: dep@ used for two different module paths (dep and goworkconflict/dep)
go: dep@ used for two different module paths (dep and goworkconflict/dep)
```

Adding `subfolder/servicec` in `go.work`:

```
$ go work use subfolder/servicec
$ go test ./...     

conflicting replacements found for goworkconflict/dep@v0.0.0 in workspace modules defined by /goworkconflict/serviceb/go.mod and /goworkconflict/subfolder/servicec/go.mod
conflicting replacements found for goworkconflict/dep@v0.0.0 in workspace modules defined by /goworkconflict/serviceb/go.mod and /goworkconflict/subfolder/servicec/go.mod

$ go work sync

conflicting replacements found for goworkconflict/dep@v0.0.0 in workspace modules defined by /goworkconflict/serviceb/go.mod and /goworkconflict/subfolder/servicec/go.mod
conflicting replacements found for goworkconflict/dep@v0.0.0 in workspace modules defined by /goworkconflict/serviceb/go.mod and /goworkconflict/subfolder/servicec/go.mod
```

Removing `dep` from the go.work (only contains `servicea`, `serviceb`, `subfolder/servicec`) produces the same result.

------

https://github.com/golang/go/issues/51204 related?