# quartz

[![Build Status](https://travis-ci.org/DavidHuie/quartz.svg?branch=travis-ci-test)](https://travis-ci.org/DavidHuie/quartz)

Quartz enables you to call Go code from within your
Ruby code. This is accomplished by running a Go program
as a child process of your Ruby application and using UNIX domain sockets
for communication.

With this gem, you can now write performance critical code in Go, and that
code can be called from anywhere in a Ruby project.

To see some real world examples where Go can aid a Ruby project, please see
the `examples/` directory.

## Defining exportable structs

Quartz shares Go code by exporting methods on a struct to Ruby.

Quartz requires that all arguments to exported struct methods be JSON-serializable
structs. Additionally, the arguments to an exported method should be of the form
`(A, *B)`, where `A` and `B` are struct types. The method should also return an error.
Here's an example of an exportable struct and method:

```go
type Adder struct{}

type Args struct {
	A int
	B int
}

type Response struct {
	Sum int
}

func (t *Adder) Add(args Args, response *Response) error {
	*response = Response{args.A + args.B}
	return nil
}
```

## Preparing a Quartz RPC server in Go

Instead of integrating Quartz into an existing Go application,
it is recommended to create a new `go run`-able file
that explicitly defines the structs that should be available
to Ruby.

First, import the Go package:

```go
import (
	"github.com/DavidHuie/quartz/go/quartz"
)
```

Quartz maintains most of Go's native RPC interface. Thus, export a struct
by calling this in your `main` function:

```go
myAdder := &Adder{}
quartz.RegisterName("my_adder", myAdder)
```

Once all structs have been declared, start the RPC server (this is a blocking call):

```go
quartz.Start()
```

## In Ruby

Naturally:

```shell
$ gem install quartz
```

After you've found created a `go run`-able file, create a Go process wrapper that
points to that file:

```ruby
require 'quartz'

go_process = Quartz::GoProcess.new(file_path: 'spec/test.go')
```

Now, you can create a client:

```ruby
client = Quartz::Client.new(go_process)
```

To list exported structs:

```ruby
client.structs
=> [:my_adder]
```

To list a struct's exported methods:

```ruby
client[:my_adder].struct_methods
=> ["Add"]
```

To call a method on a struct:

```ruby
client[:my_adder].call('Add', 'A' => 2, 'B' => 5)
=> { 'Sum' => 7 }
```

## Copyright

Copyright (c) 2014 David Huie. See LICENSE.txt for further details.
