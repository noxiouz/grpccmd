# grpccmd

grpccmd is a CLI generator for any gRPC service in Go.
While the CLI is written in Go the CLI should be able to talk to any gRPC server in any language.
The grpccmd is implemented as a plugin to protoc.

## Install

To install the protoc plugin binary run:

```
go install github.com/nathanielc/grpccmd/cmd/protoc-gen-grpccmd
```

## Example

To generate code for a CLI run this command.

```
protoc path/to/file.proto --grpccmd_out=.
```

It is recommended to place the above command as a Go generate comment in a main.go file.

Create a main.go file that references the generated code.

```go
package main

import (
	"fmt"
	"os"

	"github.com/nathanielc/grpccmd"
	// Import grpccmd generated code
	_ "github.com/nathanielc/grpccmd/example/internal/pb"
)

//go:generate protoc -I ../internal/pb/ ../internal/pb/example.proto --grpccmd_out=../internal/pb/

func main() {
	grpccmd.SetCmdInfo(
		"example-rpc",
		"Make calls to the Example service",
		"example-rpc command has been autogenerated via the protoc plugin github.com/nathanielc/grpccmd",
	)
	if err := grpccmd.Execute(); err != nil {
		fmt.Println(err)
		os.Exit(-1)
	}
}
```

The above code is part of a working example in the `./example` directory from this project.

To try it out run:

```
# Start the server
go run ./example/server/main.go
```

```
# Install the CLI
go install ./example/example-rpc
# Make a few calls to the server
example-rpc --addr localhost:50051 example getNumber
example-rpc --addr localhost:50051 example echo --input '{"str":"this is a string", "int": 42, "dbl": 6.9, "kv" : {"key":"value"}}'
```


## Server Code

The grpccmd wraps the normal grpc protoc plugin so the generated code can be used by the server as well.
Whether you do so is up to you as it is easy enough to generate the code twice, once for the server without grpccmd and once for the client with grpccmd.
Generating the code twice allows the client and server to be decoupled since they do not have to import the same package.

