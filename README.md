# MIME

Pure GoML MIME value helpers. `parse_media_type` and `format_media_type` handle
type/subtype values, quoted parameters, and UTF-8 RFC 2231 extended parameters
and continuations. Names are ASCII-case-normalized; values remain case-sensitive.
Parameter tokens use the full RFC 2045 ASCII alphabet, including `{` and `}` in
names and unquoted values. These characters are also legal unescaped bytes in
RFC 2231 extended parameter values.
Duplicate names, continuation gaps, malformed percent escapes, unsupported
charsets and decoded control bytes other than horizontal tabs are errors.
Formatting preserves tabs in quoted values, matching the parser's accepted
parameter values and [RFC 2045](https://www.rfc-editor.org/rfc/rfc2045.html#section-5.1)
quoted-string syntax. It emits UTF-8 extended parameters when values contain
non-ASCII characters, including percent-encoding any tabs in those values.

`decode_word` and `decode_header` handle RFC 2047 B/Q encoded words. Decoding
accepts UTF-8, US-ASCII and ISO-8859-1 and produces UTF-8 text. Adjacent
encoded words discard intervening horizontal whitespace. Malformed encoded
words, including truncated or overlapping framing markers, return errors.
`encode_word` emits UTF-8
B/Q words split at scalar boundaries and limited to 75 bytes per word. Q output
uses the restricted alphabet permitted in address display-name phrases and
comments, escaping punctuation such as commas, quotes and parentheses. Empty
input encodes as an empty string. Decoding requires nonempty printable ASCII
encoded text without spaces, tabs or line breaks, including in Base64 words,
as required by [RFC 2047](https://www.rfc-editor.org/rfc/rfc2047.html#section-2).

`Limits` bounds input, output, parameter count and value sizes. Inputs and
outputs are GoML strings, so raw invalid UTF-8 octets are outside this API.
The root-package helpers do not parse multipart bodies, transfer encodings, or
email addresses. Applications retain authority over header line folding,
transport limits and accepted media types.

The `quotedprintable` child package provides streaming transfer encoding and
strict decoding as a separate byte-oriented contract. The `multipart` child
package streams bounded part headers and binary bodies using textproto fields,
including bounded transport padding on boundary lines.

## Development and examples

Requires GoML 0.1.56 or newer. The `examples/basic/` example shares the root manifest and its dependencies. From the library root, run:

```sh
goml run --example basic
goml test
goml verify --timeout 300s
```

`goml test` builds the example and runs its tests. `goml verify` repeats the example checks as an independent module against an isolated registry snapshot. `(cd ../verification && just ecosystem-test mime)` also retains the library-specific smoke and compatibility checks.
