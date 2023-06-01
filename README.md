Both `serviceb/go.mod` and `subfolder/servicec/go.mod` point `goworkconflict/dep` to the same folder `./dep` but specify different relative paths. 

```
$ go work use subfolder/servicec
$ go work sync

conflicting replacements found for goworkconflict/dep@v0.0.0 in workspace modules defined by /goworkconflict/serviceb/go.mod and /goworkconflict/subfolder/servicec/go.mod
conflicting replacements found for goworkconflict/dep@v0.0.0 in workspace modules defined by /goworkconflict/serviceb/go.mod and /goworkconflict/subfolder/servicec/go.mod
```

https://github.com/golang/go/issues/51204 related?