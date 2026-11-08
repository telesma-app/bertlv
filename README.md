# bertlv

A small Go package for encoding and parsing definite-length BER-TLV objects.
It has no external dependencies.

```go
import "github.com/telesma-app/bertlv"

encoded := bertlv.Append(nil, 0x7f49, []byte{0x81, 0})
tag, value, rest, err := bertlv.Parse(encoded)
```

`Append` appends one object to the destination buffer. Tags are their wire
encoding (for example, `0x7f49`), rather than ASN.1 tag numbers. Callers supply
valid encoded tags.

`Parse` reads one object and returns its tag, value, and remaining input. Tags
are limited to four bytes; indefinite lengths are unsupported. Malformed or
unsupported input returns an error matching `bertlv.ErrInvalidEncoding` through
`errors.Is`.

The returned value and rest borrow the input buffer. The value's capacity equals
its length, so appending to it cannot overwrite the next object. Copy returned
bytes when an independent snapshot is required.

Interpretation of tags and nested objects belongs to the caller.

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE).
