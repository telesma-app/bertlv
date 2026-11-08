# telesma-app/bertlv

[![Go Reference](https://pkg.go.dev/badge/github.com/telesma-app/bertlv.svg)](https://pkg.go.dev/github.com/telesma-app/bertlv)
[![Go](https://github.com/telesma-app/bertlv/actions/workflows/go.yml/badge.svg)](https://github.com/telesma-app/bertlv/actions/workflows/go.yml)

`telesma-app/bertlv` is a Go library for encoding and parsing definite-length
BER-TLV objects. It has no external dependencies or platform-specific code.

> [!WARNING]
> This module is under active development. Its public API may change during `v0.x`.

## Support

The package supports:

- one- through four-byte wire tags;
- short- and long-form definite lengths;
- appending encoded objects to an existing buffer;
- parsing one object while retaining the remaining input;
- parsed values that borrow the original buffer.

## Installation

```sh
go get github.com/telesma-app/bertlv@latest
```

See [`go.mod`](go.mod) for the required Go version.

## Quick start

Encode a nested object and parse its outer template:

```go
package main

import (
	"fmt"
	"log"

	"github.com/telesma-app/bertlv"
)

func main() {
	contents := bertlv.Append(nil, 0x81, []byte{1, 2})
	encoded := bertlv.Append(nil, 0x7f49, contents)

	tag, value, rest, err := bertlv.Parse(encoded)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("tag=%x value=%x rest=%x\n", tag, value, rest)
}
```

Tags are their wire encoding, such as `0x7f49`, rather than ASN.1 tag numbers.
`Append` expects a valid encoded tag and appends one object to `dst`.

`Parse` reads exactly one object without decoding its value. Parse `value`
again when it contains nested objects. Continue with `rest` to read subsequent
objects; an empty `rest` means the entire input was consumed.

## Errors

Malformed or unsupported input returns an error matching
`bertlv.ErrInvalidEncoding` through `errors.Is`. This includes truncated tags,
lengths and values, tags longer than four bytes, and indefinite lengths.

## Buffer ownership

`Append` follows slice append semantics and may reuse the destination buffer.

The value and rest returned by `Parse` borrow the input buffer. Mutations are
visible through both the original input and the returned slices. The value's
capacity equals its length, so appending to it cannot overwrite the next object.
Copy returned bytes when an independent snapshot is required.

## Scope

The package provides the BER-TLV encoding and parsing primitives used by PIV,
OpenPGP and secure-channel protocols. Interpretation of tags, validation of
object schemas and traversal of nested objects belong to the caller.

It does not provide ASN.1 struct marshaling, indefinite-length encoding or
application-specific data models.

## Testing

```sh
go test ./...
go vet ./...
make lint
```

## License

Apache License 2.0. See [`LICENSE`](LICENSE).
