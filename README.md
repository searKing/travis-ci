[![Build Status](https://travis-ci.org/searKing/travis-ci.svg?branch=go-option)](https://travis-ci.org/searKing/travis-ci)
[![GoDoc](https://godoc.org/github.com/searKing/golang/tools/cmd/go-option?status.svg)](https://godoc.org/github.com/searKing/golang/tools/cmd/go-option)
[![Report card](https://goreportcard.com/badge/github.com/searKing/golang/tools/cmd/go-atomicvalue)](https://goreportcard.com/report/github.com/searKing/golang/tools/cmd/go-atomicvalue) 
[![Sourcegraph](https://sourcegraph.com/github.com/searKing/golang/-/badge.svg)](https://sourcegraph.com/github.com/searKing/travis-ci@go-atomicvalue?badge)
# go-option
Generates Go code using a package as a graceful option.

go-option Generates Go code using a package as a graceful options
Given the name of a atomic.Value type T
go-option will create a new self-contained Go source file implementing
```
// type TOption interface{
	apply(*Number)
}
// T
func (m *T) ApplyOptions(options ...TOption) *T
```

The file is created in the same package and directory as the package that defines T.
It has helpful defaults designed for use with go generate.

For example, given this snippet,

```go
package painkiller

type Pill struct{}
```

running this command
```bash
go-option -type=Pill
```

in the same directory will create the file pill_options.go, in package painkiller,
containing a definition of

```
var _default_Pill_value = func() (val Pill) { return }()

// A PillOptions sets options.
type PillOptions interface {
	apply(*Pill)
}
//
// EmptyPillOptions does not alter the configuration. It can be embedded
// in another structure to build custom options.
//
// This API is EXPERIMENTAL.
type EmptyPillOptions struct{}
//
func (EmptyPillOptions) apply(*Pill) {}
//
// PillOptionFunc wraps a function that modifies PillOptionFunc into an
// implementation of the PillOptions interface.
type PillOptionFunc func(*Number)
//
func (f PillOptionFunc) apply(do *Pill) {
	f(do)
}

func (o *Pill) ApplyOptions(options ...PillOption) *Pill {
	for _, opt := range options {
		if opt == nil {
			continue
		}
		opt.apply(o)
	}
	return o
}
```

Typically this process would be run using go generate, like this:
```bash
//go:generate go-option -type=Pill
```

If multiple constants have the same value, the lexically first matching name will
be used (in the example, Acetaminophen will print as "Paracetamol").

With no arguments, it processes the package in the current directory.
Otherwise, the arguments must name a single directory holding a Go package
or a set of Go source files that represent a single Go package.

The -type flag accepts a comma-separated list of types so a single run can
generate methods for multiple types. The default output file is t_options.go,
where t is the lower-cased name of the first type listed. It can be overridden
with the -output flag.

## Download/Install

The easiest way to install is to run `go get -u github.com/searKing/golang/tools/cmd/go-option`. You can
also manually git clone the repository to `$GOPATH/src/github.com/searKing/golang/tools/cmd/go-option`.

