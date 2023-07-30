#### Special Constructor Syntax

```go
package nostr

import "time"

type Timestamp int64

func Now() Timestamp {
	return Timestamp(time.Now().Unix()) // special constructor syntax
}

func (t Timestamp) Time() time.Time {
	return time.Unix(int64(t), 0)
}
```

#### Ensure Go Interface at Compile-Time

[Gopher stupid tricks.](https://medium.com/stupid-gopher-tricks/ensuring-go-interface-satisfaction-at-compile-time-1ed158e8fa17)

```go
// Example from Go-Nostr.

type RelayOption interface {
	IsRelayOption()
}

type WithNoticeHandler func(notice string)

func (_ WithNoticeHandler) IsRelayOption() {}

var _ RelayOption = (WithNoticeHandler)(nil)

```

#### Testing Panics

[How to test panics.](https://stackoverflow.com/questions/31595791/how-to-test-panics/62028796#62028796)

```go
func TestPanic(t *testing.T) {
    // No need to check whether `recover()` is nil. Just turn off the panic.
    defer func() { _ = recover() }()

    OtherFunctionThatPanics()

    // Never reaches here if `OtherFunctionThatPanics` panics.
    t.Errorf("did not panic")
}
```

More general solution.

```go
func TestPanic(t *testing.T) {
    shouldPanic(t, OtherFunctionThatPanics)
}

func shouldPanic(t *testing.T, f func()) {
    t.Helper()
    defer func() { _ = recover() }()
    f()
    t.Errorf("should have panicked")
}
```

#### Type-Safe Enums

[Type-safe enums for limited values.](https://stackoverflow.com/questions/69933473/type-safe-enums-for-limited-values/69933623#69933623)

```go
// unexported struct
type size struct {
   sz string
}

// exported interface
type Size interface {
   // Unexported method
   isSize()
}

// implement the exported interface for unexported struct
func (size) isSize() {}

// Stringer interface for convenience
func (s size) String() string {return s.sz}

var (
  Small Size = size{"small"}
  Large Size = size{"large"}
)
```

This way you can only use predeclared values.

[From All Design Patterns Golang.](https://github.com/mkohlhaas/All-Design-Patterns-Golang/blob/d74684976887a90d15c8219bbb1b6d152377c11b/Creational-Design-Patterns/Abstract-Factory/main.go#L96)

```go

// unexported struct;cannot create new brands
type brand struct {
   sz string
}

// implement the exported interface (see below)
func (brand) isBrand() {}

// Stringer interface
func (b brand) String() string { return b.sz }

// Public API

type Brand interface {
	isBrand() // unexported method: cannot create new Brands
}

// quasi enumeration types (type-safe(!))
var (
	Adidas Brand = brand{"Adidas"}
	Nike   Brand = brand{"Nike"}
)

func NewSportsFactory(brand Brand) (sf iSportsFactory) {
	switch brand {
	case Adidas:
		sf = &adidasFactory{}
	case Nike:
		sf = &nikeFactory{}
    // no default case(!)
	}
	return
}
```

#### Type Assertions

[Explanation of type assertions in Go.](https://stackoverflow.com/questions/38816843/explain-type-assertions-in-go/38818437#38818437)

In one line:

    x.(T) asserts that x is not nil and that the value stored in x is of type T.

Why would I use them:

    to check x is nil
    to check what is the dynamic type held by interface x
    to extract the dynamic type from x

What exactly they return:

    t := x.(T) => t is of type T; if x is nil, it panics.

    t,ok := x.(T) => if x is nil or not of type T => ok is false otherwise ok is true and t is of type T.

#### Creating a Byte Slice from a Literal String

[Converting between Byte Slices and Strings.](https://yourbasic.org/golang/convert-string-to-byte-slice/)

```go
package main

import "fmt"

func main() {
    values := []byte("abc") // creating a byte slice from a literal string
    fmt.Println(values)

    // Append a byte.
    values = append(values, byte('d'))

    // Print string representation of bytes.
    fmt.Println(string(values))

    // Length of byte slice.
    fmt.Println(len(values))

    // First byte.
    fmt.Println(values[0])
}

// [97 98 99]
// abcd
// 4
// 97
```

```go
// Convert string to bytes

// When you convert a string to a byte slice,
// you get a new slice that contains the same bytes as the string.

b := []byte("ABC€")
fmt.Println(b) // [65 66 67 226 130 172]
```

```go
// Convert bytes to string

// When you convert a slice of bytes to a string,
// you get a new string that contains the same bytes as the slice.

s := string([]byte{65, 66, 67, 226, 130, 172})
fmt.Println(s) // ABC€
```

#### Conditional Compilation

[Using build tags.](https://www.reddit.com/r/golang/comments/a8xb0x/comment/ecenc1c/?utm_source=reddit&utm_medium=web2x&context=3)

```go
// file debug.go

// +build DEBUG

package foo

const DEBUG=true
```

```go
// file nodebug.go

// +build !DEBUG

package foo

const DEBUG=false
```

```go
// file foo.go

package foo

import (
	"fmt"
	"os"
)

func bar() {
	...
	if DEBUG {
		fmt.Fprintf(os.Stderr, "This is a debug message")
	}
}
```

```shell
$ go build -tags DEBUG # to build a debug build that prints the debug message
$ go build # to build a normal build
```

[Example taken from Go-Nostr.](https://github.com/mkohlhaas/Go-Nostr)

```go
//go:build debug

package nostr

import (
	"encoding/json"
	"fmt"
)

func debugLogf(str string, args ...any) {
	// this is such that we don't modify the actual args that may be used outside of this function
	printableArgs := make([]any, len(args))

	for i, v := range args {
		printableArgs[i] = stringify(v)
	}

	DebugLogger.Printf(str, printableArgs...)
}
```

```go
//go:build !debug

package nostr

func debugLogf(str string, args ...any) {
	return
}
```
