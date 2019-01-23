# CGO required for go test -race on Windows
This is a minimal example for reproducing https://github.com/golang/go/issues/27089.

`go test -race` will fail for `main` packages, if no C compiler is available, or CGO is disabled.

```
PS D:\gopath\src\github.com\femot\race-no-gcc> go version
go version go1.11.4 windows/amd64
PS D:\gopath\src\github.com\femot\race-no-gcc> go env
set GOARCH=amd64
set GOBIN=
set GOCACHE=C:\Users\aczeczka\AppData\Local\go-build
set GOEXE=.exe
set GOFLAGS=
set GOHOSTARCH=amd64
set GOHOSTOS=windows
set GOOS=windows
set GOPATH=D:\gopath
set GOPROXY=
set GORACE=
set GOROOT=C:\Go
set GOTMPDIR=
set GOTOOLDIR=C:\Go\pkg\tool\windows_amd64
set GCCGO=gccgo
set CC=gcc
set CXX=g++
set CGO_ENABLED=1
set GOMOD=
set CGO_CFLAGS=-g -O2
set CGO_CPPFLAGS=
set CGO_CXXFLAGS=-g -O2
set CGO_FFLAGS=-g -O2
set CGO_LDFLAGS=-g -O2
set PKG_CONFIG=pkg-config
set GOGCCFLAGS=-m64 -mthreads -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=C:\Users\aczeczka\AppData\Local\Temp\go-build324202174=/tmp/go-build -gno-record-gcc-switches
PS D:\gopath\src\github.com\femot\race-no-gcc> go test ./...
?       github.com/femot/race-no-gcc/fail       [no test files]
?       github.com/femot/race-no-gcc/success    [no test files]
PS D:\gopath\src\github.com\femot\race-no-gcc> go test -race ./...
# runtime/cgo
exec: "gcc": executable file not found in %PATH%
PS D:\gopath\src\github.com\femot\race-no-gcc> go test -race ./fail
# runtime/cgo
exec: "gcc": executable file not found in %PATH%
PS D:\gopath\src\github.com\femot\race-no-gcc> go test -race -vet=off ./fail
?       github.com/femot/race-no-gcc/fail       [no test files]
PS D:\gopath\src\github.com\femot\race-no-gcc> go test -race ./success
?       github.com/femot/race-no-gcc/success    [no test files]
PS D:\gopath\src\github.com\femot\race-no-gcc> gcc
gcc : The term 'gcc' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the name, or if a path was included, verify that the path
is correct and try again.
At line:1 char:1
+ gcc
+ ~~~
    + CategoryInfo          : ObjectNotFound: (gcc:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
PS D:\gopath\src\github.com\femot\race-no-gcc> $env:CGO_ENABLED=0
PS D:\gopath\src\github.com\femot\race-no-gcc> go test -race ./success
go test: -race requires cgo; enable cgo by setting CGO_ENABLED=1
PS D:\gopath\src\github.com\femot\race-no-gcc> $env:CGO_ENABLED=1
PS D:\gopath\src\github.com\femot\race-no-gcc> go test -race ./success
?       github.com/femot/race-no-gcc/success    [no test files]
```

# Results
- Adding `-vet=off` as option can be used as workaround
- Disabling CGO results in errors that tell me to enable CGO, even though it is not required
- Happens even without any test files present
- Happens only if the package name is `main`
- Not a side-effect of any imports