# Problems

## gopls report packages.Load error

One day, I found that when I go into some third-party packages, the gopls will report 

```
[Error  - 10:56:50 AM] 2022/04/30 10:56:50 go/packages.Load: err: exit status 1: stderr: go: updates to go.sum needed, disabled by -mod=readonly

	snapshot=1
	directory=/var/folders/c5/1hy64tyn7ss_f7fds7sp0tf40000gq/T/gopls-workspace-mod800980943
	query=[file=/Users/shaoqiangwu/go/pkg/mod/git.garena.com/shopee/bg-logistics/go/chassis@v0.2.7-b4/chassis.go]
	packages=0
```

then all functionality(e.g go-to-definition) provided by gopls are gone, can't navigate the source code made me sad

so I decided to do some research.

**Locate the source code that gave me the error**

from the messages reported by gopls, I knew the `packages.Load` is a function and gopls must call it somewhere, and the function return an error

so I search `packages.Load` in gopls repository, and I found there is only one place `internal/lsp/cache/load.go:116` (how lucky I am)

```go

cfg := s.config(ctx, inv)
pkgs, err := packages.Load(cfg, query...) // this line


func (s *snapshot) config(ctx context.Context, inv *gocommand.Invocation) *packages.Config {
	s.view.optionsMu.Lock()
	verboseOutput := s.view.options.VerboseOutput
	s.view.optionsMu.Unlock()

	// Forcibly disable GOPACKAGESDRIVER. It's incompatible with the
	// packagesinternal APIs we use, and we really only support the go command
	// anyway.
	env := append(append([]string{}, inv.Env...), "GOPACKAGESDRIVER=off")
	cfg := &packages.Config{
		Context:    ctx,
		Dir:        inv.WorkingDir,
		Env:        env,
		BuildFlags: inv.BuildFlags,
		Mode: packages.NeedName |
			packages.NeedFiles |
			packages.NeedCompiledGoFiles |
			packages.NeedImports |
			packages.NeedDeps |
			packages.NeedTypesSizes |
			packages.NeedModule,
		Fset:    s.view.session.cache.fset,
		Overlay: s.buildOverlay(),
		ParseFile: func(*token.FileSet, string, []byte) (*ast.File, error) {
			panic("go/packages must not be used to parse files")
		},
		Logf: func(format string, args ...interface{}) {
			if verboseOutput {
				event.Log(ctx, fmt.Sprintf(format, args...))
			}
		},
		Tests: true,
	}
	packagesinternal.SetModFile(cfg, inv.ModFile)
	packagesinternal.SetModFlag(cfg, inv.ModFlag)
	// We want to type check cgo code if go/types supports it.
	if typesinternal.SetUsesCgo(&types.Config{}) {
		cfg.Mode |= packages.LoadMode(packagesinternal.TypecheckCgo)
	}
	packagesinternal.SetGoCmdRunner(cfg, s.view.session.gocmdRunner)
	return cfg
}
```

`gopls` is constructing a config and do `packages.Load`

I want to reproduce this error

**Test it and find the root cause**

I write a main file with code like that

```go
func main() {
	wd := "{the third party package directory}"
	fset := token.NewFileSet()
	cfg := &packages.Config{
		Context:    context.Background(),
		Dir:        wd,
		Env:        nil,
		BuildFlags: nil,
		Mode: packages.NeedName |
			packages.NeedFiles |
			packages.NeedCompiledGoFiles |
			packages.NeedImports |
			packages.NeedDeps |
			packages.NeedTypesSizes |
			packages.NeedModule,
		Fset:    fset,
		Overlay: nil,
		ParseFile: func(*token.FileSet, string, []byte) (*ast.File, error) {
			panic("go/packages must not be used to parse files")
		},
		Tests: true,
	}
	ps, err := packages.Load(cfg, "{the third party package directory file}")
	if err != nil {
		fmt.Println("err: ", err)
	}
	_ = ps
}
```

and the `packages.Load` returned an error as expected

so I started debugging it, try to find the root cause of this error, and I found the underlying source code try to run `go list -e -f {{context.ReleaseTags}}`,
but got an error.

I tried to run this command manually, and it returned

```
go: writing go.sum: open xxxx/go.sum654949661.tmp: permission denied
```

permission denied because the module directory is non-writable, I use `chmod u+w` to give it the permission, and run that again, the output is:

```
[go1.1 go1.2 go1.3 go1.4 go1.5 go1.6 go1.7 go1.8 go1.9 go1.10 go1.11 go1.12 go1.13]
```

well, it seemed work, and I noticed it also create a `go.sum`

I turned the write permission off by `chmod u-w`, and run that again. This time, it didn't need to write the `tmp` file and return the same result as above

So I run the `main` function again, and it didn't return an error. Then, I went back to vim and restarted it, and the problem is gone (perfect)

I briefly tried some commands and it perfecly worked fine without error


**Root Cause**

Some of the third party packages I used didn't upload their `go.sum` file to the repository which caused the gopls fail to call `packages.Load()` with specific configuration

Finally, as [go official recommended](https://github.com/golang/go/wiki/Modules#should-i-commit-my-gosum-file-as-well-as-my-gomod-file), the `go.sum` file should be committed as well as `go.mod`





